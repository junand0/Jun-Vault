## Basics
- billions transistors on chip
	- 1. add more memory
	- 2. increase the level of systems integration
		- bringing support functions ; graphics accelerators, I/O controllers
		- $\to$ lowers system cost, communication latency
	- 3. increase parallelism $\to$ enhance the processor's computational capability
- Let's achieve all types of parallelism
	- a single Superscalar
		- low ILP $\to$ no fully utilization
	- multiple Superscalars
		- low TLP $\to$ no fully utilization
	- $\to$ SMT
		- TLP from multithreaded parallel multi-program execution
		- ILP from each thread wide-issuing
- SMT
	- wide-issue superscalar + multithreaded processor
	- $\to$ issue multiple insts from multiple threads each cycle
	- better throughput with independent programs combination
## How SMT works
![400](https://i.imgur.com/Qvnx2xP.png)
>a: SS ; b : FGMT ; c : SMT

- SS
	- find multiple instructions which can be issued on a single thread
	- if fails $\to$ both vertical and horizontal waste
- FGMT
	- tolerate long latency operation $\to$ eliminate vertical waste
	- low ILP on each thread $\to$ cannot remove horizontal waste
- SMT
	- low ILP on each thread $\to$ multiple threads are executed together to compensate
## SMT model
- from OoO SS
	- 1. processor fetches 8 insts from I-cache in each cycle
	- 2. after decoding, register renaming maps logical register to physical register to remove false dependencies
	- 3. Insts are fed to integer or FP dispatch queues
	- 4. Insts are issued to corresponding functional units when their operands are available
	- 5. after complete execution, the processor retires them in order and frees HW registers that are no longer needed
- replicate SS resources
	- state for the HW contexts (registers and PC)
	- per thread mechanism for pipeline flush, inst retirement, precise interrupts and subroutine return
	- per thread ASID to the BTB and TLB
- Minimal redesign
	- Dynamic scheduling HW in OoO SS is functionally capable with SMT
	- Register renaming eliminates register conflicts both within and between threads so that processor can issues insts without regard to thread
- reduce negative impact targeted processor cycle time or the time to design completion
	- partition functional units across a duplicated register file to reduce ports on rf and dispatch queue
	- interleaved caches augmented with multiple independent addressed bank to increase simultaneous cache accesses
	- subdivide dispatch queue to reduce issue delay
	- reduce registers and cache ports by narrower issue width
### Instruction fetching
- fetch unit is SMT's performance bottleneck
	- fetch (independent) instructions more quickly to satisfy SMT's more efficient dynamic scheduler issuing more instructions per cycle from multiple threads
- SMT fetch unit can take advantage of the interthread competition for instruction bandwidth to enhance performance
	- partition bandwidth among the threads
		- fetch from one thread $\to$ difficult to fill issue slots due to branch inst and cache line boundary
		- increase probability to fetch useful non-speculative instruction
#### 2.8 fetching
- eight PCs, one for each thread context
- Fetch procedure
	- 1. On each cycle, selects 2 threads not I-cache miss
	- 2. fetch eight insts from each thread
	- 3. chooses a subset of these insts for decoding (to match HW issue width)
		- takes insts from the first thread until encounters branch or end of a cache line
		- takes the remaining insts from the second thread
	- cost : additional port on the I-cache & logic to locate branch inst
#### Icount feedback
- predict which threads will produce the fewest delays
- Icount feedback
	- highest priority to the threads with the fewest insts in the decode, renaming and queue pipeline stages
	- even though eight threads are sharing and competing for slots in the dispatch queues, the percentage of cycles where the queues are full is less than on a single threaded superscalar
- Performance
	- feed the dispatch queue with insts from fast-moving threads
		- avoid threads which will fill queue with insts with dependency and are consequently blocked behind long latency insts
	- maintains a fairly distribution of instructions among fast-moving threads in the queue
		- $\to$ increase interthread parallelism
	- avoids thread starvation
		- because threads whose insts are not executing will have few insts in the pipeline
### Register file and pipeline
- register renaming : maps architectural registers to hardware registers
	- HW register file size = archi registers in all thread contexts + additional renaming registers
- larger SMT RF $\to$ longer access time
	- extend SMT pipeline two stages to allow 2 cycle RF read and write
	- $\to$ avoid increasing the processor cycle time
		- extra stage btw fetch and execute $\to$ increase branch misprediction penalty by one cycle
		- two extra stages btw register renaming and commit $\to$ increase minimum time to hold HW registers
		- extra stage to WB $\to$ extra level of forwarding logic
## Simulation
### SMT vs SS
- SS
	- no exploit more ILP and any TLP
	- $\to$ vertical & horizontal waste
### SMT vs FGMT
- FGMT
	- eliminating vertical waste $\to$ higher performance than SS
	- max speedup with only four threads ; lower performance with more threads
		- eliminating vertical waste is sufficient with 4 threads
		- issue from only one thread $\to$ cannot hide the additional conflicts from interthread competition for shared resources
### SMT vs MP (multi processor)
- MP
	- fixed partitioning HW resources $\to$ cannot respond well to change TLP to ILP
	- low TLB $\to$ processors idle
- SMT
	- dynamic partitioning HW resources
	- only one thread executing $\to$ all HW resources dedicated to the thread
