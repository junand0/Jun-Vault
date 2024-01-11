## Data Dependency
![500](https://i.imgur.com/QqDRxXh.png)
- RAW
	- young inst에서 old inst에서 write하는 값을 read 해야 함
	- write하고 난 뒤의 값을 read하고 싶음
- WAR
	- old inst에서 read할 값을 young inst에서 write
	- write하기 전 값을 read하고 싶음
	- out of order일 때 문제 발생
- WAW
	- 1번 inst에서 write하고 2번 inst에서 그 값을 read하고 난 뒤에 3번 inst에서 write
	- out of order일 때 문제 발생
![200](https://i.imgur.com/GiElGNo.png)
>correct for Architecture not for $\mu$Architecture

![200](https://i.imgur.com/tX2itO6.png)
>There exists dependency btw instructions in $\mu$Architecture
## When data hazard occurs?
![500](https://i.imgur.com/bdypUlw.png)
>dependency가 있는 stage 간의 거리가 inst 간의 거리보다 같거나 크면 data hazard 발생!

![500](https://i.imgur.com/X8p6fKr.png)
resolve hazard by stalling
dist(i, j) = 4 > dist(ID, WB) = 3
### How to Stall?
![600](https://i.imgur.com/RIt4Q8n.png)

- Without Stall
	- ideally IPC = 1
- With Stall
	- N instructions and S stall cycles
	- Average IPC = $\frac{N}{N+S}$
	- HM(IPC) = $\frac{N}{\sum_{i=0}^{i=N-1}1+\frac{s}{N}}$

![500](https://i.imgur.com/b1f94TV.png)

## Data Forwading (Register Bypassing)
![500](https://i.imgur.com/P3pLDDG.png)
앞의 세 개의 inst에 dependency가 있을 경우 가장 가까운 inst의 data forwading을 선택
sequential하고 atomic하게 동작하는 architecture에서 *가장 최신 data를 사용*하는 것이 functionality를 유지하는 동작
![500](https://i.imgur.com/59kXVIC.png)
>RF forwards internally within Load dependency

forwading unit에서 rs, rt(read dest)와 rt, rd(write dest) 비교하여 dependency determine
### Data Forwarding with Load inst.
![500](https://i.imgur.com/H5LqILd.png)
LW inst dependency cannot resolve hazard even with data forwarding when dependency btw very neighbor insts
![500](https://i.imgur.com/bkwsLpt.png)
>So we need extra cycle —› load delay slot

![500](https://i.imgur.com/TWc3VWx.png)

## Why not very deep pipelines?
- split 5 stages to more stages
- What's the problem?
	- pipeline에 불리한 stage가 존재 (MEM, EX stage)
	- very complicated Data Forwarding
#### Super Pipelining
![500](https://i.imgur.com/pSopvrc.png)
- floating operation is unfriend to pipelining
- integer operation is friend to pipelining
- divide operation is unfriend to pipelining
#### Let's add extra resource instead of super pipelining
![500](https://i.imgur.com/3NFbrqq.png)
>Bigger Memory and Register File lead to slower access
