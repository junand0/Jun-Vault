## Basics
- Niagara
	- 32 HW threads $\to$ should support high BW
		- $\to$ heavy demand on memory system
		- crossbar interconnects scheme routes memory reference to banked on-chip L2 cache that all threads share
		- 4 independent on-chip memory controllers $\to$ 20GB/s memory BW
	- CMP
	- FGMT
	- Solaris OS
- In data center
	- power supply cost
	- air conditioning cost
	- dissipated heat
- Server applications
	- large degrees of client request-level parallelism
	- high TLB
		- $\to$ shared memory with discrete single threaded processors $\to$ good performance
		- SMP with multiple processors designed to exploit ILP $\to$ low power & cost efficiency
		- $\to$ SMP server on a chip :  simple cores on a single die with shared on-chip cache and high BW off-chip memory
	- large working set
	- Low ILP ($\because$ large working set & poor locality $\to$ high cache miss)
		- no high ILP (high power) single thread processor
## Niagara Overview
- 32 threads in HW
- Archi organization 4 threads into a thread group
	- The group shares a processing pipeline : Sparc pipe
- 
