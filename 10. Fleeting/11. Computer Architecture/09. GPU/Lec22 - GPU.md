## Flynn's Taxonomy

![500](https://i.imgur.com/alVtmFL.png)

![600](https://i.imgur.com/eL3xWZa.png)

## Data Parallelism
![300](https://i.imgur.com/J0zugj7.png)
>support vectors
>vector operation을 지원하지 않는 옛날 CPU에서는 실행되지 않는다.

- Data Parallelism != Data Parallel Execution
- Short Vector : better performance by parallel operations
![400](https://i.imgur.com/3nPGiJz.png)
- Long Vector : better performance by deep pipelining
	- no inter dependency between operations
![600](https://i.imgur.com/9MxMh4i.png)
## Basics of MP Universe
- Work distribution
	- Granularity of parallelism, ranging from whole programs down to bits
		- how finely the work is divided
	- Regularity
		- all nodes look same and look out the same environment
	- Static vs Dynamic (load balancing)
- Communication
	- Message passing vs Shared memory
		- Multiple Os with different apps vs OS with multiple threads
	- Granularity of communication, ranging from word to pages
	- Interconnect and interface design/performance
		- Topology, overhead vs Latency, synchronization
### SIMD
#### SIMD : Big computers
- Many PEs on a regular network grid
	- Synchronized common control
	- Direct access to local memory
	- Nearest-neighbor exchanges
	- Special broadcast/reduction
![300](https://i.imgur.com/zEd3OlO.png)
>CC-NUMA-DSM

#### SIMD: Old Vector Machines
- Vector registers
- Deeply pipelined vector functional units
- High performance load/store units and multi-banked memory
![300](https://i.imgur.com/QfQ0zuE.png)
#### SIMD : Modern Vector Machines
- Intel SSE
- NVIDIA GeForce 580 : Fermi GPU

### MIMD Shared Memory
#### MIMD Shared Memory : Symmetric Multiprocessors (SMPs)
- Symmetric means
	- Identical processors connected to common shared memory
	- All processors have equal access to system (memory and I/O)
	- OS can schedule any process on any processor
- Uniform Memory Access (UMA)
	- Process/memory connected by bus or crossbar
	- All processors have equal memory access latency
	- snoop-based cache coherence
![300](https://i.imgur.com/fURJSVg.png)

#### MIMD Shared Memory : Most Common Form of Multiprocessors Today
- General-purpose multicore processors implement the SMP paradigm on a single chip
- Even cheap motherboards provide multiple sockets 
	- one socket = one physical chip
![300](https://i.imgur.com/q6tzQYu.png)
#### MIMD Shared Memory : Distributed Shared Memory
- True UMA very expensive to scale due to concentration of BW
- Large scale SMPs have distributed memory with NUMA
	- Local memory pages - faster to access
	- Remote memory pages - slower to access
	- CC still possible but requires complicated distributed, message-based protocols
![300](https://i.imgur.com/mtWWiCP.png)

### The difference is in the cost of communication
- Processors may have to wait for data because *communication is not instantaneous*
	- Latency of network transit time and delay through intervening mechanisms
		- 각기 다른 processor의 request 중재로 인한 지연
	- Time to convey payload over finite link BW
	- A function of network implementation
- Processors must do something to send/receive a message because *communication is not spontaneous*
	- send/receive and protocol insts
	- Stall time due to blocking access to the NIU (network interface unit)
		- mmap read, write-through stall
	- A function of NIU design and placement
	- Shared memory programs also have this problem
#### Implications of communication performance
- Low BW
	- cannot communicate a large data
	- must have a lot of work to do per byte communicated
	- Only scalable for applications with high arithmetic intensity
- High Overhead
	- Cannot communicate frequently
	- Can only exploit coarse-grain parallelism
	- if DMA, the size of communication don't need to be limited
- High Latency
	- Producer cannot send data at the last minute
	- Must have high average parallelism
## Parallel Processing with GPUs & CUDA
### Graphics API
![400](https://i.imgur.com/6uYSBWg.png)
### Shading
![600](https://i.imgur.com/0MclzBW.png)
### Data Parallelism
![600](https://i.imgur.com/INY9LcM.png)
### Single Core
![600](https://i.imgur.com/P8jO7Od.png)

### 128 things in parallel
![600](https://i.imgur.com/yVXZq90.jpg)

- X cores can work on primitives ; triangles
	- geometry shader
- Y cores can work on vertices
	- vertex shader
- Z cores can work on fragments
	- pixel shader
- N cores can work on data/work
	- compute kernels, compute shaders
#### What about branching
![600](https://i.imgur.com/bIZocxf.png)
- Not all AOUs do useful work
- Worst case : 1/8 performance
- No Branch Prediction
#### How to handle stalls?
- Stalls occur when a core cannot run the next inst
- Lots of independent fragments
	- inst order is not important
	- GPUs don't have the large, fancy caches and logic that helps avoid stall because of a dependency on a previous operation
	- interleave processing of many fragments on a single core to avoid stalls caused by high latency operations
#### Hiding Memory Stalls
![400](https://i.imgur.com/wYgZMvS.png)
![400](https://i.imgur.com/41BhTxl.png)
>execution context를 더 쪼개서 group을 더 많이 만들고 한 group의 operation에 stall이 걸리면 다른 group을 실행하여 throughput을 높인다.

## CUDA Basics
![600](https://i.imgur.com/BVHETP4.jpg)
### Heterogeneous Programming
![500](https://i.imgur.com/r8LKQAT.png)

### CUDA Devices and Threads
- Compute device
	- coprocessor to the CPU or host
	- Has its own DRAM (device memory)
	- Runs many threads in parallel
	- typically GPU but also can be another type of parallel processing device
- Data-parallel portions of application are expressed as *device kernels* which run on many threads
- Differences between GPU and CPU threads
	- GPU threads are extremely light weight
		- Very little creation overhead
	- GPU needs 1000 threads for full efficiency
		- Multi core CPU needs only a few
![](https://i.imgur.com/RJKzl6t.png)

### Execution Model
- Each thread block is executed by a single multiprocessor
	- Synchronized using shared memory
- Many thread blocks are assigned to a single shared memory
	- executed concurrently in a time-sharing fashion
- Running many threads in parallel can hide DRAM memory latency
### Block and Thread IDs
- Threads and blocks have IDs
	- each thread can determine what data to work on
	- Block ID : 1D or 2D (grid)
	- Thread ID : 1D or 2D or 3D (cube)
![280](https://i.imgur.com/zI8ania.png)

### CUDA Device Memory Space 
- Each thread can :
	- R/W per-thread registers
	- R/W per-thread local memory
	- R/W per-block shared memory
	- R/W per-grid global memory
	- Read only per-grid constant memory
	- Read only per-grid texture memory
- Host can R/W global, constant and texture memories
![300](https://i.imgur.com/x1dBPvG.png)

### A Common Programming Pattern
- Local and global memory reside in device memory (DRAM)
	- much slower access than shared memory
- $\to$ Profitable way of performing computation on the device is *block data* to take advantage of *fast shared memory*
	- Partition data into ==data subsets that fit into shared memory==
	- Handle ==each data subset with one thread block== by :
		- 1. Loading the subset from global memory to shared memory, ==using multiple threads to exploit memory level parallelism==
		- 2. Performing the computation on the subset from shared memory
			- Each thread can multi pass over any data element efficiently
		- 3. Copy results from shared memory to global memory
## CUDA
- C - extension programming language
	- No graphics API
		- flattens learning curve
		- better performance
	- Support debugging tools
- Extensions / API
![500](https://i.imgur.com/wH4zlRP.png)
- Program types
	- Device program (kernel) : run on the GPU
	- Host program : run on the CPU to call device programs
### Compiling CUDA
![250](https://i.imgur.com/7FKDoT1.png)

### CUDA Function Declarations
![500](https://i.imgur.com/6JBlDLF.png)

### Language Extensions : Built in Variables
![500](https://i.imgur.com/XBjnCBF.png)

### Ex) Vector Addition
#### Kernel Code
![500](https://i.imgur.com/MsDLZUz.png)
#### Host Code
![500](https://i.imgur.com/hVKaL9r.png)

### CUDA Device Memory Allocation
- cudaMalloc()
	- Allocates object in the device global memory
	- Requires two parameters
		- Address of a pointer to the allocated object
		- Size of allocated object
- cudaFree()
	- frees object from device global memory
	- pointer to freed object
![300](https://i.imgur.com/v12hgBY.png)

### CUDA Host-Device Data Transfer
- cudaMemcpy()
	- Memory data transfer
	- Requires four parameters
		- Pointer to source
		- Pointer to destination
		- Number of bytes copied
		- Type of transfer
			- host to host
			- host to device
			- device to host
			- device to device
![300](https://i.imgur.com/0VYSZE9.png)

### Device Runtime Component : Synchronization Function
- void \_\_syncthreads() ;
	- synchronizes all threads in a block
	- once all threads have reached this point, execution resumes normally
- Used to avoid RAW / WAR / WAW hazards when accessing shared or global memory
	- thread가 같은 shared memory를 보도록 / thread block이 같은 global memory를 보도록
- Allowed in conditional constructs only if the conditional is uniform across the entire thread block
### CUDA Strengths
- easy to program ; small learning curve
	- C-extension
- Success with several complex applications
	- faster than CPU stand-alone implementation
- Allows us to read and write data at any location in the device memory
- More fast memory close to the processors : registers + shared memory
### CUDA Limitations
- Some hardwired graphic components are hidden
- Better tools are needed
	- profiling
	- memory blocking and layout
	- binary translation
- Difficult to find optimal values for CUDA execution parameters
	- Number of threads per block
	- Dimension and orientation of blocks and grid
	- Use of on-chip memory resources including register and shared memory
