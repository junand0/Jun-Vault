## Multicore Architecture
- based on a shared memory Archi
	- Cores access to a common physical addr space and memory
	- Processes or threads on different cores can communicate by writing and reading agreed upon memory locations
![500](https://i.imgur.com/tVX7CWx.png)

## Multi threading
- Process
	- each process has unique virtual address space not shared
	- If forking processes, they request regions of their VA to map to common PA
	- Copy on write
		- write할 때만 고유한 VA space를 copy해오고 read는 parent의 VA space에 link 걸어서 실행하여 process마다 copy하는 VA space size를 줄임으로서 overhead 최소화
- Thread
	- child thread끼리 parent의 VA space를 공유
	- process can comprise multiple threads
	- if forking threads, they share a common VA to map to common PA
	- 단 프로그램을 실행하는 과정은 다르기 때문에 (어떤 주소로 return할 지) stack은 each thread가 독자적으로 가진다
### Performance Analysis
- Adding 10000 numbers on 100 cores
	- 100 threads perform 100 additions each *in parallel*
	- The parent thread performs 100 additions *sequentially*
	- T = 100 + 100
	- S = 10000 / 200 = 50
- If summing 100000 on 100 cores
	- T = 1000 + 100
	- S = 100000 / 1100 = 90.9
- If summing 10000 on 10 cores
	- T = 1000 + 10
	- S = 10000 / 1010 = 9.9
- Fork, Join, distributing and collecting data are not free
![200](https://i.imgur.com/jZyv0O0.png)
### Process Fork
![200](https://i.imgur.com/vpF6ySF.png)
#### PThread create & join
![600](https://i.imgur.com/zodz3jR.png)
### Data Races when Sharing Memory
![600](https://i.imgur.com/dFmxh4V.png)

- M\[C] value를 각 thread가 동시에 읽어서 critical section을 실행하면 결과적으로 M\[C] 값에 1만 더해져서 프로그램 의도와는 다른 결과로 이어질 수 있다.
### Data Race Solution : Lock
![500](https://i.imgur.com/9CpkqP7.png)
#### Case : PThread Synchronization
![400](https://i.imgur.com/BSTUDcL.png)

>[!question] If different threads read L value simultaneously?
### Atomic Read-Modify-Write
- How to implement Acquire(L) in ISA?
- ==A special class of memory insts (atomic inst)== is needed for synchronization between concurrent or time-multiplexed threads
- Semantically atomic inst
	- read a shared memory location
	- perform computation
	- write something back to the same memory location
	- *Atomic* : This shared memory location cannot be written by anyone else between read and write
#### Atomic RMW Instructions
![400](https://i.imgur.com/CyQuzIg.png)
#### Acquire and Release with Atomic RMW insts
![400](https://i.imgur.com/A1AzIpj.png)
![400](https://i.imgur.com/2eVo4OD.png)
#### MIPS : Special load and store for atomic RMW
![500](https://i.imgur.com/28P40bc.png)
>CC protocol에 의해서 다른 process가 write하는 것을 알 수 있다

![](https://i.imgur.com/EUje7Fp.png)

#### Lock Performance & Fairness
- Traffic
	- High traffic generated during the waiting method (spinning)
	- Traffic contention at on chip - off chip network is performance killer
- Latency
	- Slow lock transfer after lock release
	- bad for lock transfer between different processors
- Fairness
	- lock acquire를 원하는 processor는 CC protocol에 의해 release한 processor로부터 L을 받아와야 때문에 lock release를 한 processor가 또다시 lock acquire할 확률이 높다.
	- $\to$ Starvation
![](https://i.imgur.com/5O9Xdc3.png)

### Test and Set Spin Lock (T&S)
![400](https://i.imgur.com/vNMsOFh.png)
- Performance problem
	- Huge CC protocol traffic during spinning $\to$ bus contention
	- spinning 동안 계속 write하기 때문에 invalid transaction을 날려서 bus 더러워지고 (bus contention) 다른 processor들은 계속 자신의 cache를 뒤져봐야 한다.
### TTS
![500](https://i.imgur.com/0U63wt8.png)
>*Read-Only spinning*
>The first test is not non-atomic but functionally correct

### Higher Lock Performance & Fairness
- TS
	- large bus traffic and bus contention
	- starvation
- TTS
	- still starvation
- LL - SC
	- Guarantee no one update the lock between my lock read and update (during lock acquire)
	- No atomic instruction used with HW support
## Memory Consistency
### Coherence vs Consistency
- *Coherence* concerns only one memory location
- *Consistency* concerns apparent ordering for all locations
- Memory system is *coherent* if
	- Can serialize all operations to that location such that,
		- Operations performed by any processor appear in program order
		- Value returned by a read is value written by last store to that location
### Multiprocessor Memory Consistency
![300](https://i.imgur.com/y34EL1K.png)
>In shared memory multi core

>[!question] Who performed the last write to X before R(X) by C2
>Global ordering of memory operation is needed

#### Sequential Consistency (SC)
![500](https://i.imgur.com/hIyHLA2.png)
- SC conditions
	- Every processor issues memory operations in program order
	- Memory operations happen atomically
- Three rules of SC
	- A thread perceives its own memory operations in program order
	- Memory operations from different processors can be interleaved arbitrary
	- But for each run, all threads on all processors must agree on a global ordering that every thread observed
![500](https://i.imgur.com/ED2fUOw.png)
>If Y' is 1 then X' cannot be 0

![400](https://i.imgur.com/KWPD6s4.png)
>In SC, T1 and T2 must see the stores to X and Y in the same order (X -> Y)

#### Dekker's Algorithm
- Mutually exclusive access to a critical region
![500](https://i.imgur.com/whHTpKz.png)
> P1 and P2 cannot enter the critical section simultaneously

### Problems with SC Memory Model
- Unnecessarily restrictive
	- Most parallel programs won't notice Out of order accesses
- Conflicts with latency hiding techniques
#### Issue : Add a Store Buffer
- Allow reads to bypass incomplete writes
![500](https://i.imgur.com/CI7dsxf.png)
>Break Dekker's Algorithm with store buffer

- P1 write A $\to$ P2 write B $\to$ P1 read B  by 0; P1 doesn't know about write B by P2 because write B not drain from store buffer to memory yet $\to$ P2 read A by 0 $\to$ enter critical section simultaneously

#### Fixing SC Performance
- Option 1 : Change the memory model
	- Weak / Relaxed Consistency
		- TSO(total store ordering) model (1 entry store buffer 허용)
	- Programmer specifies when order matters
		- Programmer가 Dekker's Algorithm을 실행할 때는 store buffer를 끈다.
	- Pros : Simple HW can yield high performance
	- Cons : Programmer must reason under non-intuitive rules
- Option 2 : Speculatively ignore ordering rules
	- In window speculation & recover if wrong
	- Store buffer에 store가 하나 남아있는 상태에서 Store가 바깥에 보이기 전에 (memory에 write되기 전에) Load들이 bypass된다. 그런데 bypass된 Load들 중에서 read한 주소가 이후에 store buffer에서 memory에 write한 주소와 겹치는 것이 있으면 old value를 read한 것이므로 이 경우에만 roll back하고 겹치는 것이 없는 경우에는 order rule을 어기지 않은 것처럼 보이기 때문에 문제 없다.
	- program끼리 memory를 공유하는 경우는 매우 드물다.
	- Pros : Performance of relaxed consistency with simple programming model
	- Cons : More sophisticated HW ; speculation can lead to pathological behavior
