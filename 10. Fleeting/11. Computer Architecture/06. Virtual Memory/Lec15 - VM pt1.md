## Two parts of Modern VM
- In a multi tasking system VM provides *each process* with the illusion of a *large, private and uniform* memory
- **Naming and Protection**
	- Each process sees a large and contiguous memory segment *without holes*
		- virtual address space is much bigger than the available physical memory
	- Each process's memory space is private ; protected from access by other process
- **Demand Paging**
	- Capacity of secondary storage - ==swap space on disk==
	- At the speed of primary storage ; ==DRAM==
## Address Translation
- VM implements protection and demand paging with a common *HW mechanism* : addr translation
- User process operates on effective address
- HW translates from EA to PA on every memory reference
	- Controls which physical locations (DRAM or swap disk) can be named by a process
	- Allows dynamic relocation of physical backing store (DRAM vs swap disk)
- OS controls address tranlation HW and policies
	- *Privileged operation* protected from users
## Base and Bound
![500](https://i.imgur.com/0UK7mMk.png)
- When switching user process, OS sets base and bound registers
- Translation and protection check on HW
	- PA = EA + base
	- if (PA < bound) then ok / if not, violation
- User proccess cannot be allowed to modify the base and bound registers
	- Only on OS privilege level
## Segmented Address Space
- Limitations of base and bound scheme
	- *Fragmentation*
		- After the system runs, it is hard to get large contiguous space ; free space may be fragmented
	- *Sharing*
		- How do two processes share some memory regions but not others
- Give a user multiple memory segments
	- Each segment is defined by base and bound pair which is the unit of protection
	- Each segment is a contiguous region
### Segmented Address Translation
- EA comprise a segment number and a segment offset
	- SN may be specified explicitly(code) or implied(data)
	- Segmant can have different sizes
- Segment translation table
	- Maps SN to corresponding base and bound
	- Separate mapping for each process
	- Must be a privileged structure
![500](https://i.imgur.com/Pwu78SB.png)
### How to Protect?

|base | bound | extra bits|
| ---- | ---- | ---- |

- gerenric options
	- readable
	- wrtieable
	- executable
	- cacheable
- normal data page : RW!E
- static shared data page : R!W!E
- code page : R!WE
- illegal page !R!W!E
### How to extend an old ISA to support larger addresses for new applications while remaining compatible with old applications?
- User level segmented addressing
	- old applications use identity mapping in table
	- new applications *reload segment table at run time* with active segments to access different regions in memory
	- complication of *non linear* address space
		- dereferencing some pointers(long pointer) requires more work if it is not in an active segment
## Paged address space
- Divide PA and EA space into fixed size segments
	- page frames or pages - 4KB
- EA and PA are interpreted as page number (PN) and page offset (PO)
	- Page table EPN to PPN
	- EPE equals PPO and concatenate to PPN to get PPA
![300](https://i.imgur.com/ng2FwA2.png)
### Fragmentation
- **External Fragmentation**
	- A system may have plenty of unallocated DRAM, but they are useless if they do not form contiguous region of sufficient size
	- ==paged memory management== eliminates external fragmentation
- **Internal Fragmentation**
	- with paged memory management, a process is allocated an entire page (4KB) even if it need 4 bytes
	- A smaller page size reduces likelihood of internal fragmentation
>[!question] Why modern ISA are moving to larger page size
## Demand Paging
- Use ==main memory== and ==swap disk== as automatically managed levels in the memory hierarchies
- Similar to cache
	- but drastically different size and time scale, so that drastically different design considerations
- Basic issues
	- where to cache a virtual page in DRAM
	- How to find a virtual page in DRAM
	- Granularity of management
	- When to bring a page into DRAM
	- Which virtual page to evict from DRAM to disk to free up DRAM for new pages
### Terminology and Usage
- PA
	- addr that refer directly to specific locations on DRAM or swap disk
- EA
	- addr emitted by user applications for data adn code access
	- usage is most often associated with protection
- VA
	- addr in a large, linear memory seen by user application
	- often larger than DRAM and swap disk capacity
	- some addresses are not backed up by physical storage
	- usage is most often associated with demand paging
![400](https://i.imgur.com/G9xeQQY.png)
>segmentation

![400](https://i.imgur.com/QtwSDmy.png)

### Page Table size & speed
- Page table has following information
	- VPN to PPN
	- Protection bits : user-level? shared?
	- access types : dirty? valid?
- Each process must have a unique page table
	- page table can be very large
	- access to page table can be very slow
		- accessing DRAM to read a data from L1 cache?
## Page Table
- array of PTEs that maps VP to PP
- *Page hit*
	- reference to VM word that is in physical memory (DRAM hit)
![400](https://i.imgur.com/sCX28iv.png)
- *Page Fault*
	- reference to VM word that is not in physical memory (DRAM miss)
![400](https://i.imgur.com/cwUNq5o.png)
#### Handling Page Fault
- Page miss causes page fault : exception
- Page fault hadler selects a victim to be evicted
- move the page from swap disk to DRAM
- Offending inst is restarted, then page hit
- [ref](https://jesus-never-fail.tistory.com/m/31)
#### How large could be the page table?
- Don't need to keep track entire VA space
	- not all VA space is active
	- the system cannot use more memory locations than the physical storage (DRAM + swap disk) ; VM >> PM
- A clever page table can scale *linearly* with the size of physical storage not VA space
### How to reduce the size of page table?
#### Hierarchical Page Tables
- tree data structure in DRAM
![400](https://i.imgur.com/mcBTlVe.png)
- tree graph
	- Assume 4 byte descriptors and PTEs then each table is 4KB which is *size of single page* so that the page tables can be *themselves demand paged btw DRAM and disk*
- sparse tree graph
	- if no virtual page associated with an L2 table is in used,  then L2 table don't need to exist ; L1 descriptor = NULL
	- In general, an entire unused sub-tree can avoid
##### 32 bit VA with 4MB in use
- Best case : one contiguous 4MB region in VA aligned on 4MB boundaries
	- 1024 physical pages (4KB single page)
	- needs one L1 table + one L2 table = two 4KB
	- overhead ~ size of(PTE) / page size per physical page = 4B/4KB
![400](https://i.imgur.com/yu9S56Z.png)

- Worst case : 1024 4KB regions in VA each is 4MB aligned
	- 1024 physical pages
	- needs 1024 L2 tables ; only 1 PTE per L2 table is used
		- = (1+1024) 4KB
	- overhead ~ 1 page per data page
![400](https://i.imgur.com/Be0uqPn.png)
- Locality says we should be close to the best case
#### Inverted Page Table
- Maximum \# of (VA $\to$ PA) mappings are bounded by the size of physical memory anyway not VM
- Say we can have maximum N physical pages in the physical memory
	- N entry mapping table
	- use hash function to make an index of mapping table
	- Must search all N mappings to find a specific VA $\to$ PA ; CAM
- At least 1 entry per physical page, but probably more to avoid too frequent *hash conflicts*
	- 1GB DRAM $\to$ 256K frames each size 4KB $\to$ 256K PTEs
- Page table works like a *hash table*
	- each entry must be tagged by PID and VPN to detect collision
![400](https://i.imgur.com/wUrA6jy.png)
>index of inverted page table is PA

- Size of hashed page table is a function of *physical memory size* (at least)
	- Just large enough to reduce *hash collision*
- Hashed page table can store translation for pages currently in DRAM
	- On a miss, must consult a complete table structure to determine if the VPN is on swap disk or if the VPN is non existent
	- Collision Chain : a list of PTEs for same hash index - slow
