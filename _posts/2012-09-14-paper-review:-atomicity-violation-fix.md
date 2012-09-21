---
layout: post
category : 
tags : [research]
---

The paper is [Automated Concurrency-Bug Fixing](http://pages.cs.wisc.edu/~shanlu/paper/cfix-OSDI-2012.pdf). Guoliang Jin, Shan Lu, PLDI'11


###Brief Introduction

One of common bugs in concurrent programs is the atomicity violation, accessing shared variables within several thread/processes without locking  protection. This paper proposes the systematic way to generate patches for this   kind of bugs automatically. By the aid of atomicity  bug detecting tool, Ctrigger's help, the author get bug location described in triple form, uses the simple but strong method to identify protected node in critical region. What's more, the paper also demonstrates the work of merging different patches and runtime feedback, leaving space for patch refinement. 

Even though it focus on a specific point, but the complete and thorough research aspects and novel way to solve real-world problem effectively compared with hand-debugging, makes it practical. I think where its contribution lies.



###Advantages

####1) Good Method to distinguish two kinds of nodes

The most impressive point in the paper is its succinct but powerful method to calculate the protected nodes in critical region. After simple twice search, forward from P point and backward from C point, and then conduct the injection of searched nodes, it got all protected nodes. What's more, the method strength also comes from its rightness: all path from P to C would fall within the protected nodes. Dichotomy of nodes for critical regions makes the problem to be clear and perusaive. In fact, I have tried to disprove its correctness by following code segment:

for (; ...; ...)
{
	if (...){
		P point;
	}
	else{
		C point;
	}
}

But after short thinking, the paper's algorithm also works fine in this situation. As the nodes in loop would form a circle and each node in loop is  successor and predecessor of all nodes in the loop, including itself. But I still have tiny questions about the algorithm, referred in Question section.

*
After reading second paper(Practical Memory Leak), I found the twice search approach, forward and backward, has been put up before. So they're maybe inspired by previous work.


####2) Various Aspects for the subject
 
Another advantage of the paper is about work completeness. Even though the problem just focus on single variable's atomicity violation, the paper ranges from different approaches comparison, runtime information collection, deadlock consideration, and merging. Often I want to point out drawbacks, but it supply addition explanation in later section. 



###Defects

####1) Efficiency is largely decided by the Bug Detector Tool
 
Just as the paper presents, bugs in FFT, PBZIP2 are not greatly ameliorated by Afix's patches and it results from the false positive report of another bug by CTrigger.  So we could know that the patch quality is largely decided by the Ctrigger's bug detection precision. And the Ctrigger's ability seems to be limited. 

The situation becomes worse if the target program to be patched is full of many kinds of bugs. These  would increase the CTrigger system false atomicity violation identifications. It is quite normal in early version of a complicated system.

In other words, because of great dependence on Bug Detector, it narrows down the application range of Afix patch system.



####2) Merging Operation Needs further consideration 

As for merging operation, the paper presents several benefits for merging, including code readability, performance, correctness, deadlock risk. It is not as good as it says.

Merging operation's nature is  combine several overlapping critical regions together. Because of fewer locks involved, the decrement in  rate of deadlock should occur as expected.

But the performance, would not always be improved. The merging brings about larger critical region, larger locking granularity. The negative side of this is potential loss of parallelism, which would degrade the performance. And it's not a negligible problem, because we are handling the parallel program, and speed comes from the paralelism.

I haven't come up with a specific example to illustrate this situation. But there is no evidence to negate this problem's existence. Maybe by some runtime experiment recording, could we get results easily.


####3) Its overestimated argumentation of no new bugs imported


The paper stress its no importation of new bugs. But in fact, the deadlock is great problem. The time-out way to check deadlock is also temporary and not completed. Timeout can also occurs in other situations, especially when the program does some time-consuming computation or IO operations in critical region.

Even thought the merging version could decrease the rate of deadlock, but it's due to larger critical regions and fewer locks inserted, referred above.


####4) Interpretation of Experiment's Data

In paper's Table 3,  the author lists the failure rates under circumstances where program is injected with noises of random probability and length to sleeping. But it does not clarify the meaning of failure rate? Whether it's the deadlock rate or the atomicity violation rate, or the both? I think, in fact, they are two different metrics to measure correctness of patch. The atomicity violation rate judges the patch's efficiency. The rate of runtime error, such as deadlock, reflects the probability of importing new bugs. But the paper does not distinguish them.

As for testing failure rate, I think they still used Ctrigger to detect atomicity violations.


###Ideas

####1) Implementation of lock/unlock operation on edges

After get computing results of protected and unprotected nodes, insertion of lock() and unlock() occurs at the edge crossing between these two kinds of nodes. The implementation is not so simple to just put the lock() statement at the beginning of basicblock. Following picture illustrate my points,

<p>
	<img src="/images/2012-09-14-1.png"/>
</p>

The dark color node stands for protected, light stands for unprotected. 
Because node 2 has a edge starting from P, so we should insert unlock().
How to make the unlock() only invoked when the predecessor of 2 is P, not invoked when predecessor is Node 1? 
I have come up with two solutions.

a) Add a flag variable to identify whether it passed through P node or 1 node by assignments in P and 1 node, and only the flag variable meets the id of P, it would invoke unlock(), easily implemented by a if(){} statement.

b) Add a node between P and 2, let name it node T, the graph would change to,
p → T → 2.
We could add the unlock() /lock() statement in T node.

I prefer second solution. 

 

####2)
The CTrigger has to be provided bug-triggering input to generate record. But in fact, this inputting may be hard to obtain. There is no general rule/format to describe the inputting case, and the bug may fail to reappear. If the bug detector could be more wisdom, the Afix would be in better use. I admitted that automatic atomicity bug detecting may be challenging.



####3) The Application of the Afix

The paper wants to generate and apply the patches automatically by its Afix 
engine. But consider this, if there is a bug in program, would the developers be reassured after telling them the bug could be removed by  machine-generated patches? Obviously not. Too much relying on this tool may lead to poor readability and difficulty in code maintenance. 

Instead, I think the tool could be used as developing guidance, giving hints about the bugs and its generated reasons. The patches could be used as probable solution offered to programmers. With these help, they could greatly improve the debugging efficiency. Because programmers are the people most familiar with program structure and design.

Even thought it sounds not cool in theory, but maybe the most useful way in practice.


###Questions


####1) About the call stack

I notice the paper use call stack heavily, to analyze the containing relation between two critical regions. But as call stack is dynamically determined, how could it guarantee the analyzing results covering all situations? I think instead, static analysis may work better.

####2) The way dealing with recursive function

When coming across critical region in recursive function, author used the reentrant lock. But I'm not sure about that. As I know, tail call optimization would make returning to first caller directly. So program may lose the chance to unlock reentrant lock enough times. Is my concern superfluous?
