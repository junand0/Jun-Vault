## Latency vs Throughput
- *Latency*
	- Time btw start and finish of a single task
	- applicable in interactive applications
- *Throughput*
	- Number of task finished in unit time
	- applicable in batch applications
- *Throughput is not always $\frac{1}{latency}$ when concurrency exists*
	- concurrency : context switching을 통해 여러 개의 task를 바꿔가며 실행하여 마치 동시에 여러 개의 프로그램이 실행되는 것처럼 보이는 것
	- Improving latency does not necessarily improve throughput vice versa
- *But, Not completely distict*
	- increasing throughput of component processing shortems the latency of the overall task
### Unix time command
- Elapsed time
	- wall-clock time
- User CPU time
	- time spent running your code
- System CPU time
	- time spent running other code on behalf of your code
	- 내 프로그램을 실행하기 위해 다른 프로그램을 실행하는데 걸린 시간
- time spent running other code unrelated to your code
	- =Elapsed time - (user CPU time + system CPU time)
## IPC, MIPS and GHz
>[!cite] *Iron Law*
>wall clock time = $\frac{time}{cycle}\times \frac{cycle}{inst}\times \frac{inst}{program}$

![500](https://i.imgur.com/oW2m5X9.png)
> inst : 실제 실행한 inst 총 개수
> This is for "single core" latency - performance only!

## FLOPS
Scientific Computing people often use FLOPS
*Nominal number of floating-point operations* program runtime
ex) FFT of size N has nominally $5N\log_{2}N$ FP operations
- This is fair metric to compare performance only when talking about same problem
	- Not all FFT algorithms have the same number of FP ops
	- Not all FP operations are equal
		- FADD vs FMULT vs FDIV
## Multi-dimensional Comparisons
### Runtime and Energy
- Intereted in not only minimizing individual metrics but also consider trade off
	- spending more energy to run faster
	- if run slower then less energy used
- Metrics which is interested in power consumption
	- $Power=\frac{Energy}{Ti me}$
	- $Energy\ d elay\ pr oduct=Energy\times ti mes$
	- In general, f(Energy,Time)
		- ex) $E^{2}T$ for the case power is more important
## Relative Performance
- $Perfor manc e= \frac{1}{Ti me}$
	- Shorter latency then higher performance
	- higher throughput than higher performance
![400](https://i.imgur.com/OY7oLzD.png)
>the result is different in terms of view
#### How to avoid confusion?
![500](https://i.imgur.com/SnFdKim.png)
#### Speedup
![400](https://i.imgur.com/HduuQhn.png)
#### Amdahl's Law
![400](https://i.imgur.com/9Pr4lkd.png)
*Make the "common case" faster, frequently occurring events or used mechanisms*
#### Arithmetic Mean
when workload is application A0, A1, ... , An-1
AM of the application run time = $\frac{1}{n}\sum_{i=0}^{n-1} Ti me_{A_{i}}$
- if $\frac{AM_{X}}{AM_{Y}}=n$ Y is n times faster than X
	- True only when all of applications are run equal number of times always
	- False if some applications are run more frequently than others
	- it is not considered that longer applications have greater contribution
#### Weighted AM
$\sum_{i=0}^{n-1}w_{i}=1$
where $w_{i}=\frac{A_{i}'s\ run\ n umber\ of\ ti mes}{total\ run\ n umber\ of\ t imes}$
- Weighted AM is
	- $\sum_{i=0}^{n-1}w_{i}Ti me_{A_{i}}$
#### Geometric Mean
GM=$^{n}\sqrt{ \Pi_{i=0}^{n-1} \frac{Ti me_{A_{i\ on\ X}}}{Ti me_{A_{i\ on\ Y}}} }$

$\frac{GM(X_{i})}{GM(Y_{i})} = GM(\frac{X_{i}}{Y_{i}})$

*report the performance of computer X as relative speedup over the reference computer Y*
![500](https://i.imgur.com/tVitckO.png)

![400](https://i.imgur.com/U2rFAmD.png)
![400](https://i.imgur.com/Lj0GhEv.png)

#### Harmonic Mean
![500](https://i.imgur.com/hF9ZIox.png)
![500](https://i.imgur.com/RjHqnKJ.png)
>not AM(IPC)!
>don't take $\frac{1}{n}\sum_{i=0}^{n-1}IPC_{i}$

#### Conclusion
- Arithmetic Mean for *Amount*
	- Latency, time
- Geometirc Mean for *Ratio*
	- Based on normalization
- Harmonic Mean for *Rate*
	- Speed, IPC

## Standard Benchmarks
![500](https://i.imgur.com/XxOTV2J.png)
