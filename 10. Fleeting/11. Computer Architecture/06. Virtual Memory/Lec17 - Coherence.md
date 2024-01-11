## Chip-Multiprocessor (CMP)
![400](https://i.imgur.com/zDfYwBk.png)
- current CMP adopt the familiar SMP paradigm
	- Symmetric (Shared Memory) Multi Processing
	- Multi processor(or cores) architecture where multiple processors(or cores) are connected to a single shared main memory
- Future designs focus on the uncore
	- How to support inter processor communication
	- How to support programmability
## Share Memory Abstraction
![400](https://i.imgur.com/oncETr7.png)
- What is the key advantage of shared memory system?
	- *Single Physical Address Space*
- Latencies from $P_{n}$ to Memory
	- UMA (uniform memory access) vs NUMA (non uniform memory access)
### Implementation 1 : Snooping Bus MP
- $\mu$Architecture
- Bus based system
	- typically small : 2~8 processors
		- 많이 연결되면 불리
		- critical latency까지 기다렸다가 그에 맞춰서 snoopping 해야함
		- 모두가 snooping 하는데 같은 시간을 소요한다는 가정
	- typically processors split from memories (UMA)
		- Sometimes multi processors on single chip (CMP)
		- Symmetric multi processors (SMPs)
![600](https://i.imgur.com/4gDXnYx.png)
### Implementation 2 : Scalable MP
![400](https://i.imgur.com/9XBdxpQ.png)

- General point to point network-based systems
	- Typically processor/memory/router blocks (NUMA)
		- Glueless MP : no need for additional glue chips
	- Can be arbitrarily large : 1000's of processors
		- Massively parallel processors (MPPs)
	- In reality only government (DoD) has MPPs
		- Companies have much smaller systems : 32~64 processors
		- Scalable multi processors

## MPs with Private Caches : Cache Coherence
![400](https://i.imgur.com/4EIMyJH.png)
### Extreme Solutions to Cache Coherence
- No caching of shared variables
	- bad performance down
- Allow multiple copies, but make sure they all have the same value at all times
	- Updates to on copy must be visible to all copies where ever they are (memory and all of the caches) ; it is so difficult to search all of copies
	- Thus can have multiple reader and writers at once
- Allow only one copy of a memory location at a time
	- L1(on chip) read sharing is impossible ; 다른 processor 까지 가서 읽어와야 하기 때문에 성능 저하
	- if location X is cached in one cache then it is not valid in memory or another cache (set valid bit 0)
	- Another processor must have a way to find out who has location X and take over ownership before reading or writing
	- Thus can only have one reader and writer per location
### Multiple Identical Copies protocol
- cache coherence protocol
- A cache line can be either valid or invalid
- Based on a **write-through** scheme
	- A cache issues a *read transaction* on a *read or write miss*
	- A cache issues a *write transaction to memory* whenever the *cache line is changed* by the processor
	- A cache does *not need to write back* when a line is displaced
- ==All writes are write-through== : cache is coherent with memory
- All caches ==snoop the bus for other's write transactions==
	- Check if the write is to a currently cached location
	- If a write is to a cached location
		- overwrite the old value with the new snooped value *(write update coherence)*
		- delete the old value *(valid-invalid coherence)*
#### Write-Through Scheme ; Write-Update Coherence
![600](https://i.imgur.com/AuTiq3U.png)
- Update all values of A in caches and memory
	- support multiple reader & multiple writers
- But 15% of cache accesses are stores
	- Tremendous bus and cache tag bandwidth required
#### Write-Through Scheme ; Valid-Invalid Coherence
![600](https://i.imgur.com/SuOaHti.png)
- Allows multiple readers, but must write through to bus
	- Write through & No allocate cache
	- just after write, only one copy exist ; make other copies invalid
- All caches must snoop all bus traffic
	- Simple state machine for each cache frame
##### Valid-Invalid Snooping Protocol
![600](https://i.imgur.com/YvuLzCb.png)
### One Copy at All Time protocol using Write Back cache
- Write Through schemes consume too much BW
- Cache issues a ==read transaction on a read or write miss==
- Cache issues a ==write back to memory only when a line is displaced==
- A cache line can be either *Modified* or *Invalid*
- All caches ==snoop the bus for other's read transactions==
	- If a cache observes a request to a currently cached line, then respond with a value in its cache and mark is own copy *Invalid*
	- Alternatively, a cache can also *ask the requestor to retry later* and in the meantime, *write back* its copy to memory
	- Allow only one copy so that don't need to snoop for write back transactions
![](https://i.imgur.com/lBo1P06.png)
### Multiple Copies protocol with Write Back cache
- must keep track of which cache is modified state
- add notion of ownership to Valid-Invalid
	- Mutual exclusion
		- When owner has only replica of a cache block, it may update it freely
	- Sharing
		- multiple readers are ok, but they may not write without ownership
- Must find which cache is an owner on read misses
- Must update memory eventually, so write are not lost
#### CC Protocol for Bus based system
- Bus is a broadcast medium so that bus snooping allows every cache to see what everyone wants to do
- A cache can even intervene in another cache's bus transaction
	- a cache might ask another cache to retry the transaction later or respond in place of the memory

#### MSI Protocol
![](https://i.imgur.com/VbzUZUl.png)

- An efficient policy for single writer and multi reader usage
	- allow multiple read - only copies (all identical) ; *Shared*
	- allow only a single writable copy ; *Exclusive, Modified*
	- minimizes the number of bus transactions
- Based on a *write back* scheme
	- on a read miss, issue a read transaction for a read only copy
	- on write miss, issue a read with intent to modify for an exclusive copy
	- on a write hit to a read only copy, issue an invalidate transaction
	- when displacing a exclusive line do nothing
	- when displacing a modified line write the dirty value back to memory
- All caches snoop the bus for other caches read, RWITM and invalidate transactions
- Three States
	- Invalid
		- cache does not have a copy
	- Shared
		- cache has a read only copy
		- Clean : memory is up to date
		- some other caches might have this copy as well
	- Modified
		- cache has a writable, dirty copy
		- dirty : memory is out of date
		- No other cache has this copy
- Three actions from processors
	- Load, Store, Evict
- Five messages from bus
	- BusRd, BusRdX, BusInv, BusWB, BusReply

![500](https://i.imgur.com/gqdi2Jb.png)

![500](https://i.imgur.com/ROrmSEI.png)

#### MESI Protocol
- 내가 유일한 S state이면 write할 때 BusInv transaction을 issue할 필요 없음
- $\to$ Exclusive state
	- only on copy ; writable, clean
	- Can detect exclusivity *when memory provides reply to a read*
	- Store transition to Modified to indicate data is dirty
	- No need for BusWB from Exclusive
![500](https://i.imgur.com/8rYtCng.png)

## Implementation of Snoopy Bus
- Every bus snoop requires a cache lookup
	- Needs a *dual ported cache* or at least an *extra tag lookup port*
	- In an inclusive L1 and L2 arrangement, snoop only goes to L2 and does not compete with processor for L1 bandwidth
		- True benefits of  inclusive cache
- Bus is not very scalable
	- physical limits force bigger bussses to clock slower
	- Bus bandwidth is divided when you add more processors
- Implementing snoopy protocols can get complicated
	- MESI state transitions are not really atomic
	- CPU and bus transactions are not atomic
	- CC issues can become related with memory consistency
## Scalable Cache Coherence
- Bus Bandwidth
	- Replace *non-scalable bandwidth substrate* (bus) with *scalable bandwidth* one (point to point network)
- Processor snooping bandwidth
	- most snoops result in no action
	- $\to$ Replace *non-scalable broadcast protocol* (spam everyone) with scalable *Directory protocol* (only spam processors that care)
	- 누가 어떤 state인지 tracking ; directory에 적어놓음
### Directory Coherence Protocols
- Physical address space statically partitioned
	- Bus-based protocol
		- broadcast events to all processors / caches
		- $\to$ Simple and Fast, but non-scalable
	- Can easily determine which memory module (home) holds a given line
	- Can't easily determine which processors have line in their caches
- Directories
	- *non-broadcast* coherence protocol
	- Extend memory to track caching information
	- For each physical cache line, its home memory records
		- ==Owner== : which processor has a dirty copy ; M state
		- ==Sharers== : which processors have clean copies ; S state
	- Processor sends coherence event to home directory
		- Home directory only sends events to processors that care
#### Centralized Directory
- Single directory contains a copy of cache tags from all nodes
- Advantages
	- Send Invalidate/Update only to nodes that have copies
	- Central serialization point ; ==easy to get memory consistency==
- Issues
	- Not scalable (What about traffic from 1000's of nodes)
	- Directory size/organization changes with number of nodes
![500](https://i.imgur.com/vBxjhBC.png)

![500](https://i.imgur.com/Mezjmt7.png)

#### Distributed Directory
- Memory block = Coherence block (=cache line)
- Home node : node with directory entry
	- dedicated main memory storage for cache line
- Scalable : Directory grows with memory capacity
- But directory can no longer serialize accesses across all addresses
	- Memory consistency becomes responsibility of CPU interface
![500](https://i.imgur.com/qltRUSL.png)

- Directory-based CC for many nodes
	- Bit Vector-based Directory CC
		- 10010110 $\to$ 1, 4, 6, 7 CPU are sharing the block
		- 8개의 CPU에 대해서 8 sharer 표시
	- For too many nodes, Pointer-based Directory CC
		- 1001 0110 $\to$ CPU 9, CPU 6 are sharing the block
		- 16개의 CPU에 대해서 2 sharer 표시
##### Designing a Directory Protocol
- Local Node (L)
	- node initiating the transaction we care about
- Home Node (H)
	- Node having directory or main memory for the block
- Remote Node (R)
	- Any other node that participates in the transaction
##### Read Transaction (data in home memory)
![400](https://i.imgur.com/NDnWhj9.png)
##### 4-hop Read Transaction (data in a remote cache)
![400](https://i.imgur.com/F3FwZwX.png)
##### 3-hop Read Transaction (data in a remote cache)
![400](https://i.imgur.com/Guq6YcZ.png)
>3 msg가 H, L 어느 쪽에 먼저 도착할 지 모름
##### Writeback & Read
![400](https://i.imgur.com/E9Vm91n.png)
>non-atomic senario
### Performance Issues in CC
- Deadlock
	- No one is making a progress
	- Everyone stopped to wait for a forever busy resource
	- P1 has modified copy of A, P2 has modified copy of B ; P1 issue read miss req for B on the bus, P2 issue read miss req for A on the bus ; What if they want to serve their older requests first?
- Livelock
	- Some are making progress, but to reach the old same status
	- Everyone tries to get a resource at every odd second, while the resource becomes available at every even second
	- If all CPUs try to write A simultaneously, they must invalidate other copies of A in CPUs. What if As in all caches continue to get invalidated before any write is completed?
- Starvation
	- Only some are making progress (unfair progress)
	- only one guy keeps winning in getting the resource
	- What if a specific CPU continues to become successful in writing to A, while other CPUs failing to write to A
## Share Memory Summary
- Shared memory multiprocessors
	- Pros : simple SW ; easy data sharing, easy programming
	- Cons : Complex HW ; must provide illusion of global address space
- Implementation
	- SMP (UMA)
		- bus based communication network in order
		- Pros : Low latency, simple protocols that rely on global order
		- Cons : Low BW, Poor scalability
	- MPP (NUMA)
		- point to point based communication network out of order
		- Pros : Scalable BW
		- Cons : Higher latency, complex protocols
- Coherence
	- consistent view of individual cache lines
	- Absolute coherence not needed, relative coherence OK
	- VI and MSI protocols, cache to cache transfer optimization
	- Implementation
		- SMP : snooping bus
		- MPP : directories
- Consistency
	- consistent global view of all memory locations
	- Programmers intuitively expect *sequential consistency*
		- Global interleaving of individual processor access streams
		- ==Not naturally provided by coherence==, needs extra stuff
	- Relaxed ordering
		- consistency only for synchronization points
