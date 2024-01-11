## Preview
![|600](https://i.imgur.com/17cotto.png)
>Computer System Abstractions

![|500](https://i.imgur.com/QesWzpH.png)
>There exists *object files* for each of functions called by function call
>linker does link include files

```shell
//Programs are translated and linked using a compiler driver:
unix> gcc -02 -g -o p main.c swap.c
unix> ./p
```
![|500](https://i.imgur.com/pkpsp3e.png)
>*relocatable*
	"Feature" that the locations of programs or files are not important
> fully linked ➡️ executable object file

>[!question] How to execute programs which include other programs inside
>![|600](https://i.imgur.com/Du9hi2h.png)
>*Relocatable Object Files ➡️[Link] Executable Object File*
>Relocatable Object Files 
>	: Don't know the location of other programs yet
>Executalbe Object File 
>	: Know the location of other programs

![|600](https://i.imgur.com/2UoQ1rK.png)
>Executable Object File is loaded virtual memory space
>stack & hip info is stored in virtual memory space for using to execute program

>[!question] n-bit computer?
>1. 계산 단위가 n-bit
>2. $2^n$ bit memory space

>[!question] Which part set virtual memory address?
>?

## Stack
[stack frame REF](https://20plus3.tistory.com/52)

![|300](https://i.imgur.com/nn2ccSg.png)
> when finish function and return 
> %esp go to %ebp
> %ebp go to old %ebp *(should remember old %ebp)*
#### Current Stack Frame
- Argument build
	- parameters for function *about to call*
- Local variables
	- if can't keep in registers
- Saved register context
	- caller saved
	- callee saved
- Old %ebp
	- old frame pointer
#### Caller Stack Frame
- Return address
	- pushed by *"call"* instruction
- Arguments
	- for current function

## Architecture
describe the attributes of a system *as seen by the programmer*
➡️ *conceptual structure* & *functional behavior* as ==distinct from== the organization of the data flow, controls, logic design and physical implementation.

#### Architecture Level
- Car
	- driving manual & operation manual
	- you don't have to be a car mechanic to drive a car
- Computer
	- you don't have to be a circuit designer to use a computer
#### Microarchitecture Level (implementation)
A particular computer design has a certain configuration of datapath and control logic units
ex) 4M cache vs 1M cache
#### Circuit Level (physical implementation)
gates, wries, semiconductor, transistors

## ISA
**User's manual for the computer**
#### Programmer Visible State (Architectural State)
![|400](https://i.imgur.com/2Ty53n4.png)

>[!cite] *Context switching*
>OS가 architecture state을 기억해두고 프로그램을 여러 개 넣었다 뺐다 하며 실행함
>사용자가 보기에는 여러 프로그램이 동시에 실행되는 것처럼 보임

#### Memory Usage Convention
![|400](https://i.imgur.com/tGMGF5h.png)

#### Stored Program (Von Neumann) Architecture
##### Stored - program Architecture
- instructions in a linear memory array
	- insts are in memory
- Instructions can be modified just like data
	- Both forms consist of 0s and 1s
##### Sequential instruction processing
PC identified the current inst.
➡️ Inst is fetched from mem and executed
➡️ PC is advanced (PC+4 or branching according to inst)
Repeat...
##### Von Neumann vs Dataflow
>[!info] *Von Neumann bottleneck*
>CPU processing(computation)보다 memory 속도가 너무 느림

![|200](https://i.imgur.com/LShQHJF.png)
>위의 연산을 실행할 때
- Von Neumann
	- Memory에서 inst 5번 가져와서 실행해야함
- Data Flow
	- Instruction ordering specified by *dataflow dependence*
	- No PC
	- inst can be executed when all of operands are received
	- each inst specifies who should receive results
![|100](https://i.imgur.com/f9hXtM0.png)
>3 cycle computation

>[!question] Which one is better?
>- Von Neumann
>	- Von Neumann bottleneck ➡️ slow
>- Data Flow
>	- Fast
>	- if prgram change ➡️ hardware should be changed
>	- program data flow가 변하지 않고 input 값만 바뀌는 경우 유리함

#### What are specified/decided in an ISA?
- 1. Data format and size
	- character, binary, decimal, floating point, negatives...
- 2. **Programmer Visible State** (architectural state)
	- memory, registers, PC...
- 3. Instructions
	- how to *change* the ==Programmer Visible State==
	- what to perfrom and what to perform next
	- where the operands are
- 4. Instruction to binary (or vice versa) encoding
- 5. How to interface with the outside world?
- 6. Protection and Privileged operations
- 7. Software conventions
*very often you compromise immediate optimality for future scalability and compatibility*
#### General Instruction Classes
##### Arithmetic and Logical operations - add, sub, and, or
1. Fetch operands from specified locations
2. Compute a result as a function of operands
3. Store result to a specified location
4. Update PC to the next sequential instruction
##### Data movement operations - move, load, store
1. Fetch operands from specified locations
2. Store operand values to specifed locations
3. Update PC to the next sequential inst
##### Control flow operations - beq, j
1. Fetch operands from specifed locations
2. Compute a branch condition and a target addr
3. if branch condition is true, then PC ‹— target addr **/** else next sequential inst

#### Register Architecture
![|500](https://i.imgur.com/iTEgeLI.png)
>Accumulator architecture

![|400](https://i.imgur.com/T9WoTs9.png)
>a ➡️[En1] f ➡️[En2] a+1 ➡️ b ➡️[En1] f ➡️[En2] a+1 // But we should return to b+1
>A n1 ➡️[En2] a ➡️ A n1 // but we should get A n1+1 not A n1

>[!question] How to fix it?
>*코드에 숫자를 사용하지 않고 register를 사용하자*
>compile ➡️ binary가 나와버리면 숫자의 경우 바꿀 수가 없지만 
>register를 이용하면 코드를 재사용하면서 메모리에서 한 번  읽어 온 값에서 address를 계속 전진시킬 수 있다

##### Accumulator
ACC에서 1씩 더해감
##### Accumulator + address registers
- need *register indirection*
- initially address registers were special purpose
	- ie. can only be loaded with an address for indirection
- Eventually *arithmetic on addresses* became supported
![|200](https://i.imgur.com/6bDLhpA.png) ![](https://i.imgur.com/xS9Pal1.png)
##### General Purpose registers - GPR
All registers good for all purposes
*register가 많을 수록 메모리에 가는 횟수가 줄어들기 때문에 코드 길이가 줄어들어 컴파일러 입장에서 좋고  메모리가 느리기 때문에 적게 갈 수록 성능이 좋아짐*

>[!question] Can ALU operands be in memory?
>Yes - CISC
>No in *load-store architecture* - RISC
>➡️ load를 통해 memory의 값을 register에 읽어와서 ALU 연산을 해야하기 때문에 1줄짜리 코드가 3줄로 늘어남

>[!question] Why CISC ➡️ RISC?
>1. *Compiler의 발전*으로 high level language로 코드를 짜주면 compiler가 필요한 명령들을 만들어내기 때문에 어차피 사람이 쓰는 코드는 짧다.
>2. *CLK frequency의 발전*
>RISC는 명령들이 모두 단순하기 때문에 각 명령을 실행하는데 걸리는 시간이 매우 짧아 clk frequency가 매우 짧음 
>그러나 CISC는 worst case의 latency에 dependency가 있는 하드웨어로 설계되었기 때문에 clock frequency가 비교적 길 수 밖에 없음

![|500](https://i.imgur.com/ObI7chs.png)

#### MIPS ISA
- Triadic
	- 3 operands : 2 input 1 output
- Simple instructions
	- ➡️ decoding이 쉽고 빠르다 - binary 해석
![|500](https://i.imgur.com/K52egDd.png)

>[!question] Why Intel still use CISC?
>To keep *binary compatibility*
>이전의 binary 코드가 cpu에서 동작해야하기 때문에 architecture가 유지되어야함

>[!question] 그런데 어떻게 성능이 보장될까?
>CISC 코드를 RISC 코드로 변환해주는 하드웨어가 내부에 있음
>결국 내부적으로는 RISC를 사용함

#### Evolution of ISA
##### Why were the earlier ISAs so simple?
- ex) EDSAC
- Technology limitation
- Inexperience, lack of precedence
##### Why did it get so complicated later?
- ex) VAX11
- Assembly programming - compiler가 좋지 않아서 assembly programming이 힘듦
- Lack of memory size and performance
- Microprogrammed implementation - friendly to program
##### Why did it become simple again?
- Memory size and speed (cache)
- Compilers
#### Design ISA
- we don't want to change
- Permit future extensions by “reserving” spare bits in instruction encoding
