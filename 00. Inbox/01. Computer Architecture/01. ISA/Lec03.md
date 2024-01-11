- special Register
	- PC
	- stack pointer
32bit computer
pc ➡️ pc + 4
64bit computer
pc ➡️ pc + 8

inst 실행 ➡️ archi state가 변화
add R1 R2 R3 실행 ➡️ R3 값과 PC 값 변화

>[!question] Why register is necessary?
>memory access ⬇️ -> performance ⬆️
>compiler 입장에서 메모리 접근이 줄기 때문에 코드 길이가 줄어든다.

>[!question] Can ALU operands be in memory?
>No in stored program architecture
>Then use load - store architecture ➡️ load memory value to register and use register value as operand

>[!question] Why CISC ➡️ RISC
>- resolve length of inst issue with development of compiler
>- resolve performance with development of clock frequency

CISC는 가장 복잡한 inst에서 걸리는 시간을 기준으로 CPI가 정해져 clock frequency가 제한된다. 하지만 RISC는 간단한 inst들로 쪼개져 있기 때문에 CPI가 짧고 clock frequency가 빨라질 수 있다.
>[!question] Why intel still use CISC
>binary compatibility
>CISC ➡️ RISC 내부적으로 RISC CPU 가지고 있음
>Hardware translator which translate CISC to RISC is implemented inside

Big Endian | Little Endian
--- | ---
sign | even or odd
> advantage Big Endian vs Little Endian

