## Translation Look Aside Buffer (TLB)
- reduce the access latency of page table
- Every user memory reference requires a translation
- TLB : a cache of most recently used translations
	- Given a VPN, returns a PTE - PPN & protection
	- TLB entry
		- Tag : addr tag from VA, PID
		- PTE : PPN, protection bits
		- Misc : valid, dirty...
### Direct Mapped TLB
![500](https://i.imgur.com/ZqQujpm.png)
### TLB Design
- locality가 클 것이므로 작게 만들어도 hit rate 클 것이다.
- C : if the L1 Icache is 64KB
	- A minimum of 16 TLB entries ($\to$ 4KB page) $\times$ some safety factor (2~8)
- B : After accessing a page, how likely is it to access the next page
	- typically one PTE per TLB entry
	- MIPS stores 2 consecutive pages' translations per entry
- a : what associativity to minimize collision?
	- In the old days, fully assoc (CAM)
	- Nowadays, 2~4 way assoc to support many translations
### TLB Miss
- TLB miss $\neq$ Page Fault
	- DRAM page table hit vs DRAM page table miss (on swap disk)
- Must walk the page table to determine translation
	- usually done by HW (MIPS walks in SW)
	- Bottom up (used in MIPS to reduce \# of memory access)
	- or Top down
- Table walking
	- if PTE is found, valid and page is in DRAM, then replace TLB with new PTE and continue
	- if PTE is found, valid, but the page is on disk then trigger *page fault* exception to initiate kernel handler for demand paging
	- if PTE is found, but it is not valid or PTE is not found then trigger *segmentation fault* exception to initiate kernel handler
![400](https://i.imgur.com/GSe83NJ.png)
>Hybrid : L1 cache access동안 TLB access하고 cache miss 시에만 PA로 memory access
## Virtual L1 cache
- Why not access cache with virtual addresses and only translate on a cache miss to DRAM
	- make sense if TLB hit time >> L1 cache hit time
- Die size has gotten large enough to integrate L1 cache
- MMU and TLB still off chip
- CPU has gotten fast enough that off chip SRAM access is too slow
### Synonyms & Homonyms
- Homonyms
	- Same EA(in different processes) points to different PAs
	- Sol
		- flush virtual cache after context switching
		- Include PID in cache tag ; use VA including PID
- Synonyms
	- Different EAs (from same or different process) point to the same PA
		- A PA could be cached twice under different EAs
		- Updating one cached copy would not reflect other cached copy
	- Sol
		- Make sure that synonyms cannot co-exist in the cache
		- OS can force synonyms to have the same index in a direct mapped cache
![300](https://i.imgur.com/Md3TlEB.png)

### Hybrid : Virtually - Indexed, Physically - Tagged
- If  C $\leq$ (page size $\times$ associativity), the cache index come only from page offset (same in VA and PA)
	- C/a $\leq$ page size $\to$ \# line of single cache bank of set $\leq$ \# line of page
- If both cache and TLB are on chip
	- index both arrays concurrently using VA bits
	- check cache tag (physical) against TLB output at the end
![600](https://i.imgur.com/js6BAx8.png)
- the capacity of single cache bank == capacity of physical page
	- using cache idx from PO of VA which is same with PO of PA
	- so that resolve synonym issue
- using TLB idx from VPN of VA
	- determine cache hit by comparing PPN of TLB entry and tag of physical cache corresponding to idx from PO of VA
- To increase cache size, must increase associativity not \# of idx (height of cache line)
	- To avoid synonym
### Alias in large virtually indexed cache
- if C > (page size $\times$ associativity), the cache index include VPN
	- $\to$ Synonyms cause problems
- Sol
	- increase associativity and reduce \# of cache index
	- increase page size or decrease cache size
![400](https://i.imgur.com/BbOC3de.png)

## HW or SW managed TLB?
- handle a TLB miss
	- HW
		- fast
		- little flexibility
	- SW
		- slow ; using plenty of special insts with exception
		- high flexibility
## Hierarchical or Inverted Page Table?
- Hierarchical
	- Can reduce the size of table if locality exists
	- Multi level page table walk on TLB miss
		- $\to$ bottom up walking can reduce some \# memory references on table walking as TLB miss
- Inverted
	- The size grows linearly with the size of physical memory
	- Possible hash collisions and require a back up page table for pages on disk
## ASID or Segmentation
![500](https://i.imgur.com/l7FxZl8.png)
## VM in MIPS
### MIPS R10K
![500](https://i.imgur.com/brA4MSU.png)
### MIPS TLB
- 64 entry fully associative unified TLB
	- each entry maps 2 consecutive VPNs to 2 different PPNs
- handled by SW
	- 7 inst page table walk (best case) on a TLB miss
	- TLBWR (write on random TLB partition)
		- HW chooses a random entry for TLB replacement
	- TLBWI (Write on wired/indexed TLB partition)
		- OS can reserve some TLB entries to keep critical translations that should not miss
- TLB entry
	- N : non cacheable // talk with main memory directly
	- D : dirty // a wirte enable bit
	- V : valid
	- G : global entry ; ignore ASID matching
		- To share address space btw other proccesses
![500](https://i.imgur.com/2i848l4.png)
