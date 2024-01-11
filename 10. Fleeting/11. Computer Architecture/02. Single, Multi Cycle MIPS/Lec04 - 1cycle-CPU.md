## Context Switching
OS에서 여러 개의 architectural state(Program Visible state)를 바꿔가며 실행하고 저장하는 것을 반복하는 과정이다. 이로 인해 유저에게는 마치 여러 개의 프로그램이 동시에 실행되는 것처럼 보인다.
## Magic Memory and Register File
Combinational Read
register file의 Read는 async하게 일어난다.
Synchronous Write
register file의 write는 sync하게 이뤄진다.
## Instruction Processing
*5 steps*
- IF
	- Instruction Fetch (from memory)
- ID
	- Instruction decode and operand fetch
- EX
	- ALU / execute
- MEM
	- Memory access
	- only for load and store inst. 
- WB
	- Write - Back
	- when load instruction
## Single Cycle Datapath for Arithmetic and Logical Inst.
### Datapath for R and I-type ALU inst.
![|500](https://i.imgur.com/NLPVzZn.png)

In ID step, determine select signal of RegDest and ALUSrc and ALUOp from the instruction.
If the inst. is I-type, then the rt field value is pushed to Write register addr of rf and imm field value is pushed to ALU input through the sign-extend processing.
- IF
- ID
	- select signal of RegDest, RegWrite, ALUSrc and ALUOp
- EX
	- ALU
- WB
	- write back result of ALU to rf
## Single-Cycle Datapath for Data Movement Inst.
### Load inst.
![|500](https://i.imgur.com/lRcNYNS.png)
- IF
	- Load instruction is fetched
- ID
	- Determine seleciton signal of RegDest, ALUSrc, RegWrite, MemRead and ALUOp
- EX
	- caculate effective addr by read data from rf at rs + sign-extended imm through ALU
- MEM
	- read mem data from effective addr
- WB
	- write back mem data to register at rt
*critical delay instruction*
*processing all 5 steps*
### Store inst.
![|500](https://i.imgur.com/vCBCig9.png)

- IF
	- Store inst. is fetched
- ID
	- Determine selection signal of RegDst, RegWrite, ALUSrc, MEMWrite and ALUOp
- EX
	- caculate effective addr by sign-extended imm + read data from rf at base
- MEM
	- Write read data from rf at rt to effective addr of memory
### Datapath for Non-Control Flow Inst.
![|500](https://i.imgur.com/e7sJKgD.png)
> without jump inst.
> Current always flows through all of parts of circuit!

## Single-Cycle Datapath for Control Flow inst.
### Unconditional Jump Datapath
![|600](https://i.imgur.com/KqQYTST.png)

- IF
- ID
	- Make PCSrc signal
	- {MSB 4b of PC, imm, 2'b00} is next pc

### Conditional Branch inst.
![|600](https://i.imgur.com/o2vXOnh.png)

- PCSrc
	- 1. pc+4
	- 2. beq : pc + sign-extended imm
	- 3. JR
	- 4. J : {MSB 4b of PC, imm, 2'b00}
## Control
![|](https://i.imgur.com/d2KS3Oa.png)

