---
layout  : page
title   : "Register allocation"
date    : 2018-10-26
author  : "pshung"
---
## Problem:
Assigning infinite virtual registers into limited number of physical registers, and interfering variables must be assigned to different physical registers.

N-colorable map problem is a NP-compete problem.

# Term:
**Program point**: a point between two instructions.   
A variable(V) is **alive** at a program point(P) if there is an instruction after P using variable V.   
**Live range** of variable V: a collection of program points which V is alive.  
**Live interval**: the longest path in live range of variable V.  
**Coalescing**: if two variables are related by a move instructions and their live ranges doesn't overlap. Allocator can assign the same register to both variables and eleminate the move instruction. This technique merges the live range of two variables and forms a longer live range; thus, it may incur more register spills.  
**Live range spliting**: the inverse of coalescing, by adding a move instruction and rename the name of variable, it can split a long live interval into two smaller live intervals.   



# Graph coloring algorithm:
Using interference graph as data structure and adopt a heuristic algorithm to generat a suboptimal allocation.
Illustraction in most compiler textbook.  

# Linear scan algorithm:
Why not graph coloring? because building an interference graph is very expensive.  
Linear scan is a simple but suprisingly good enough algorithm. [used in llvm since 2004 and be replaced by greedy and basic RA in llvm 3.0](http://blog.llvm.org/2011/09/greedy-register-allocation-in-llvm-30.html)  

How does linear scan work? 
First, sorting live interval by the starting program point.  
Second, visiting live intervals in the sorted order and allocate the registers.  
Keeping an active list(alive variable till current program point) to record the overlapping varaibles. (inactive when the live interval ends)  

# Greedy register allocation ()

# Reference:
[1] A survey of register allocation, Fernando Magno Quint˜ao Pereira October 12, 2008  
[2] LLVM Greedy Register Allocator – Improving Region Split Decisions Marina Yatsina Intel Corporation, Israel April 16-17, 2018 European Developers Meeting Bristol, United Kingdom  