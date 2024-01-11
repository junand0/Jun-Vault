## Single Cycle Datapath Analysis
- Matches naturally with the sequential and atomic semantics inherent to most ISAs
	- Initiate programer visible state one to one
	- Map instructions to a next state combinational logic
- Limitation
	- clock frequency is limited by critical delay instruction that is *Load* through all 5 steps
![|500](https://i.imgur.com/gn7RKBJ.png)

## Multi Cycle Implementation
![600](https://i.imgur.com/5DCL6os.png)
>PVS updates only at the end of instruction's cycle - sequence
>1 cycle과 다르게 clock이 끝날 때마다 PVS update 하지 않음
>instruction's effect(result) is still combinational from PVS to PVS

![500](https://i.imgur.com/GjnazXg.png)

#### MicroSequencer
**What about the rest of the control signals? —› MicroSequencer**
![500](https://i.imgur.com/oslq920.png)
>Lookup Table which determines state transition where to go next in FSM

![500](https://i.imgur.com/Sc9FKht.png)

#### Micro-code controller
![500](https://i.imgur.com/CdptZ7H.png)
- $\mu$PC indexed into a $\mu$program ROM to select an $\mu$instruction
- Field of the $\mu$instruction maps to control signals
- this is $\mu$Architecture, so don't expose to outside then not support $\mu$program-visible state
## Performance Analysis
![500](https://i.imgur.com/GsOGycP.png)
>nice lower T
>nice greater MIPS

![500](https://i.imgur.com/sftwdMt.png)
>1667MHz = 1/600ps of LW

## Reducing Datapath(ALU) by Sequential Reuse
![600](https://i.imgur.com/POPoEOC.png)

Add IR(inst register), MDR(memory data register),  A, B latch and ALUout
to make timing difference btw IF/ID and PC update (PC=PC+4)

![300](https://i.imgur.com/1QJLTbx.png)
![300](https://i.imgur.com/ROBVUKS.png)
![300](https://i.imgur.com/0Af2iZ1.png)

#### Synchronous Register Transfers
- Synchronous state with latch enabled by control
	- PC, IR, RF, MEM
- Synchronous state that always latch
	- A, B, ALUOut, MDR
![500](https://i.imgur.com/il9YZ89.png)

![500](https://i.imgur.com/VWrrYS4.png)

![500](https://i.imgur.com/kGWVjiQ.png)
>if A == B —› branch

![500](https://i.imgur.com/dLMmwBN.png)

## Horizontal Microcode
![500](https://i.imgur.com/8RglX1O.png)

Microcode memory의 addr 역할을 하는 uPC를 inst와 이전 uPC 정보 등을 바탕으로 결정
uPC 주소에 있는 Microcode memory data는 control signal로 역할하게 됨

![500](https://i.imgur.com/nj9sxrx.png)

uPC를 찍으면 Micorcode stroage에서 m개의 1bit 신호 조합이 나온다. 조합에서 각 1bit signal들은 해당되는 RT의 실행 여부를 의미한다. m개의 1bit 신호 조합을 input으로 받으면 ROM에서 이를 토대로 control signal을 결정하고 output으로 내보낸다.

