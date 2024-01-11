## Introduction
Conventional performance counter using bottom up fashion doesn't provide information which can determine the accurate CPI component on out-of-order superscalar processors. But novel performance counter using top down fashion provides such information on out-of-order superscalar processors. 
## Constructing CPI stacks
CPI data is shown as stacked bar where each CPI components stacked on the top of each other CPI components stack while the base CPI is located on the lowest bottom of stacks. Application developers can use CPI stacks to optimize their software. Naive approach computing CPI stack components is to multiply the number of miss events by average penalty per miss event. However, the average penalty is not accurate and the miss events vary according to programs. The branch misprediction penalty varies according to benchmarks and can be larger than frontend pipeline length so that taking misprediction penalty by frontend pipeline length underestimates the actual penalties. Subsequently, naive approach doesn't consider the hidden or overlapped miss events by OoO superscalar processor so that the estimation can be skewed for CPI components. Finally, naive approach doesn't distinguish miss events along mispredicted control path and miss events along correct predicted control path so that counting events for both of them is not accurate. To resolve latter issue, Intel Pentium 4 uses naive-non-spec approach using tagging mechanism which tags instructions as they flows into pipeline and event counters are updated only when the instruction is completed.
In case of the IBM POWER 5, implements the dedicated HW performance counter to inspect the completion stage of pipeline with completion stall counters which counts the stall cycles for some given stall conditions.
- 1. ROB is empty
	- 1-1. I-cache miss or I-TLB miss leads to empty ROB, then I-cache/I-TLB completion stall counter increments counter until instruction flows into ROB again.
	- 1-2. Branch misprediction leads to instruction flush so that new instruction fetch is needed. ROB is empty until the new instruction flows into ROB through frontend pipeline. In this case, branch misprediction completion stall counter counts the number of cycle with empty ROB.
- 2. Head instruction of ROB is not completed so that zero-completion cycle happens.
	- 2-1. D-cache miss or D-TLB miss leads to instruction stall, then D-cache/D-TLB completion stall counter increments counter until memory operation resolved
	- 2-2. The head instruction is long latency instruction such as multiply, divide, FP operation so that the instruction is not completed. Then long latency completion stall counter counts the number of cycles until completion progresses again
## Underlying Performance Model
- Interval Analysis
	- top down approach
	- no need tracking of individual instructions
	- partition execution time into discrete time intervals by miss events
	- superscalar processor behavior and performance is determined for interval type
	- overall performance is estimated by aggregating the individual interval performances
### Frontend misses
#### I-cache & I-TLB misses
![400](https://i.imgur.com/y8Zpywv.png)

- L1 cache miss is constant for all benchmarks
	- small fluctuation with presence of fetch buffer
- novel approach : L1 cache miss가 났을 때 이미 pipeline에 들어온 inst들은 frontend pipeline length만큼 걸려 dispatch 된다. 이는 cache miss가 해결되고 fronend pipeline length를 거쳐서 다시 ROB가 채워지는 데 걸리는 시간과 동일하기 때문에 *L1 I-cache miss penalty = miss delay*
- naive approach multiply miss events by miss penalty cycles so that compute accurately
- POWER 5 consider only zero-completion cycles due to empty ROB that is after ROB is drained completely so that underestimates miss penalty. Because it except the time to drain ROB. If the time to drain ROB is larger than the miss delay since ROB is filled largely and there is low ILP or long latency insts, then any cycles won't be counted for miss
#### Branch misprediction
![300](https://i.imgur.com/4KSfewi.png)

- mispredicted branch가 ROB에 들어오고 나서부터 useful inst가 ROB에서 drain되기 시작하고 mis-speculated instruction들이 들어와서 ROB를 채우기 시작함. Branch miss resolution이 되면 pipeline을 flush하고 correct control path의 useful instruction을 fetch하기 시작한다. useful inst가 ROB를 다시 채우기까지 frontend pipeline length만큼 걸리기 때문에 그동안 이전의 useful inst가 모두 drain되고 나서부터는 zero-issue region이 생긴다.
	- mis-predicted branch가 언제 들어왔는지 track하고 있어야 한다.
	- branch misprediction penalty = branch resolution + pipeline length
- naive approach : pipeline length를 branch misprediction penalty로 계산하기 때문에 underestimate 한다.
- POWER 5 : zero-completion region만 count하기 때문에 naive approach보다 더 underestimate한다.
#### Backend miss
##### Short miss
- L1 D-cache miss can be hidden with out-of-order execution of independent instructions issued from large enough ROB
- So, consider load D-cache miss in a similar manner as the way considering instruction issued to long latency functional units
##### Long miss
- L2 cache miss, D-TLB miss
![](https://i.imgur.com/tQCdkCz.png)

- Single L2 D-cache miss
	- ROB will fills eventually because D-cache miss instruction blocks the head of ROB and then dispatch stops so that issue and commit stops. After data return from memory, instruction issuing resumes
		- long D-cache miss penalty equals the time between ROB fills and data returns from memory
- long D-cache miss instructions that follow closely another independent cache miss
	- closely : within W instructions following first D-cache miss instruction where W is ROB size
	- no additional penalty for single long D-cache miss penalty because their miss penalty is essentially hidden by first one's penalty
		- If the following miss instruction follows the first by S instructions, it flows into ROB before the first blocks the head of ROB and takes S/I cycles from the first one's issuing where I is dispatch width. S/I is enough time to overlap the remaining latency of following miss.
		- S only needs to be smaller than W
- Naive approach : ascribe total miss latency to all long backend misses, not consider the overlapping of long backend misses so that overestimate the actual penalties
- POWER 5 : ascribe a single miss penalty to overlapping backend misses
	- difference btw new approach : start to count just the long latency miss reaches the head of ROB whereas new approach waits until ROB is effectively filled up so that not count the works done in overlap with the long cache miss
#### Interactions between miss events
##### Interactions between frontend miss events
- Interaction between frontend pipeline misses is limited because the penalties cannot be overlapped. They disrupt serially instruction flow
- Frontend pipeline misses along mispredicted control flow paths should not be counted
	- Naive approach count all miss penalties including misses along mispredicted control flow paths so that leads to inaccurate estimation of penalties
	- Naive-spec approach, POWER 5 and new approach don't include misses along along mispredicted control flow paths
##### Interactions between frontend miss events and long backend miss events
- frontend misses can be overlapped by long backend misses
- but since the fraction of overlapped cycle is very limited any mechanism will result in accurate CPI stack so that implement hardware performance counter to count frontend miss penalty for overlapping between frontend miss and long backend miss unless ROB is full. If ROB is full, counts long backend miss penalty.
## Counter Architecture
### Frontend misses
#### FMT : Frontend Miss event Table
![400](https://i.imgur.com/s2DrmHe.png)

- *circular buffer*
- Three pointers
	- fetch pointer
	- dispatch head pointer
	- dispatch tail pointer
- as many rows as the processor supports outstanding branches
	- new branch instruction is fetched and decoded $\to$ FMT entry is allocated by advancing the fetch pointer and by initializing the entire row to zeros
	- $\to$ branch dispatches $\to$ dispatch tail pointer is advanced to that branch in the FMT
	- $\to$ the inst's ROB ID is inserted in the ROB ID column
	- $\to$ resolve that misprediction $\to$ mispredict bit of corresponding FMT entry with ROB ID is set
	- retirement of a branch $\to$ dispatch head pointer increment, which de-allocates the FMT entry

- calculating frontend miss penalties
	- L1, L2 I-cache miss, I-TLB miss $\to$ no instruction fetch $\to$ local counter in the FMT entry pointed by the fetch pointer increments every cycle until cache miss resolved
		- cache miss delay = miss penalty
	- branch
		- branch penalty counter is incremented every cycle for all branches between the dispatch head and tail pointer in the FMT
		- unless the ROB is full ; if ROB is full, counts long backend miss penalties to avoid double count for overlapping miss events
- global CPI component cycle counter
	- branch is completes
		- updated when a branch instruction completes, adding local counters except branch penalty to the respective global cycle counters
	- branch is mispredicted
		- if branch is mispredicted, branch penalty counter is added to the global branch misprediction cycle counter and from that moment, the global branch misprediction cycle counter is incremented every cycle until new instructions enter the ROB
		- dispatch tail pointer is placed to point to the mispredicted branch entry and the fetch pointer to the next FMT entry
#### Improved design : sFMT ; shared FMT
![400](https://i.imgur.com/vLvM9aV.png)
- only one shared set of local I-cache and I-TLB counters
	- The local I-cache and I-TLB counters in the FMT are updated in the FMT entry pointed to by the fetch pointer and the fetch pointer is advanced as each branch is fetched, which help to avoid counting frontend miss penalties past branch mispredictions. But tracking frontend miss penalties along mispredicted control paths needs high cost
	- Since no per branch I-cache and I-TLB counters in the sFMT, the sFMT only requires a fraction of storage bits compared to the FMT
- sFMT operation
	- the local I-cache and I-TLB counters is updated on I-cache and I-TLB misses
	- The completion of an instruction whose I-cache/I-TLB miss bit is set
		- 1. adds the local I-cache and I-TLB counters to the respective global counters
		- 2. resets the local I-cache and I-TLB counters
		- 3. resets the I-cache/I-TLB miss bits of all the instructions in ROB (completed inst 이후에 ROB에 들어온 frontend miss instruction들의 miss penalty가 이미 모두 counter에 반영되어 있음)
	- when mispredicted branch is completed
		- the local branch penalty counter is added to the global branch misprediction cycle counter and the entire sFMT is cleared including the local I-cache and I-TLB counters
		- clearing the sFMT on a branch misprediction avoids counting I-cache and I-TLB miss penalties along mispredicted paths
		- However, when an I-cache or I-TLB miss is followed by a misprected branch followed by an I-cache or I-TLB miss, then sFMT is inaccurate, because it counts I-cache and I-TLB penalties along mispredicted control paths
			- Since I-cache misses and branch mispredictions typically occur in bursts, this case is very limited $\to$ small error compared to FMT
### Long backend misses
- When ROB is full
	- if the instruction blocking head of ROB is L2 D-cache miss : D-cache miss counter is incremented until data is return from memory
	- if the instruction blocking head of ROB is D-TLB miss : TLB miss counter is incremented until data is return from memory (page table)
### Long latency unit stalls
- When ROB is full
	- if the instruction blocking head of ROB is L1 D-cache miss : count the cycle as L1 D-cache miss
	- if the instruction blocking head of ROB is long latency instruction, count the cycle as resource stall
## Evaluation
- naive approach
	- overestimate
- POWER 5 
	- underestimate

>[!question] branch misprediction resolution이 되어서 sFMT를 clear했는데 ROB에서 앞에 있는 instruction이 아직 completion 되지 않아 correct control path의 instruction의 I-cache miss penalty count가 날아간다면?

