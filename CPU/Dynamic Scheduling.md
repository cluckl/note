---
title: "Dynamic Scheduling"
date: 2021-04-19T00:13:24+08:00
draft: false
toc: false
images:
tags:
  - CPU
---
# Overview
Dynamic Scheduling（动态调度）：在**程序的执行过程**中，依靠专门**硬件**对代码进行调度，减少[Data Dependence](Dependence.md#Data%20Dependence)导致的停顿。
>静态调度: 依靠编译器在**编译期间**把相关的指令**拉开距离**来减少可能产生的停顿

limitation: In-order Issue（按序流出）、 In-order Execution（按序执行）
solution: Out-of-order Execution（乱序执行）
为了能够乱序执行 ，将 5 段流水线的译码(ID)阶段再分为两个阶段：
![525](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210424155306.png)

1. **流出（Issue, IS）**：指令译码，检查是否存在结构相关和WAW冲突。（in-order	issue)
2. **读操作数（Read Operands, RO）**：等待数据冲突消失，然后读操作数。

* 动态调度让指令**乱序完成(Out-of-order Completion)**，会大大增加异常处理的难度。
* 动态调度需要保持正确的异常行为，但动态调度处理机仍可能发生不精确异常，使得在异常处理后难以接着继续执行程序。
	
	> 不精确异常：指令发送异常时，流水线可能已经执行完该指令之后的指令或没有执行完该指令之前的指令。

# Scoreboard
记分板（scoreboard）负责管理指令的流出和执行，包括检查[Dependence](Dependence.md)。
![](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210424155307.png)
## Steps
1. Issue
	The scoreboard issues the instruction **only when**:
	* a functional unit for the instruction is **free**(structural hazards)
	* no other active instruction  has the same destination register(avoid [WAW](Dependence.md#WAW)) 
	
	==Otherwise,  the scoreboard stalls the instruction.==and no further instructions will issue <u>until these hazards are cleared (in-order issue).</u>

2. Read operands
	The scoreboard issues the instruction **only when**:
	* **All sources operand** is available, which mean no **earlier issued active instruction** is going  to write them(avoid [RAW](Dependence.md#RAW)) 
	
	==Otherwise,  the scoreboard stalls the instruction.==
	After issuing, the  scoreboard tells the functional unit to proceed to read the operands from the  registers and begin execution. 
	
	<u>	Instructions may be sent into execution **out of order**.</u>(只是stall该条指令，不会因此stall别的指令)
3. Execution
	The functional unit begins execution upon receiving operands. When the result is ready, it **notifies the scoreboard **that it has completed execution.
4. Write result
	Once the scoreboard is aware that the functional unit has completed execution, it writes results to destinaton oprands **only when**:
	* **earlier issued active instruction** don't need to read **one of the destinaton oprands ** (avoid [WAR](Dependence.md#WAR)) 
	
	==Otherwise,  the scoreboard stalls the instruction.==
	
## Implement
To implement the [Steps](#Steps), scoreboard have 3 tables to record status:
* Instruction status: which of 4 [steps](#Steps) the instruction is in
	![](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210424155308.png)
	
* Functional unit status
	![](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210424155309.png)
	* Busy: Indicates whether the unit is busy or not
	* Op: Operation to perform in the unit (e.g., + or –)  
	* Fi: Destination register
	* Fj, Fk: Source-register numbers
	* Qj, Qk: Functional units producing source registers Fj, Fk
	* Rj, Rk: Flags indicating when Fj, Fk are ready

* Register result status
	![](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210424155310.png)
	Indicates which functional unit will write each register, if one exists.  
	
## Limitations
* Number, types, and speed of the functional units
	* This determines how often a structural hazard results in stall.
* 没有实现 [Data Forwarding](Dependence.md#Data%20Forwarding)
* 没有完全解决[Data Dependence](Dependence.md#Data%20Dependence)，只是通过简单的stall。

# Reference
* [Dynamic Scheduling](http://ece-research.unm.edu/jimp/611/slides/chap4_3.html)
* Jianhua Li-College of Computer and Information-Hefei University of Technology-08-Exploiting ILP 