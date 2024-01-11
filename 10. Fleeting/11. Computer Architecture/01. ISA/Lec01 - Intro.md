#ISA 

>[!question] Performance Law
>Latency = ${1\over\#\ of\ cores} \times {1\over frequency} \times \#\ of\ insts \times CPI$

>[!question] Von Newman Computer
> Both program and data are stored in memory
> memory is getting slower than processor ➡️ bottleneck
![|500](https://i.imgur.com/Fxb1SN1.png)

### Transistor
#### Invertor
![|200](https://i.imgur.com/4LA0oFs.png)
#### NAND
![|500](https://i.imgur.com/fkhxPXF.png)
>NAND gate as CMOS layout

#### NOR
![|200](https://i.imgur.com/tTctq0u.png)
#### Two Invertors means "memory"
![|600](https://i.imgur.com/u5bo6cC.png)

>[!cite] Moore's Law
>\# of transistors on a chip roughly doubles every two years
>➡️ Moore's Law tells me transistors will become smaller

>[!question] Is CPU getting 2X faster every two year?
>No 
>$\because$ Dennard Scaling

>[!cite] Dennard Scaling
>Dennard Scaling tells me as transistors became smaller, they must use less power to achieve the same chip power density *(We should sustain power per chip)*
>➡️ lower supply voltage, current and capacitance
>But *lower $V_{th}$* ➡️ *greater $I_{leak}$*

![|500](https://i.imgur.com/oOK5HS4.png)
>Dennard's Scaling rule

>[!cite] PDP
> = power delay product
> *shorter channel ➡️ increament of efficiency of parasitic resistance & capacitance ➡️ increment of delay time by k*
>➡️ Scaling Factor $\frac{1}{k^3}$

