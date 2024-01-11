## Static RAM (SRAM)
- Read
	- sense bitline difference
	- 양쪽의 bitline을 1로 precharge 해놓고 pull down speed에 따라 빠르게 pull down되면 0, 느리게 pull down되면 1로 구분
- Write
	- overwrite bitlines using strong signals
- 1 Cell = 6 transistors = two invertors + two switches
![300](https://i.imgur.com/sz7ZilW.png)
>SRAM cell

- Read sequence
	- (assume bitlines are precharged)
	- 1. Address decode
	- 2. drive row select ; drive selected wordline
	- 3. selected bit-cells drive bitlines ; entire row is read together
	- 4. difference sensing and column select $\to$ data is ready
	- 5. precharge all bitlines for next read or write
- *Access latency* dominated by step 2, 3
- *Cycle time* dominated by step 2, 3, 5
	- step 2 $\propto 2^{m}$
	- step 3 and 5 $\propto 2^{n}$

![300](https://i.imgur.com/HqPD24x.png)

## DRAM
- SRAM과 DRAM은 한 공정을 통해 한 wafer 안에서 나올 수 없다.
- Bits stored as charge in capacitor
	- Precharge bitline to $\frac{Vdd}{2}$
	- bit cell 1 $\to$ bitline charge increase
	- bit cell 0 $\to$ bitline charge decrease
- Requires periodic refresh
	- performed by memory controller
	- unavailable during refresh ; frequent refresh is bad for performance
- *Disruptive read*
	- read가 값을 손상시키기 때문에 read한 뒤에는 read 값으로 refresh 시켜준다.
	- read가 곧 refresh이다.

![300](https://i.imgur.com/7Rnocto.png)

![300](https://i.imgur.com/AawtY8q.png)

- SRAM
	- faster access using bit-differential sense
	- no need refresh
	- static power consumption due to Vdd for inverter operating
- DRAM
	- smaller cells so that higher density
	- need refreshing
	- dynamic power consumption due to refreshing
### DRAM organization
- 1 bit = 1 capacitor
- CPU $\to$|cache line 단위| DRAM $\to$|page 단위| DISK
![400](https://i.imgur.com/lScoOAE.png)

#### Access Process
- **Bus Transmission**
![500](https://i.imgur.com/ohl4SpZ.png)

- **optional Precharge & Row access**
![500](https://i.imgur.com/RQ8TS6o.png)

- **Column access**
![500](https://i.imgur.com/KP30ZVO.png)

- **Data Transfer**
![500](https://i.imgur.com/sJsCSdh.png)

- **Bus Transmission**
![500](https://i.imgur.com/XsiN521.png)

#### DRAM Latency
![500](https://i.imgur.com/a4ALt2t.png)
- Sense Amps에는 가장 최근에 읽거나 쓴 값이 남아있다.
### DRAM physical organization
![500](https://i.imgur.com/PD8lOdw.png)
#### Bank
![500](https://i.imgur.com/sFqr1zj.png)
>Bank

![300](https://i.imgur.com/Pr3jQMR.png)
#### Rank
![500](https://i.imgur.com/kfBHcqM.png)
#### DIMM
![500](https://i.imgur.com/uKGgj1k.jpg)
### Read timing of DRAM
#### Conventional DRAM
![500](https://i.imgur.com/3nClbCb.png)
#### Fast Page Mode ; FPM
![500](https://i.imgur.com/4dpaijD.png)
>sense Amp에 있는 entire row data와 같은 row의 data를 읽을 때는 column address만 줘도 됨
#### Extended Data Out ; EDO
![500](https://i.imgur.com/VzRttHM.png)
*overlapped CAS with Data out*
#### Burst EDO
![500](https://i.imgur.com/Nwd3j5g.png)
#### Synchronous DRAM ; SDRAM
![500](https://i.imgur.com/pYJbseY.png)

## DRAM real world design issues
- BW
	- GPU
- Latency
	- CPU
- Clock Skewing
- BUS and Path Length issues
### SDRAM Topology
![500](https://i.imgur.com/EqvDT5g.png)

### DDR SDRAM topoloty
![500](https://i.imgur.com/7s8BNJc.png)

#### DDR vs DDR2 vs DDR3
![](https://i.imgur.com/GlzsBNV.jpg)
- Single rate SDRAM vs DDR SDRAM
	- double pumping data at both rising and falling clk edges
	- 2x BW over SDRAM ; 2b prefetch
- DDR vs DDR2
	- doubling I/O bus and data bus clk
	- 2x BW over DDR ; 4b prefetch
- DDR2 vs DDR3
	- doubling I/O bus and data bus clk ; 8b prefetch
	- 1.8V to DDR2 $\to$ 1.5V to DDR3 ; 30% less power consumption
### DRAM Memory Controller
#### First come first serve ; FCFS

- serves memory requests in order
- not distinguish different threads or memory episodes
- favors memory intensive threads
- not exploit row-buffer locality
- not exploit bank level parallelism of threads
#### First ready FCFS ; FR-FCFS
- Prioritize *Row-hit* requests over others $\to$ maximum throughput
	- exploit row buffer locality
- Older requests over younger ones
- Thread-unawareness
	- starvation for low row-hit threads
	- 서로 다른 thread가 같은 row의 data를 요구할 가능성이 낮음
#### Fair queuing memory scheduler (FQM)
- exploit the fair queuing algorithm from computer network
- partition memory BW equally among threads
	- For each thread, in each bank, FQM records a counter (virtual time) and increases this counter when memory request of the thread is serviced
	- Prioritize early virtual times for fairness
- not work well for non-memory intensive workloads
- not exploit row buffer locality
## FLASH
- electrically modifiable and non-volatile storage
- Transistor with a floating gate
### Flash cell operation
![400](https://i.imgur.com/olVEuwU.png)

- erase = less resistant = fast turn on
- programmed = strong resistance = slow turn on
	- FG에 trap된 electron들이 channel 형성을 방해

#### Flash Programming
- Programming Flash
	- Apply high voltage on control gate
	- $\to$ hot electron injection
	- $\to$ Vt increase $\to$ turn on higher voltage $\to$ low current = 0
- Erasing Flash
	- Discharge charges trapped in FG
		- Apply high voltage on drain
		- electron tunneling
### FLASH : NOR model
- NOR = even only single cell can pull down the bitline to 0
- Each cell connected to GND and bitline
	- On WL activated, the connected cell pulls the bitline low
	- 1 or 0 based on the pull-down speed
		- 1 : pull down fast
		- 0 : pull down slowly
- Enable fast & word-level programming
	- fast and random access ; 원하는 word line read 가능
	- Read/Write on word level
	- However, erasure is still done at block level
- In-place memory
![500](https://i.imgur.com/rSfELI6.png)

### FLASH : NAND model
- NAND = all other cells should be 1 to pull down the bitline to 0
- All cells are connected to global GND in serialized manner
	- All cells pulled up over Vt (high) + one cell pulled up at Vt (low)
	- 1 or 0 based on whether current flows or not
- Sequential, block-level erasing
	- Block = a group of pages
	- Block : 16KB ~ 512KB ; erasure granularity
	- Page : 512B ~ 4KB; read & program granularity
- Higher density
- slow (sequential access) but page 단위로 큰 data에 접근하여 page 전체 한번에 read
![500](https://i.imgur.com/D6GsmPI.png)

### NAND vs NOR

![500](https://i.imgur.com/WonbKFz.png)

- NAND
	- high density
	- fast write, erase
	- slow read
	- used for mass storage
- NOR
	- byte-level random access
	- fast read
	- slow write & erase
	- used for program ROMs
### Issues in FLASH
- Block granularity erasure
	- 0 cell $\to$ entire block erasure $\to$ 1 cell
	- even NOR cannot offer random access erasure
- Memory wearing ; endurance
	- Finite number of program-erasure cycles
	- Wear leveling
		- Spread writes to each block
		- Use sparse blocks
		- Trigger garbage collection in background
- Read disturbance
	- Many reads on a NAND cell changes the value of neighbor cells
	- need ECC 
![500](https://i.imgur.com/qySmYN1.png)

### Denser FLASH
- SLC (single level cell)
	- two levels of current 0 or 1
- MLS (multi level cell)
	- N levels of current = $\log_{2}N$ - bits logical states
	- = N threshold voltages based on 4 different levels of charges collected on FG
	- ECC is more critical than in SLC

## SSD : Solid State Disk
**FLASH based disk**
![600](https://i.imgur.com/KMh6a6G.jpg)

![500](https://i.imgur.com/XEs8aX9.png)
- Pages : 512B ~ 4KB
- Blocks : 32 ~ 128 pages
- read/write page granularity
- page can be written only after its block has been erased
- block wears out after 10k writes (erase 수반됨)

![300](https://i.imgur.com/gBgEEuQ.png)
>SSD vs HDD

