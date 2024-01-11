## Data Format
#### Most things are 32 bits (word)
- Instruction and data addresses
- Signed and unsigned integers
- Just bits
#### Also 16 - bit word and 8 - bit word
#### Floating - point numbers
- IEEE standard 754
- Single precision
	- 1-bit sign
	- 8-bit exponent
	- 23-bit fraction
- Double precision
	- 1-bit sign
	- 11-bit exponent
	- 52-bit fraction
#### Big Endian vs Little Endian
![|500](https://i.imgur.com/xhoj42L.png)
Big Endian | Little Endian
---|---
sign (pos or neg) | even or odd

## Instruction Formats
#### R-type
- 3 register operands
![|500](https://i.imgur.com/zhxcHbT.png)
#### I-type
- 2 register operands and 16-bit immediate operand
![|500](https://i.imgur.com/VxWeLAT.png)
#### J-type
- 26-bit immediate operand
![|500](https://i.imgur.com/sd2KODS.png)
#### Simple Decoding
- 4 bytes per inst regardless of format
- must be 3-byte aligned *(LSB 2bit of PC must be 2b'00)*
- Format and fields readily extractable

## ALU Instructions
#### R-type
- *Assembly*
	- ADD $rd_{reg}$ $rs_{reg}$ $rt_{reg}$
- Machine encoding
![|500](https://i.imgur.com/XyVUCe1.png)
- *Semantics*
	- GPR\[rd\] ‹— GPR\[rs\] + GPR\[rt\]
	- PC ‹— PC + 4
- *Exception handling*
	- overflow
- *Variations*
	- Arithmetic 
		- {signed, unsigned} $\times$ {ADD, SUB}
	- Logical
		- {AND, OR, XOR, NOR}
	- Shift
		- {Left, Right-Logical, Right-Arithmetic}
#### I-type
- *Assembly*
	- ADDI $rt_{reg}$ $rs_{reg}$ $immediate_{16}$
- Machine encoding
![|500](https://i.imgur.com/KN5oUbl.png)
- *Semantics*
	- GPR\[rt\] ‹— GPR\[rs\] + sign-extend (immediate)
	- PC ‹— PC + 4
- *Exception handling*
	- overflow
- *Variations*
	- Arithmetic
		- {signed, unsigned} $\times$ {ADD, ==~~SUB~~==} (음수 더하면 됨)
	- Logical
		- {AND ,OR, XOR, LUI}
##### Assembly Programming
*High level Code*
```c
f = (g+h) - (i+j)
```
*Assembly Code*
- suppose f, g, h, i, j are in $r_{f}, r_{g}, r_{h}, r_{i}, r_{j}$
- suppose $r_{temp}$ is a free register
```shell
add r_temp r_g r_h # r_temp = g + h
add r_f r_i r_j # r_f = i + j
sub r_f r_temp r_f # f = r_temp - r_f
```

#### Load Instructions
- *Assembly*
	- LW $rt_{reg}$  $offset_{16}$ ($base_{reg}$)
	- Addr = base_reg에 저장된 값 + offset
	- rt_reg ‹— MEM\[Addr\]
	- load 4 byte word
- *Machine encoding*
![](https://i.imgur.com/pinZpVA.png)
>I - type
- *Semantics*
	- effective_address = sign-extend(offset) + GPR\[base\]
	- GPR\[rt\] ‹— MEM\[translate(effective_address)\]
		- translate virtual address to physical address
	- PC ‹— PC + 4
- *Exceptions*
	- Address must be "word - aligned"
	- MMU(Memory Management Unit) exceptions ⬅️ Let's handle unexpected behaviors

>[!question] What if you want to load an unaligned word?
>![|300](https://i.imgur.com/3lXy30f.png)
**⬇️** **⬇️** **⬇️**

##### Data Alignment
![|300](https://i.imgur.com/sKpKMZA.png)
>Big Endian
- *LW/SW alignment restriction*
	- Not optimized to fetch memory bytes not within a word boundary
	- Not optimized to rotate unaligned bytes into registers
- *Can use separate opcodes for the infrequent case*
![|500](https://i.imgur.com/E52QMFW.png)
>LWL : loaded to MSB of dst. register
>LWR : loaded to LSB of dst. register
>➡️ align process

#### Store Instructions
- *Assembly*
	- SW $rt_{reg}$  $offset_{16}$ ($base_{reg}$)
	- store 4 byte word
- *Machine encoding*
![|400](https://i.imgur.com/ejYNjEz.png)
>I-type
- *Semantics*
	- effective_address = GPR\[base\] + sign-extend(offset)
	- MEM\[translate(effective_address)\] ‹— GPR\[rt\]
	- PC ‹— PC + 4
- Exceptions
	- address must be "word aligned"
	- MMU exceptions

##### Assembly Programming
- High level code
```c
A[8] = h + A[0] // A is an array of integers (4 byte each)
```
- Assembly Code
	- suppose &A, h are in $r_{A}, r_{h}$
	- suppose $r_{temp}$ is a free register
![|500](https://i.imgur.com/rNk3VyL.png)

#### Load Delay Slots (old days...)
![|200](https://i.imgur.com/VyAoFiM.png) ➡️[slow memory access] ![|200](https://i.imgur.com/3WDGQcT.png)
 
>[!question] Why Load Delay Slots are needed?
>probability using old register value of past cycle
>$\because$ slow memory access or hardware feature
>➡️ allocate bubble instruction(~~cycle~~) using delay slot ➡️ process safely and precisely
>but slow and bad performance wasting cycle

- R2000 load has an architectural latency of 1 *inst\**
	- The inst immediately following a load (in the "delay slot") still sees the old register value
	- The load instruction no longer has an atomic semantics
	- notice the latency is defined in *"instruction"* not cycle or sec
>[!question] Is this a good idea?
>- compiler가 delay slot으로 empty instruction을 넣어주는데 그럼 나중에 memory access가 더 느려졌을 때 delay slot으로 empty inst를 2개, 3개... 몇 개를 사용해야 할 지 애매해짐
>- R4000 redefined LW to *complete atomically*

## Control Flow Instructions
![|500](https://i.imgur.com/6ThBdiV.png)

#### Conditional Branch Instructions
- *Assembly*
	- BEQ $rs_{reg}$  $rt_{reg}$  $immediate_{16}$
- *Machine encoding*
![|500](https://i.imgur.com/HS07njE.png)
>I-type
- *Semantics*
	- target = PC + sign-extend(immediate) $\times$ 4 
		- ($\times 4$ = << 2b)
		- imm는 *명령어(word) 단위*이고 PC는 byte 단위이기 때문에 imm을 *word ➡️ byte* 단위로 바꿔줘야함
		- *PC 기준으로 $-2^{15} \sim (2^{15}-1)$개의 명령어*(word)만큼 branch 가능 (MSB 1bit는 sign bit)
	- if GPR\[rs\] == GPR\[rt\]
		- then PC ‹— target
		- else PC ‹— PC + 4
- *Variations*
	- BEQ : branch if equal
	- BNE : branch if non equal 
	- BLEZ : branch if less than or equal to zero 
	- BGTZ : branch if greater than zero
>[!question] Why isn't there a BLT or BGT inst
>complicated and slow than BLEZ and BGTZ
>두 값을 빼서 0보다 큰 지 작은 지 비교하면 됨

>[!question] How far can you jump?
>PC 기준으로 $-2^{15} \sim (2^{15}-1)$개의 명령어(word)만큼 branch 가능
>($2^{15} \times 4\ byte$)


#### Jump Instructions
- *Assembly*
	- J $immediate_{26}$
- *Machine encoding*
![|500](https://i.imgur.com/8CqA4Qb.png)
>J-type

![|500](https://i.imgur.com/BxgenME.png)

- *Semantics*
	- target = PC\[31:28\] $\times 2^{28}|_{bitwise\ or}$ zero-extend(immediate) $\times 4$
	- PC ‹— target
- *Variations*
	- JAL : jump and link
	- JR : jump registers
>[!question] How far can you jump (in both direction)?
>PC가 포함되어 있는 임의의 256MB 안에 어디로든 jump 가능

##### Assembly Programming
- High level code
```C
if(i == j) then
	e = g
else
	e = h
f = e
```
>Sudo code
![|150](https://i.imgur.com/VOLp6gr.png)
- *Assembly code*
	- suppose e, f, g, h, i, j are in $r_{e}$, $r_{f}$, $r_{g}$, $r_{h}$, $r_{i}$, $r_{j}$
![|500](https://i.imgur.com/mjcZBLn.png)

#### Branch Delay Slots
- R2000 branch instructions also have an architectural latency of 1 *instructions*
	- the instruction immediately after a branch is always executed (in fact PC-offset is computed from the delay slot instruction)
	- branch target takes effect on the $2^{nd}$ instruction
![|500](https://i.imgur.com/v7FvN4E.png)
> add $r_{e}$ $r_{g}$ $r_{0}$ 는 j L2와 상관없이 무조건 실행되기 때문에 compiler가 code 순서를 바꿔서 nop대신 넣은 binary로 compile함

##### Strangeness in the Semantics
>[!question] Where will you end up?
>![|200](https://i.imgur.com/takNOrP.png)
>j L1 실행 전에 다음 inst 하나 실행하고 j instruction 실행하려 함
>그런데 다음 inst가 또 j inst여서 문제 발생

#### Function Call and Return
- *Jump and Link*
	- JAL  $offset_{26}$
	- return address = PC + 8  ($\because$ branch delay slot)
	- target = PC\[31:28\] $\times 2^{28}|_{bitwise\ or}$ zero-extend(immediate) $\times 4$ 
	- PC ‹— target
	- GPR\[r31\] ‹— return address
	- *On a function call, the callee need to know where to go back to afterward*
- *Jump Indirect*
	- JR  $rs_{reg}$
	- target = GPR\[rs\]
	- PC ‹— target
	- PC - offset jumps and branches *always jump to the same target* every time when the same inst is executed
	- Jump Indirect allows the same inst to *jump to any location specified by rs* (usually r31)

##### Assembly Programming
![](https://i.imgur.com/WqNnhMV.png)
>... A ➡️[call] B ➡️[return] C ➡️[call] B ➡️[return] D ...

>[!question] How do you pass argument between caller and callee?
>using stack

- if A set r10 to 1, what is the value of r10 when B returns to C
- What registers can B use?
- What happends to r31 if B calls another function

##### Caller and Callee Saved Registers
- *Callee - Saved Registers*
	- Callee saves the old values of registers to memory which should not change when returned to Caller
	- and restore them before return to Caller
- *Caller - Saved Registers*
	- When Callee return to Caller the registers will not keep same values
	- So Caller save the registers itself on *stack* before call Callee
![|700](https://i.imgur.com/BhNV4Y7.png)
> Caller save & Callee save

![|500](https://i.imgur.com/m8v4fo8.png)
