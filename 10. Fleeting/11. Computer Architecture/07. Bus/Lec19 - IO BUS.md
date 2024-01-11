## Processor I/O
- User Interface
	- Keyboard, mouse, video display
- Data transfer
	- disk, tapes, punch cards
- Communication
	- network interface
- Sensor / Control
![500](https://i.imgur.com/Tt7djPx.png)
## How to talk with I/O devices?
### I/O Port Registers (special purpose)
- Special input and output registers as part of the ISA PVS
	- Output : values written to an output register appear on the output pins of an external port
	- Input : reading from an input register returns the values at the input pins of an external port
- Simple
	- Easy to use, low latency
	- Can be specialized for apps
- Not general
	- Pre determined number of I/O signals ; fixed \# of pins BW and I/O registers
	- Specialized for specific apps
	- Non scalable
![200](https://i.imgur.com/oPVF4kE.png)
### Memory Mapped I/O : general approach
- Suitable with Load/Store Archi
	- communicate with I/O device using memory
	- Load/Store insts perform I/O to memory from the processor's perspective
	- LD/ST address identifies a specific memory location
		- ![150](https://i.imgur.com/1KFobYa.png)
	- LD/ST data conveys information
- Map a subset of the unused memory addresses to registers of external devices
	- LW from a memory map address means reading from the corresponding register of devices
- Memory and devices on the bus are programmed to respond only to their own address ranges
![300](https://i.imgur.com/nHkWk5t.png)
#### Mmap I/O : Idempotency problem
- LW/ST semantics are idempotent
	- when reading location M\[A], should get back the last value wrote to M\[A]
	- $\to$ Writing to M\[A] once has the same effect as writing to M\[A] with the same value 10 times
	- $\to$ Reading from M\[A] always return the same value unless write another value to M\[A]
- Mmap LW/ST have side effects
	- The receiver of mmap LW/ST may not be memory - like (idempotent)
	- The action of reading or writing a device register can imply other state changes
#### Mmap I/O : Cache Problem
- Issuing a LW or SW to a cacheable address ==may not lead to an external bus transaction==
- A cacheable address may appear in a bus transaction *spontaneously without being directed by LW or SW*
- Disallow caching on mmap address
	- flush mmap address entry from cache or make invalid
### DMA
- mmap I/O is slow and consumes CPU cycles
	- Let the I/O device read/write memory directly to move large data blocks to/from memory
- Commands to DMA device
	- ex) Read/write 1024KB starting from location 0x54100
	- CPU issues commands using mmap writes to device registers
- Commands to DMA engine
	- Copy a source block to a destination block
	- Source and destination region could be memory or mmaped
#### DMA : Virtual Memory problem
- Using VA
	- DMA command friendly
	- Size doesn't matter
	- requires translation from VA to PA
		- VA block don't need to be contiguous in PA
		- May not even be in memory at all
- Using PA
	- DMA command unfriendly
	- No translation
	- Cannot go over page boundary
	- Actual pages can be far distant
- SW solution *with some HW support*
	- User allocate special page if they are going to doing DMA transfer later
	- Kernel copies from user buffer to *pinned, contiguous buffer* before DMA
		- Pinned contiguous pages $\to$ no need translation
- Smart DMA engines : *a series of page level transfer*
	- OS creates a linked-list of commands in memory for moving non-contiguous blocks
	- DMA engine follow the linked list to perform gather/scatter
- Virtually addressed I/O bus : *translations at I/O*
	- I/O devices refer to contiguous data blocks by VA
	- Translate by I/O TLB into PA before accessing main memory
	- Can cause TLB miss or page fault
## When to serve I/O devices?
### Polling I/O
- ready : a read returns true if a new character is available
- data : a read returns the new character typed and ==resets ready== if no more characters are available
![500](https://i.imgur.com/v1yUdGi.png)
> CPU 1개가 위의 코드 계속 돌리고 있음

- Inefficient for *infrequent, but latency-sensitive* I/O events
### Interrupts
- cannot ask a busy executing thread to check I/O events
- Interrupt Vector
	- A set of input pins that can be asserted by I/O devices who need servicing
	- the device rings a door bell at the CPU
- Handle I/O only when interrupts are asserted
	- Stop the running thread
	- Switch to interrupt handler to service the interrupt
	- Return to the running thread after handling the interrupt
#### Interrupt Driven I/O
- Instead of polling, \_checkkbd is called by the interrupt handler only when the corresponding interrupt line is raised
- Suitable for
	- infrequent events
	- long latency operations ; signal the end of DMA transfer
- Performance consideration
	- I/O BW = transfer size / transfer time
	- Transfer Time = overhead + transfer size / raw_bandwidth
		- DMA has high raw bandwidth but large setup overhead
		- Mmap I/O has low bandwidth but no setup overhead
- CPU consideration
	- What fraction of processing is lost to I/O operations?
	- Can CPU do something else useful while I/O is happening?
	- How long can afford to let I/O wait?
## How does the bus work?
### Bus
- A broadcast medium
	- A common datapath connecting multiple devices
		- ==Reducing interconnection cost==
	- Single driver, multi receivers at a time
	- Time-multiplexed by transactions
		- ==Bandwidth is shared==
- Protocol
	- A set of handshake signals and rules of conduct to manage sharing by the bus devices
- Device types
	- Master : device who initiate transactions
	- Slave : devices who respond to transactions
	- Arbiter : a special bus device that manages shared bus usage
### Bus Transactions
- Memory-like R/W commands, address and data
	- Master issues R/W transactions to an address
	- Each slave is allocated an address range to respond in a memory-like way
- Transaction phases
	- Master requests ownership for the bus by asking arbiter
	- Arbiter grants ownership to master
	- Master drives address for all to see
	- Slave claims transaction
	- Master (or slave) drives data for all to see
	- Master terminates the transaction and bus ownership
### Bus Signals
- Private signals to/from arbiter per master
	- REQ : assert to request ownership ; de-assert to signal the end of transaction
	- GNT : ownership is granted
- Broadcast signals shared by all devices
	- R/W : read / write
	- AD : address / data
![400](https://i.imgur.com/8qu7DKD.png)

![400](https://i.imgur.com/ScllRcI.png)

![400](https://i.imgur.com/tbg34lX.png)

### Asynchronous Bus Protocols
- In Sync protocol
	- Slave can always decode address in 1 cycle : latency fixed
	- Slave can always respond in 1 cycle : latency fixed
	- Reasonable for the primary bus where the allowed devices are restricted ; processor, memory, chip-set
	- Unreasonable for an expansion bus that work with Gigabit Ethernet card
- Asynchronous handshaking
	- REQ/GNT is async handshake
	- MRDY and SRDY to coordinate AD usage
	- A bus cycle is valid iff MRDY && SRDY
![400](https://i.imgur.com/Ii6RDRw.png)

- Sync bus
	- common clk with fixed latency
	- simple logic for fast bus
	- cannot be long due to clk skew
- Async bus
	- no need clk
	- support long bus
### Bus Performance : latency
- REQ/GNT latency
	- Function of bus contention
	- Function of arbitration strategy
		- FIFO, round-robin
- Transaction latency
	- Function of slave reaction time
	- sync protocols have less wastage but cannot allow variable latency salves (fixed latency)
- Actual latency seen by the processor core is much longer than the raw bus latency
	- because the processor is biased to access the cache as quickly as possible to go out to the bus infrequently
### Bus Performance : Bandwidth
- BW for N-byte AD bus at frequency f
	- Peak BW = N • f
	- Must subtract overhead
- Each transaction has overhead cycles that don't contribute to data bandwidth
	- REQ/GNT phase
	- address and claim phase
	- termination phase
- overhead cycles can be amortized over ==multiple data cycles==
	- Consecutive address
	- Fixed sized burst on sync protocol 
		- transfer an entire cache line
	- Variable sized burst on async protocol
		- transfer to/from stream device
![400](https://i.imgur.com/B0zdSJh.png)

## Error Detection
- Bus is not an electrically-friendly environment
	- Long transmission lines with multiple taps that could drive and receive
	- Each device needs to switch a lot of external loads (AD bus) simultaneously
	- Noise
- AD bus transfers need to be protected by information redundant coding to detect bit errors
- Even-parity scheme
	- Add an *parity bit* to the AD bus
	- The driver set the parity bit such that ==XOR of all bus bits is 0==
	- The receiver checks the parity of the received bus value
	- Guaranteed to detect an odd number of bit flips but cannot detect even number of bit flips and cannot identify which bits flipped
#### Error Correction Codes : SEC-DEC
- Single error correction and double error detection
	- Insert parity bits in position $2^{i}$
	- p1 for the LSBs, p2 for the 2nd LSBs, p3 for the 3rd LSBs...

![](https://i.imgur.com/F4FvUpz.png)

![](https://i.imgur.com/z7BEHxj.png)

- SEC (Hamming Code)
	- we can correct a single bit error
- DED
	- A two bit error is seen as one bit error at a wrong position. However the final parity bit tells us something different from SEC
#### Advanced (Memory) Busses
- Pipeline Bus
	- Separate address and data busses
	- Pipeline the "request", "address" and "data" phases of 3 different transactions
	- works best if all transactions have the same latency
- Out of order Bus
	- Completely separate arbitration for the data bus
	- Each address bus transaction is assigned an unique *tag* and terminates without waiting for the data phase
	- When slave needs to respond, slave arbitrates for a data transaction on the data bus using the *tag* to identify the master
	- A slow slave doesn't stall the entire bus
- Switched Data Bus
	- With a split-phase protocol, the data bus doesn't have to be broadcast-based anymore
	- Use a switched network to send data at dedicated/high BW
