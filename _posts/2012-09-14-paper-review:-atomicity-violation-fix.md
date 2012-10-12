---
layout: post
category : 
tags : [research]
---

The paper is [**_Automated Concurrency-Bug Fixing_**](http://pages.cs.wisc.edu/~shanlu/paper/cfix-OSDI-2012.pdf). **Guoliang Jin, Shan Lu, PLDI'11**


###Brief Introduction

One kind of bugs in concurrent programs is the atomicity violation, accessing shared variables within several thread/processes without locking  protection. This paper proposes the systematic way to generate patches for this automatically. By the aid of atomicity  bug detecting tool, Ctrigger's help, the author get bug location, uses the simple but strong method to identify protected node in critical region. What's more, the paper also demonstrates the work of merging different patches and runtime feedback, leaving space for patch refinement. 

###Advantages

####1) Good Method to distinguish two kinds of nodes

The most impressive point in the paper is its succinct but powerful method to calculate the protected nodes in critical region. After simple twice search, forward from P point and backward from C point, and then conduct the injection of searched nodes, it got all protected nodes. What's more, the method strength also comes from its rightness: all path from P to C would fall within the protected nodes. Dichotomy of nodes for critical regions makes the problem to be clear and perusaive. It works well.

*
After reading (Practical Memory Leak Dectection), I found the twice search approach, forward and backward, has been put up before. So they're maybe inspired by previous work.



###Defects

####1) Patch quality is largely decided by the Bug Detector Tool
 
Just as the paper presents, bugs in FFT, PBZIP2 are not greatly ameliorated by Afix's patches and it results from the false positive report of another bug by CTrigger.  So we could know that the patch quality is largely decided by the Ctrigger's bug detection precision. And the Ctrigger's ability seems to be limited. 

The situation becomes worse if the target program to be patched is full of many bugs. These  would increase the CTrigger generated wrong patches. In other words, because of great dependence on Bug Detector, it narrows down the application range of Afix patch system.



####2) Merging Operation Needs further consideration 

Merging operation's nature is  combine several overlapping critical regions together. Because of fewer locks involved, the decrement in  rate of deadlock should occur as expected.

But the performance, would not always be improved. The merging brings about larger critical region, larger locking granularity. The negative side of this is potential loss of parallelism, which would degrade the performance. 

####3) Its overestimated argumentation of no new bugs imported


The paper stress its no importation of new bugs. But in fact, the deadlock is great problem. The time-out way to check deadlock is also temporary and not completed. Timeout can also occurs in other situations, especially when the program does some time-consuming computation or IO operations in critical region.

Even thought the merging version could decrease the rate of deadlock, but it's due to larger critical regions and fewer locks inserted, referred above.


####4) Interpretation of Experiment's Data

In paper's Table 3,  the author lists the failure rates under circumstances where program is injected with noises of random probability and length to sleeping. But it does not clarify the meaning of failure rate? Whether it's the deadlock rate or the atomicity violation rate, or the both? I think, in fact, they are two different metrics to measure correctness of patch. The atomicity violation rate judges the patch's efficiency. The rate of runtime error, such as deadlock, reflects the probability of importing new bugs. But the paper does not distinguish them.


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

 

####2) Prerequsite to apply Afix
The CTrigger has to be provided bug-triggering input to generate record. But in fact, this inputting may be hard to obtain. There is no general rule/format to describe the inputting case, and the bug may fail to reappear. If the bug detector could be more wisdom, the Afix would be in better use. I admitted that automatic atomicity bug detecting may be challenging.

###Questions


####1) About the call stack

I notice the paper use call stack heavily, to analyze the containing relation between two critical regions. But as call stack is dynamically determined, how could it guarantee the analyzing results covering all situations? I think instead, static analysis may work better.

####2) The way dealing with recursive function

When coming across critical region in recursive function, author used the reentrant lock. But I'm not sure about that. As I know, tail call optimization would make returning to first caller directly. So program may lose the chance to unlock reentrant lock enough times. Is my concern superfluous?
