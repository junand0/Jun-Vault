## Exception
CPU must prepare for *unplanned* events
- Case of unplanned events
	- inst fail or cannot complete
		- division by zero, overflow
	- External I/O device
	- Quantum expiration in a time-shared system
		- context switching
- how to handle unplanned events?
	- write every program with continuous check(*polling*) for every possible contingency
		- acceptable for simple embedded systems such as toaster
	- write program as if nothing unusual happens
		- but must detect exceptional conditions in HW
		- transparently transfer control to exception handler who knows how to resolve the exception and then back to program
### Interrupt Control Transfer
- Interrupt is an unplanned function call to system (*interrupt handler*)
- Unlike normal function call, cannot anticipate the control transfer or prepare it 
- Contorl is returned to the main thread (interrupted inst) later
*control transfer to the interrupt handler and back must be transparent to the interrupted thread*
$\to$ context switching must be done implicitly
### Type of Interrupts
- Synchronous Interrupts (*exceptions*)
	- during the operation of inst, inside program
	- exceptional conditions tied to a particular inst
	- ex) illegal opcode, illegal operand, virtual memory management faults
	- No forward progress unless handled *immediately*
- Asynchronous Interrupts (*interrupts*)
	- by external components
	- external events no tied to inst
	- ex) I/O events, timer over
	- Some *flexibility* on when to handle $\to$ delay allowed utill resolved, but not forever
- System Call / Trap Inst
	- inst whose purpose is to raise an exeption only
	- by user
	- ex) "printf" function
### Interrupt Sources
![400](https://i.imgur.com/Ma381bz.png)
>Exceptions or System Call / Interrupts

### Privilege Levels
![400](https://i.imgur.com/8JVwblM.png)
#### Control and Privilege Transfer
![400](https://i.imgur.com/Ai9tym6.png)
when interrupt,  user mode $\to$ privileged mode (OS kernel)
when return from OS to user mode, privileged mode $\to$ user mode

## Implementing Interrupts
### Precise interrupt / exception
![400](https://i.imgur.com/FcwEdsu.png)
- *Sequential Semantics*
	- 하나씩 순서대로 실행되는 것처럼 보여야함
- older inst than interrupted inst must be finished completely
	- after finishing older inst, transfer control to handler
- younger inst than interrupted inst must be as if never happened
	- younger inst는 pipeline에 들어오지 않은 것처럼 보여야 함
- On synchronous exception은 바로 끊고 transfer to handling
- On asynchrounous interrupt는 여유를 두고 transfer to handling 가능
	- cycle frequency가 ns scale이기 때문에 유저가 인지하지 못한다
### Exception Sources
![500](https://i.imgur.com/HFvAwyj.png)
protection fault : 권한에 벗어남
ex) read only인데 write 하려 함
### Pipeline Flush for Exceptions
![600](https://i.imgur.com/ibh1UXZ.png)
## MIPS Interrupt Architecture
![500](https://i.imgur.com/LADGWFH.png)
- *EPC - CR14* 
	- : return address after handling
- *interrupt mask*
	- interrupt의 우선 순위 표시
	- 현재 handling하고 있는 interrupt보다 우선 순위가 낮은 interrupt는 handling에 들어가지 않고 무시 $\to$ dead lock 방지
- On an interrupt transfer, the CPU hardware saves the interrupt address to *EPC*
	- overwritten *immediately*, not leave frozen in the PC
	- cannot use *r31*(conserved to user's return address) as in a function call
		- transparent : user level에서 handling이 노출되어서는 안된다.
- CPU HW must save information that cannot be saved and restored in SW by interupt handler
	- GPR can be managed in SW by the interrupt handler using a callee-saved
	- r26, r27 are reserved for OS kernel to be available immediately by kernel at the start and end of an interrupt handler
### Interrupt Servicing
- On an interrupt transfer, CPU HW records the cause of interrupt in a privileged register
	- *interrupt cause register - CR13*
- Method 1. Control is transferred to a **pre-fixed default interrupt handler address**
	- *flexible, but slow*
	- this initial handler exmines the cause $\to$ branch to the appropriate handler subroutine
	- protected from user-level $\to$ cannot branch to it by user code
- Method 2. **Vectore Interrupt**
	- *not flexible, but fast*
	- Bank of privileged registers holds separate specialized handler address for each interrupt source (cause)
	- HW transfer control directly to the appropriate handler $\to$ *save interrupt overhead*
#### Example of Causes
![500](https://i.imgur.com/xGdaorA.png)

#### Handler Examples
- On asynchronous interrupts, device specific handlers are invoked to service the I/O devices
- On exceptions(synchronous interrpts), kernel handlers are invoked
	- resolve the faulting condition and continue the program
		- ex) emulate the missing FP functionality, update virtual memory management
	- signal back to the user process if a user-level handler function is registered
	- Flush the process if the exception cannot be resolved
- System call is a function call from user process to kernel service routine
#### Returning from Interrupt
- **SW restores all architectural states saved at the start of the interrupt handling**
- (1) MIPS32 uses a special jump inst
	- ERET
	- atomically restore the automatically saved CPU states
	- atomically restore the privilege level (kernel $\to$ user)
	- atomically jump back to the interrupted address in EPC
- (2) MIPS R2000 uses a pair of insts
	- mfc0 r26 epc : copy EPC to r26
	- jr r26 : jump to a copy of EPC in r26 (r26 is conserved for OS kernel)
	- rfe : restore from exception mode *(restore archi state)* and must be used in the *delay slot!*
##### Example
![500](https://i.imgur.com/QLCp1DN.png)

![500](https://i.imgur.com/FusxugV.png)
>sw and lw $\to$ callee save
>rfe $\to$ restore archi state
### Nesting Interrupts
- On an interrupt control transfer, exceptions or asynchronous interrupts are *disabled automatically*
	- *Issue* : Another interrupt would overwrite the EPC, Interrupt Cause Register and Interrupt Status Register
	- (1) handler must be carefully written not to generate synchronous interrupts itself
	- (2) handler must disable further assync interrupts using *Interrupt State Register* (CR12)
		- Interrupt Mask
	- However, for long running handlers interrupt must be re-enabled not to miss critical interrupts
		- handler must save EPC, Interrupt Cause/Status Register to *memory stack* before re-enable async interrupt
		- Once interrupts are re-enabled $\to$ *EPC, Interrupt Cause/Status Register is updated* by the next interrupt handler
#### Interrupt Priority
- *Async interrupt* sources are ordered by *priorities*
	- Higer priority $\to$ more timing critical
	- muliple interrupts are triggered $\to$ handles the highest priority interrupt first
- Interrupts from different priorities are *selectively disabled* by setting the *mask in the Status Register*
- Handler only *re-enable higher priority* interrupts
	- re-enable same or lower priority interrupts lead to *infinite loop (dead lock)* if device interrupts are repeated
#### Nestable Handler
![500](https://i.imgur.com/SdCvAOQ.png)
>set priority of interrupt $\to$ prevent dead lock
