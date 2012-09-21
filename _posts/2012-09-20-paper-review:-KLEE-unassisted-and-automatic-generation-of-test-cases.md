---
layout: post
category : 
tags : [research]
---

The paper is [Klee: Unassisted and Automatic Generation of High-Coverage Tests for Complex Systems Programs 
](http://www.stanford.edu/~engler/klee-osdi-2008.pdf). (Cadar, Dunbar, Engler) OSDI'08

###Brief Introduction

The test cases are important to ensure the software's quality, but the handful writing test cases are time-consuming, costly, and affected by programmers' preconceptions. To address this problem, this paper propose the a method to generate test cases automatically. 

Its idea is to implement a interpreter-like operating system. The variable is described as symbolic value, not having specific value. And when coming to branch, the system would clone the processes and record the constraint of certain variables in order to follow that branch. KLEE system follows random or weighted path to cover more statements. When the interpreted program ends, it would compute the constraint and give out the test input. It also explains its way to solve environmental problems, that many values are determined in the interactive process, such as command line, input file.

In the experimental section, it reports the results on GNU COREUTILS programs, showing high statement coverage compared with manual test-suites.

I also consult related information, finding KLEE has been widely used, popular in both industry and academy. Lots of further work are also conducted on the base of KLEE. For those contributions, I really admire this work.


	
###Advantages

####1) Novel Idea to Generate Test Case

I feel a little excited after understanding its idea, implementing the micro operating system and putting the constraint on the symbolic value while walking on the paths. Even though the paper has not analyzed why its got high statement coverage ratio compared with others, I think it is largely due to its complete control over the program when executing in the interpreter. 

The pure static method, like the data flow analysis, is limited because it's difficult to make results determined. Aliases, point-to problem could low down the precision of static way. But this paper's approach avoid this drawback. It interpret the program, so a lot of things could be determined dynamically without doing complex static analysis.


####2) Dividing The Environment Variables into two Conditions

Instead of treating the environment values unconstrained and random, the paper divide environment into two cases, the concrete file and unconstrained symbolic file. When the parameters of the system call is concrete, there is no need to assume the file containing any possible unconstrained characters. 

This dichotomy could  decrease uncertainty so as to enhance the precision.


####3) Emphasis on the extension and wide applicability

Just as the paper emphasizes, KLEE system is built with extension ability. Taking the environment variable's constraint as example, the constraint model code is written in C, completely independent of KLEE system itself. This feature could help it be extended with users' supplement of constraining code. For an instance, compiler tool Yacc and Bison requires highly grammar input text, so if the developers could describe the grammar in C code, it could be used to test the tools.

In the experiment section, the paper puts up another usage of KLEE, verify equivalence of different versions of the same program. By the aid of KLEE, we could replace complicated therm prove.

####4) Optimization to Support large number of process

KLEE has to hold up many process in order to cover most statements. To deal with the problem of limited memory, the paper puts up the plan to share common values among different processes, such global value, and just clone private variables, like stack objects. It could largely saves up the memory.
What's more, copy-on-write strategy could low down the overhead.


###Defects


####1) Limitation of detecting bugs, especially in loop

From the paper, I notice that it adopts the random or weighted method to search the possible path when coming to branches because its goal is to cover new statement. But it's limited especially in loops. Because once the statement in loops has been executed, it would low down the possibility to executed again. And as the variable is defined as symbolic value, so the looping times could not be determined. However, loop is the place easy to generate bugs.

	for ( int i = 0; i < 10000; i++)  
		malloc(100);

The bug could be hardly detected unless the loop body has been executed enough times.

Of course, the problem of loop is not easy to solve. As we could not designate a specific cycle value to find bugs. And each cycle would add one constraint, so it may be tough for query solver to give out the answer.
 

####2) Taking statement coverage as metrics

The paper takes the statement converge as the metrics to measure the completeness of the test case. Even though statement coverage is the higher the better, it could not stands for the completeness of the test. In software engineering testing, other target, such as the path combination, is also vital to find the bug. The goal of the test is to verify the correctness and bugs, instead of merely covering the statements.

In fact, I think another important metric is the redundancy of test cases. 2000 test cases and 20 cases both could test the program to the same degree, but we surely wants fewer cases to achieve the same coverage. 

KLEE path searching strategy aims to cover new statements, so it should not have this redundancy.

###Questions & Understanding


####1) How To find bugs

When reading its example of tr program, I'm curious about how to find bugs. As it explains in the paper, program would fork at Line 3,4,15,18.
And then it constructs the test case('\[') that triggering overflowing array bug. But in fact, there are many possible inputs match the first 
three constraints along the path, such as '\[aaa'. How could it select '\[' to trigger the bug?

One possible explanation is that each time the program access value in the array, the interpreter would make the check to see whether it exceed array's boundary. If it's possible after make query to query solver, then generate the corresponding test case. But the problem is, if each time accessing array do the checking, the performance is affected. 

What's more, if KLEE detect bugs by this special checking way, it's not applicable to other kinds of bugs as KLEE just focus on these kinds of bugs.


