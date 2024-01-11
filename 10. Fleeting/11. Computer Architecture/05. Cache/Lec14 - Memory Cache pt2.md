## Cache Miss
- Cold miss
	- design factor : B
	- First reference to an address always results in miss
	- Subsequent reference should hit unless the cache block is evicted
	- Sol : increase block size
	- *dominates when locality is poor*
		- both temporal and spatial!
- Capacity miss
	- design factor : C
	- It occurs even in a fully associative cache of same capacity
	- Sol : increase cache size
	- *dominates when C < W*
- Conflict miss
	- design factor : a
	- Data displaced by collision under direct-mapped or set-associative cache
	- Defined as any miss that is neither a cold nor a capacity miss
	- Sol : increase associativity
	- *dominates when C ≈ W or when C/B (\# of blocks) is small*
![300](https://i.imgur.com/vap2MJj.png)
> cold miss

![300](https://i.imgur.com/5KgG0KF.png)
> capacity miss

![300](https://i.imgur.com/AOFHS94.png)
> conflict miss

## Write Issues
- Tag bank needs to be accessed before the data bank to ascertain hit or miss
- In modern CPUs, tag and data bank accesses are *decoupled in scheduling*
	- *On read* attempt to schedule *simultaneous access* to tag and data banks as early as possible
		- Why? Delayed load is usually bad for performance $\to$ prevent that following load is delayed
	- *On write*, tag bank access scheduled first, meanwhile the data bank access possibly many cycles later
		- Why? Prevent that following back to back cache operation is delayed
- Issues on *partial word write*
	- The data bank must be read first to retrieve the unmodified bytes before writing back a complete word
	- Conversely on a *partial word read*, we pick out the bytes we want from entire block and ignore the rest
	- 모든 write는 read를 수반한다.
		- 전체를 read한 뒤에 partial word write 가능
### Store Buffer
- if store takes multiple cycles and immediately followed by memory inst
	- *structural hazard* stall occur by back to back cache operations
	- load inst is delayed
- Delay updating data to cache
	- After checking the tag bank to ascertain write hit, buffer write data until next free data bank cycle
	- It must be gauranteed that the cache line is not replaced from cache miss by others
- Memory Forwarding
	- *Later loads* must check the *pending store addresses* in the store buffer for *RAW dependency*
![400](https://i.imgur.com/3szGgaS.png)
> check store buffer before read
### Blocking Miss or Non-Blocking Miss?
- While cache miss is being handled...
	- In high-clock-rate ILP processors, it is essential not to lose many inst opportunities during cache miss
	- Must take care of the ordering and dependency while memory operations are completing out of order
- Even in an in-order pipeline, non-blocking write miss is useful
	- The pipeline does not have to stall just because a SW hasn't completed
	- But on a RAW dependent LW, must stall or forward
#### MSHR for Non-Blocking cache
- Non-blocking cache
	- keep executing insts while resolving a cache miss
	- *uArchitecture*
- *MSHR* ; miss status handling register
	- CAM 구조로 매 cycle마다 병렬적으로 search 해야 함
		- MSHR에 해당 data를 포함하는 block이 존재하는지 확인하고 해당 data를 사용하는 inst들에게 알려줘야 함 (ISQ에서 issue 대기 중)
	- Each entry in MSHR tracks the status of outstanding memory requests to a cache miss being serviced
- Miss definition
	- Primary miss
		- miss to a cache line
	- Secondary miss
		- Another miss to the cache line while the primary miss is still being serviced
		- L2 cache 가기 전에 MSHR에 적힌 block에 miss난 data가 포함되어 있는지 search $\to$ found $\to$ secondary miss
	- Structural miss
		- All MSHRs being used

![500](https://i.imgur.com/4zkb6cK.png)
1. On a miss, allocate one entry per cache miss
2. Prevent the pipeline from stalling due to other cache misses
3. New cache miss should check MSHR
	1. If there is matched item, this cache is being serviced due to miss
	2. All instructions that missed to be notified when miss is completed
4. Pipeline is stalled only when all MSHRs are used
### Write Hit Policy
#### Write Through Cache
- On a write hit in $L_{i}$, update also $L_{i+1}$
	- write through to lower level hierarchy
- Search can be done only by looking at the lower level hierarchy
	- good for I/O device : accessing lower level memory directly can get values *consistent with the upper level caches*
- Not a good option for high performance processors today
	- L1 write through to L2 is still possible if both are in same core
	- But L2(on chip) to L3(off chip) is expensive
#### Write Back Cache
- On a write hit in $L_{i}$, update only $L_{i}$ with *Dirty bit*
	- Write back to lower hierarchy $L_{i+1}$ which has older value when replaced
	- Reduce required bandwidth at lower hierarchies
- *Dirty bit*
	- status bit per block to record that block has ever been modified
	- If not dirty, no write back on replacement

>[!question] What if an I/O device wants to check upper-level hierarchy?

### Write Miss Policy
#### Write allocate
- On a write miss in $L_{i}$, allocate the block in $L_{i}$
- match to Write Back cache
#### Write No Allocate
- On a write miss in $L_{i}$, allocate the block in lower hierarchies
	- no allocate in $L_{i}$
- match to Write Through cache
### Unified vs Split I/D
- Harvard Archi
	- separate storages for inst and data
- Princeton Archi
	- Von Neumann : unified storage for inst and data
- Modern high performance processor
	- use split and asymmetrical L1 caches for inst and data
	- inst and data memory footprints typically *disjoint* 
		- inst : smaller footprint, higher spatial locality and read only
	- Split L1 caches provide *free doubled bandwidth*, *no cross pollution* and separate design customizations
- L1 and L3 are typically unified
### Multi level Caches
![600](https://i.imgur.com/3DylNnU.png)

![500](https://i.imgur.com/ZPxNqqE.png)
> L1 write through to L2
> L2(on chip) write back to L3(off chip) ; expensive write through

#### Multi Level Cache Design
- Upper Hierarchies
	- *Small C* : upper bounded by SRAM access time
	- *Smallish B* : upper bounded by C/B effects and the benefits of *fine-grain spatial locality*
	- *a* : some degree required to counter C/B effects
- Lower Hierarchies
	- *Large C* : upper bounded by chip area
	- *Large B* : to reduce *tag storage overhead* and to take advantage of *coarse grain spatial locality*
	- *a* : upper bounded by the implementation complexity
### Inclusive Cache
- Make $L_{i}$ contents is always a *subset of* $L_{i+1}$
	- L2(on chip)의 모든 data를 L3(off chip)에 써놓기 힘듦
	- If a memory location is important enough to be in $L_{i}$, it must be important enough to be in $L_{i+1}$
	- External agents(I/O, other processors) only have to check the lowest level hierarchy ; don't have to consume L1 bandwidth
- *Nontrivial to maintain when $L_{i+1}$ has lower associativity*
![400](https://i.imgur.com/j2RRzEa.png)

### Cache configuration in computer
- The presence or lack of a cache should not be detectable by functional behavior of SW
- measuring timing behavior infers the number of cache misses
#### Capacity Experiment (for D-cache)
- allocate a buffer of size R
- read every memory location in R in order and repeat by increasing R
- For small R $\leq$ C, we expect to hit in the cache
- For large R > C,  we expect to miss in the cache and experience a noticeable jump in memory access time
- By continuing increasing R, we can look step function for access time whenever the buffer size overs the next cache level
![300](https://i.imgur.com/E17NleM.png)

#### Block Size Experiment (with known C)
- Allocate a buffer of size R that is muliple of C
- For increasing S, read just every S'th memory location in the buffer and repeat
- Since R > C, we expect to miss on the first access to each cache block
- When S $\geq$ B, we only use one word per cache block
- When S < B, we expect improving average memory access time for smaller S since we read more words per cache block miss
>[!question] How do you detect block size for the lower cache hierarchies?


#### Associativity Experiment (with known C)
- For increasing values of R, where R is a multiple of C
- read every C'th memory location in order and repeat
- All R/C referenced addresses map to the same set
- When a $\geq$ R/C, we expect the references to hit in the cache since all referenced addresses fit whithin the set
- When a < R/C, we expect some misses since all referenced addresses cannot fit simultaneously
	- 100% cache miss if LRU is used
>[!question] How do you detect associativity for the lower cache hierarchies?
>hashing 때문에 실제로 알아내기 힘듦
>we don't know such address is mapped to which set



