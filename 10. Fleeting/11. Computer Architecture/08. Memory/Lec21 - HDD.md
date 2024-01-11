## HDD
### HDD Component
![400](https://i.imgur.com/FfM6oGZ.png)
- Cylinder
	- 동일한 line의 track 집합
### DISK Geometry
- *Disks* consist of *platters*
- each *platter* consist of two *surfaces*
- each *surface* consists of concentric rings called *tracks*
- each *track* consists of *sectors* separated by *gaps*
![400](https://i.imgur.com/To5nAtx.png)

![400](https://i.imgur.com/UZ0w36h.png)

### Magnetic Bit
- Bit cell composed of magnetic grains
	- 50~100 grains per bit
- 0
	- Region of grains with uniform magnetic polarity
- 1
	- Boundary between regions of opposite magnetization
![300](https://i.imgur.com/QQ9dYOR.png)

![500](https://i.imgur.com/EdTZ04p.jpg)
>Longitudinal Recording

![500](https://i.imgur.com/91ocTbO.jpg)

### HDD Organization
- $power\propto(\#\ platters)(RPM)^{2.8}(Diameter)^{4.6}$
	- Trade off between RPM and Diameter
- Read/Write head
	- reading : Faradays's Law
	- writing : Magnetic Induction
- Data channel
	- encoding of data to magnetic phase changes
	- decoding of data from magnetic phase changes
### Computing Disk Capacity
*Capacity* = (\# bytes/sector) x (avg. \# sectors/track) x (\# tracks/surface) x (\# surfaces/platter) x (\# platters/disk)
### Storage Density
- Density Metrics
	- Linear density (BPI) = bits/inch
	- Track density (TPI) = Tracks/inch
	- Area Density = BPI x TPI
![200](https://i.imgur.com/ga8z7nU.png)

### Disk Access
![500](https://i.imgur.com/5AsdcOX.png)
>Service Time components

#### Seeking
- Seek time depends on
	- arm actuator motor의 관성력
	- Distance between outer and inner disk recording radius (platter diameter)
- Components of a seek
	- 1. Speedup
		- Arm accelerates
	- 2. Coast
		- Arm moving at maximum velocity
	- 3. Slowdown
		- Arm brought to rest near desired track
	- 4. Settle
		- Head is adjusted to reach the desired location
- Seek Time
	- Very short seeks (2~4 cylinders)
		- Settle time dominates
	- Short seeks (200~400 cyllinders)
		- Speedup and slow down time dominates
	- Longer seeks
		- Coast time dominates
	- Higher TPI with smaller platter sizes
		- Settle time become more important
### Internal Data Rate (IDR)
- Rate data can be read/written from physical media 
- IDR is determined by
	- linear density ; BPI
	- platter diameter
	- rotational speed ; RPM
	- IDR = BPI x platter diameter x RPM
### HDD Power
![500](https://i.imgur.com/sfChTGm.png)

### Redundant Array of Independent Disk : RAID
#### RAID 0
![600](https://i.imgur.com/9cDJBwb.png)
- Striped Disk Array without fault tolerance
- *Block level striping* without parity or mirroring
	- space efficiency : 1
	- no fault tolerance
	- high bandwidth
#### RAID 1
![600](https://i.imgur.com/bQEYr4T.png)
- Mirroring & Duplexing
	- without parity or striping
	- space efficiency : 1/2
	- survive a single disk failure
	- cannot resolve B != B issue, only detection possible
#### RAID 2
![600](https://i.imgur.com/GOm8Glq.png)

- Hamming Code ECC
- Bit-level striping with SEC code
- Space efficiency : Hamming code overhead
- Correct 1 bit error
- Can survive a single node failure
- Must access all disks for a single write

#### RAID 3
![600](https://i.imgur.com/BfZTZ4s.png)

- bit interleaved parity
- Bit level striping with parity
	- only detect single bit error possible, cannot correct
- Space efficiency : $\frac{n-1}{n}$
- Can survive a single node failure
- Must access all disks for a single write

#### RAID 4
![600](https://i.imgur.com/Dt8SzP0.png)
![600](https://i.imgur.com/ADRkK6f.png)

- Block Interleaved Parity
- RAID 2, 3처럼 한 block을 쓰기 위해 모든 block에 접근하지 않아도 됨 대신 write할 block과 parity block의 old value를 읽어야 함
- Block level striping with Parity
- Space efficiency : $\frac{n-1}{n}$
- survive a single node failure
- parity disk can become a traffic bottleneck

#### RAID 5

![600](https://i.imgur.com/DNCNvEh.png)

- Block Interleaved Distributed Parity
- Block level striping with distributed parity
- RAID 4는 parity disk가 항상 바쁘다 $\to$ distribute parity bit $\to$ parity generation does not create bottleneck

#### RAID 6
![600](https://i.imgur.com/zirk86n.png)

- Independent Data Disks with two independent parity schemes
- Block level striping with double distributed parity
- Add additional parity block to RAID 5
- Space efficiency : $\frac{n-2}{n}$
- Surviving two different kinds of failures
