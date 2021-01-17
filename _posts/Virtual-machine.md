---
title: Virtual machine
date: 2020-12-11 18:00:09
tags: JVM
---

[learn from this great article](https://markfaction.wordpress.com/2012/07/15/stack-based-vs-register-based-virtual-machine-architecture-and-the-dalvik-vm/)

## **Stack Based Virtual Machines**

eg. Java Virtual Machine, the .Net CLR

> A stack based virtual machine implements the general features described as needed by a virtual machine in the points above, but the memory structure where the operands are stored is a stack data structure. Operations are carried out by popping data from the stack, processing them and pushing in back the results in LIFO (Last in First Out) fashion.



## **Register Based Virtual Machines**

eg. Lua VM, and the Dalvik VM

> In the register based implementation of a virtual machine, the data structure where the operands are stored is based on the registers of the CPU. There is no PUSH or POP operations here, but the instructions need to contain the addresses (the registers) of the operands. That is, the operands for the instructions are explicitly addressed in the instruction, unlike the stack based model where we had a stack pointer to point to the operand.

### Dalvik 

> Dalvik differs from the Java virtual machine in that it executes Dalvik byte code, and not the traditional Java byte code. There is an intermediary step between the Java compiler and the Dalvik VM, that converts the Java byte code to Dalvik byte code, and this step is taken up by the DEX compiler.