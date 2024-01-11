## Pipeline Idealism
- Motiv
	- increase throughput with little increase HW
- Repetition of identical operations
- Repetition of independent operations which have no dependency with each other
- Uniformly partitionable sub-operations - Same length sub-task
## Performance & Cost of Pipeline
![400](https://i.imgur.com/ZTT6vYu.png)
pipeline 성능이 좋은 이유 :
task를 쪼개어 clock frequency가 증가했기 때문

![400](https://i.imgur.com/lppIFlm.png)
>trade off : latch 수가 증가하기 때문에 task를 쪼갠 만큼 latch cost가 늘어난다.

![500](https://i.imgur.com/ybXYEu9.png)
## Pipeline Implementation
![600](https://i.imgur.com/rwMY1Ww.png)
![600](https://i.imgur.com/iN3hDXJ.png)
>control signal도 같이 pipeline을 타고 이동한다. latch transfer
#### Sequential Control
![600](https://i.imgur.com/FTBdGNG.png)
- ID stage에서 control signal을 한 번에 얻고 pipeline을 따라 이동하면서 각 stage에서 사용될 때까지 buffer해서 가지고 있는다.
- inst field를 가지고 pipeline을 따라 이동하며 각 stage에서 local하게 decoding을 하여 필요한 control signal을 얻는다.
#### Issue of Pipelining
![500](https://i.imgur.com/87h3Pcd.png)
- inst 종류마다 필요한 stage가 다름
- inst끼리 dependency 존재 not atomic
- sub operation들이 uniform하지 않음
	- Mem access는 상대적으로 오래 걸린다.
