![600](https://i.imgur.com/nwN6xpt.png)
## Resolve Control Hazard with delay slot by reodering Inst.
![500](https://i.imgur.com/pUOlvDF.png)
>must reordering data independent instruction
## Resolve Data Hazard on Branch with Data Forwarding
![400](https://i.imgur.com/dPfBbDg.png)
>verify branch condition in ID stage by adding extra resource(comparator)
## Resolve Data Harzard with Data Forwading by extra resource
![500](https://i.imgur.com/Td2ul0Q.png)
- IF에서 PC+4 생성 $\to$ control hazard distance decreases from 1 to 0
- ID에서 verify brach condition and create branch target PC $\to$ Control Hazard distance decrease from 2 to 1 ; brach delay slot guarantees a dist btw dependent insts "2"
- 실제로는 branch resolution이 1 cycle이 아니라 더 오래 걸리기 때문에 Hazard distance를 1로 줄일 수 없다.
	- $\to$ *branch prediction*
## Branch Prediction
- predict the branch condition : True or False?
- predict the target address
### Simple Branch Prediction
**Predict not taken or PC+4**
- only 20% of inst is control flow
	- 70% taken and 30% not taken
	- predict nextPC=PC+4 $\to$ 86% correctness
- Misprediction must not be exposed by HW handling
	- $\to$ Flush
#### Flush
![500](https://i.imgur.com/4lggjey.png)
#### Misprediction Penalty
![500](https://i.imgur.com/tMG4sRz.png)
>branching을 EX(2)에서 하느냐 MEM(3)에서 하느냐 차이
![500](https://i.imgur.com/pnrhSyw.png)
>resolving brach condition을 ID 단계에서 하면 penalty 1 cycle로 감소

## Branch Target Buffer
![500](https://i.imgur.com/GJJ6uOR.png)
- store PC to BTB when encountering inst first time
- branch inst이면 항상 branch taken으로 predict
	- BTB에서 BTB idx address의 entry에 저장된 taget addr로 branch
### Tagged BTB
*to resolve collision*
![400](https://i.imgur.com/JiODgb6.png)
- BTB index가 동일한 inst끼리 collision 발생 가능
	- $\to$ introduce Tag!
![400](https://i.imgur.com/D1Xd4Ue.png)
- Branch inst만 BTB에 저장
	- save BTB storage
	- BTB에 일치하는 tag를 가진 entry가 없으면 PC+4
	- 있으면 branching
![400](https://i.imgur.com/WMcLwXL.png)

#### Branch History Table and Target Buffer
![400](https://i.imgur.com/KA92uYM.png)
>이전의 branch taken 기록을 BHT에 저장하고 따른다.
#### Branch Prediction FSM Example
![400](https://i.imgur.com/2x7XGZi.png)

![400](https://i.imgur.com/ruHyBXL.png)

![400](https://i.imgur.com/TaVxEmh.png)

## Global Path History
[ref](https://chopby.tistory.com/41)
![300](https://i.imgur.com/6WTItpP.png)
So far, we focused only on each branch's behavior
But branch outcome often correlated to other branches
meaning that branch outcome is affected by other branches' outcome
### Gshare Branch Prediction
*To capture other branches' information*
[ref](https://chopby.tistory.com/4)
![400](https://i.imgur.com/X3zmWfn.png)
### RAS
[ref](https://velog.io/@apphia39/%EC%BB%B4%ED%93%A8%ED%84%B0%EA%B5%AC%EC%A1%B0-Instruction-Level-Parallelism-ILP-2)
![400](https://i.imgur.com/siI8umA.png)
- *BEQ*나 *J*와 같은 명령어들은 동일한 branch에 대해 Target Address가 바뀔 일이 없다 
	- 따라서 **BTB만 사용**
- 하지만 *JAL*은 함수 호출이 끝나면 목적지(Return해야 할 Address)가 매번 바뀔 수 있다
	- 따라서 JAL 명령어는 **BTB와 RAS를 함께 사용**
- *JR* 도 RAS 사용해야함
	- register에 따라 target addr가 변할 수 있다.
### 2 - Level branch "direction" prediction
direction : taken or not taken?
![500](https://i.imgur.com/nUEXKw0.png)

- N-bit PC
- **BHT**
	- $2^{N}$가지 PC 조합에 대한 각 branch taken history를 k-bit로 저장
	- $2^{k}$가지 BHT entry 조합 존재
- **PHT**
	- $2^{k}$가지 BHT entry 조합에 대한 pattern을 m-bit로 저장
	- BHT entry에 대한 m-bit branch prediction FSM state 값이 PHT의 m-bit entry 값

![600](https://i.imgur.com/rjRRNsA.png)
>GAg is most global
>PAp is most local
- BHT
	- G : global
	- S : per set of addresses
	- P : per address
- PHT
	- g : global
	- s : per set of addresses
	- p : per address
#### Ex) Tournament Predictor
![400](https://i.imgur.com/BFti2oG.png)
>choose Pap or Gag according to Gag selector
#### Ex) Multiple predictor
![500](https://i.imgur.com/WMfWl8d.png)

![500](https://i.imgur.com/OzhBsU5.png)


![500](https://i.imgur.com/KOnVLtC.png)

![500](https://i.imgur.com/tUc4DSC.png)

![500](https://i.imgur.com/ZPqT2Cu.png)

compiler가 똑똑하면
$\to$ VLIW : dependency가 없는 inst 조합 생성하여 사용 가능
