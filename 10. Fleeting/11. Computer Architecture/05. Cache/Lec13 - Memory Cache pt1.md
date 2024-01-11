## Ideal Memory
- Each program sees a *contiguous* 4GB memory
- We can access anywhere in memory in 1 cycle
## The Reality
- cannot afford and don't need as much memory as *the size of user address space*
- Most machines are multi tasking on several programs
	- sharing memory?
- No memory technology for *large size (GBytes)* and *few cycles (GHz)*
- Memory Abstraction for...
	- Large and fast
		- Memory hierarchy
	- Contiguous and private
		- Virtual memory
## The Law of storage
- Bigger is slower
	- latency : SRAM < DRAM < Flash Memory < Hard Disk
- Faster is more expensive
	- cost : SRAM > DRAM > Flash Memory > Hard Disk
## How to make memory Bigger, Faster and Cheaper?
### 1. Locality
- Typical programs are composed of *loops*
#### Temporal Locality
- If you *just* did something, it is very likely to do *same thing* again *soon*
- A program tends to reference the *same* memory location soon
#### Spatial Locality
- If you just did something, it is very likely to do something *related* again
- A program tends to reference a *cluster* of memory locations at a time

>[!corollary]
>A program may reference a large number of different memory locations over its live time but not all at the same time so that locality may change over time, but still exist

## Memoization
Storing the answer whose computation cost is expensive for a while, just in case the answer is needed again later.
- *Without Locality*
	- Storing a large number of different answers which may not reused
	- Locating the answer on demand later can be more expensive than recomputing it
- *With Locality*
	- Storing small number of the most frequently used answer avoids most recomputation
	- The answers may be reused many times
## Cost Amortization
>[!cite] *Overhead cost*
>one time cost to set something up

>[!cite] *Per unit cost*
>cost for per unit of operation

- $Total\ cost = Overhead+Per\ unit\ cost \times N$
- $Average\ cost= \frac{Total\ cost}{N}=\frac{Overhead}{N}+Per\ unit\ cost$
- High overhead is okay for large number of units so that the average cost can be lowered
## Memory Management
- manage data movement across hierarhies *manually*
- *Automatic* management across hierarchy
	- most recently used items in fast memory
	- Programmers don't need to know about size of cache
## Memory Hierarchy
![|400](https://i.imgur.com/FP7ij9j.png)

#### Hierarchical Performance
- $T_{i}=h_{i}t_{i}+m_{i}(t_{i}+T_{i+1})=t_{i}+m_{i}T_{i+1}$
	- recursive latency
	- smaller access time $t_{i}$ for smaller cache size
	- smaller miss rate $m_{i}$ for bigger cache size and good caching
![400](https://i.imgur.com/h3loYRm.png)
>T2=18+0.1x50x3.6
>T1=4+0.1xT2

- Low miss rate
	- Replacement : evict what you won't use
	- Prefetching : insert what you will use
- Low $T_{i+1}$
	- Faster low hierarchies increase cost
	- $\to$Intermediate hierarchies
- *DRAM*
	- used for large memory
	- optimized for capacity
	- T is fixed for a given tech generation
- *SRAM*
	- used for cache
	- optimized for latency and then capacity
	- different compromise btw capacity and latency possible
- *Hierarchies bridge the gap btw CPU speed and DRAM speed*
	- $T_{pclk} \sim T_{DRAM} \to$ hierarchy unneeded
	- $T_{pclk} \ll T_{DRAM} \to$ One or more levels of SRAM hierarchies are needed to minimize $T_{1}$ sustaining fixed cost
#### Why is DRAM slow?
![600](https://i.imgur.com/B8BVb7D.png)

## Cache
- any structure that memoizes frequently used results to avoid repeating long latency operations to reproduce the results
- Automatically managed memory hierarchy based on SRAM
	- Memoize the most frequently accessed DRAM memory locations in SRAM to avoid paying for the DRAM access latency repeatedly
### Cache interface
![400](https://i.imgur.com/WEQhgO7.png)
- Most of the time, result or side-effect valid after a short/fixed latency
- Caches may not be valid/ready on every cycle
### Issues
- How to keep the most frequently used items in M bytes of memory in C bytes of cache where C << M
- 1. Where to cache a memory location?
- 2. How to find a cached memory location?
- 3. Granularity of management ; large, small, uniform?
	- Instruction has high spatial locality, thus caching a large number of neighborhood data at a time with long word line
- 4. When to bring a memory location into cache?
- 5. Which cached memory location to evict to free up space?

- $M=2^m$
	- size of the addr space in bytes
	- sample value : $2^{32}, 2^{64}$
- $G=2^{g}$
	- cache access granularity in bytes
	- sample value : 4, 8
- C
	- capacity of cache in bytes
	- sample value : 16KB (L1), 1MB (L2)

### Direct Mapped Cache
![400](https://i.imgur.com/57ltciw.png)

#### Tag storage overhead
- For each G bytes cache block, additional t+1 bits must be stored
	- $t=\log_{2}M-\log_{2}C$
- *Solution* : let multiple G bytes words share a common tag
	- each B bytes block holds $\frac{B}{G}$ words where G is a size of unit word
![400](https://i.imgur.com/0xLQfux.png)

#### Cache Block
##### Block size & Collision
- C bytes of storage divided into $\frac{C}{B}$ blocks ($\frac{c}{b}$ entries)
- A block of memory is mapped to the cache block indicated by the same block index
- All addresses with the same block index field are mapped to the same cache block
	- $2^{t}(2^{\#\ tag\ bits})$ addresses which have same block index can be cached to same cache block
	- Even if C > working set size, collision is possible
	- Given 2 random addresses, chance for collision is $\frac{1}{\left( \frac{C}{B} \right)}$
	- increasing \# cache blocks $\frac{C}{B}\to$ decreasing likelihood for collision
- *Pros*
	- loading a multi word block at a time has high spatial locality
	- pay miss penaly only once per block
	- works well in inst cache especially
- *Cons*
	- increasing block size for fixed C
	- $\to$ reduces the \# of blocks
	- $\to$ increases possibility for collision
![200](https://i.imgur.com/gPzRj27.png)

##### Block size & total access time
- Loading a large block can increase $T_{i+1}$
	- although want only last word of block, should wait for the entire block to be loaded
- Sol1
	- Critical word first reload
	- $L_{i+1}$ returns the requested word first and then returns the rest words
	- Supply requested word to pipeline as soon as available
- Sol2
	- Sub blocking reload
	- Individual valid bits for each sub blocks
	- Relaod only requested sub block on demand
![600](https://i.imgur.com/iVjfXAs.png)
> all sub blocks share common tag
##### Cache Block & Locality
![500](https://i.imgur.com/ypJIp8F.png)
> 이웃한 word끼리 block이 형성되지 않고 띄엄띄엄 떨어진 word끼리 묶여 block이 형성되어 spatial locality를 잘 살리지 못한다.

![500](https://i.imgur.com/UJ9J86K.png)
> spatial locality는 그대로 유지하지만 block size를 줄인 것과 같은  효과를 통해 collision을 감소시킴?

### a-way Set Associative Cache
![400](https://i.imgur.com/5OXxWYU.png)

- C bytes of cache divided into *"a"* banks each with $\frac{C}{a\times B}$ blocks
- requires additional *"a"* comparators and *"a to 1"* mux
- $2^{t}$ addresses with same block index can be cached *"a"* such blocks at a time
	- $\to$ decreases collision
#### Replacement Policy
- A new cache block can evict old cached block in the same set
	- evict the one least recently used *(LRU)*
- if you actively use less than "a" blocks in a set, then any sensible replacement policy will quickly converge
- if you actively use more than "a" blocks in a set, then no replacement policy can help you
#### Pseudo Associativity
- Pseudo "a" - way associativity
	- Given an address A ,sequentially search "a" sets
	- {0, A\[log(C/B/a)-1 : logB]}, {1, A\[log(C/B/a)-1 : logB]}, ... {a-1, A\[log(C/B/a)-1 : logB]}
#### Fully Associative Cache ; a=C/B
![400](https://i.imgur.com/sVOq1Pv.png)

- CAM memory
- No index bits
- Any address can go into any of the C/B blocks
	- if C > working set size, then no collision
- One comparator per cache block, huge mux, many long wires
	- expensive
- For large C/B, a=4~5 is as good as a=C/B (fully assoc.)

![400](https://i.imgur.com/fGheU2p.png)
