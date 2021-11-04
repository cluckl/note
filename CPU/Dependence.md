---
title: "Dependence"
date: 2021-04-18T00:13:24+08:00
draft: false
toc: false
images:
tags:
  - CPU
---

**Dependences** dictate ordering requirements between instructions.Also called **Hazards**
Two types:

* [Data Dependence](Dependence.md#Data%20Denpendence)
* [Control Dependence](Dependence.md#Control%20Dependence)
# Data Dependence
Types of data dependences
* [RAW](Dependence.md#RAW)(Flow dependence) 
* [WAW](Dependence.md#WAW)(Output dependence) 
* [WAR](Dependence.md#WAR)(Anti dependence) 

> Flow dependence also called true dependence, and output dependence and anti dependence are called Name Dependence. 
## RAW
read after write, also called Flow dependence 
B tries to read a register before A has written it and gets the old value.
![600](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210424155155.png)
This is common, and [Data Forwarding](Dependence.md#Data%20Forwarding) helps to solve it.
## WAW
write after write, also called Output dependence.
 B tries to write an operand before A has written it.
 ![600](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210424155156.png)
 After instruction B has executed, the value of the register should be B's result, but A's result is stored instead.

This can only happen with pipelines that write values in more than one stage, or in variable-length pipelines (i.e. FP pipelines).
## WAR 
write after read, also called Anti dependence
![600](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210424155157.png)
In this case, A uses the new (incorrect) value.

This type of hazard is rare because most pipelines read values early and write results late.

## Simple soultion
![600](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210424155158.png)
### Write in frist
 Write the register file in the first half of the cycle and read it in the second half.
![600](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210424155159.png)
Fixes the hazard shown in green.
### Data Forwarding
Also called **Data Bypassing**.
* Allow the CPU to move a value directly from one instruction to another without going through the register file.
* Feeding back the data values from the pipeline registers to the inputs of functional units(future instructions) behind them in the datapath.

![600](https://raw.githubusercontent.com/cluckl/Pinnned-repo/master/img/20210424155200.png)
Fixes the hazard shown in green.
### Stall 
[Data Forwarding](Dependence.md#Data%20Forwarding) helps Loads, but it does NOT solve all the problems: Forwarding helps Loads, but it does NOT solve all the problems.!
[](../pic/Pasted%20image%2020210417231439.png)
talling is necessary in this case for proper execution until the hazard is cleared.
> Insert a bubble into the pipeline also be right.
# Control Dependence
All instructions are control dependent on previous ones. 
* The fetched instruction is a non-control-flow instruction, which the next Fetch PC is the address of the **next-sequential** instruction.
* The instruction that is fetched is **a control-flow instruction**.
## Solution
* **Stall** the pipeline until we know the next fetch address
* **Branch prediction**: Guess the next fetch address
* **Branch delay slot**: Instruction Scheduling


# Reference
* [Pipelines Hazards](http://ece-research.unm.edu/jimp/611/slides/chap3_3.html)
* Jianhua Li-College of Computer and Information-Hefei University of Technology-06-Pipeling & Hazards
* Jianhua Li-College of Computer and Information-Hefei University of Technology-07-Dependence Handling