---
layout: post
category : 
tags : [research]
---

The paper is [Practical memory leak detection using guarded value-flow analysis ](www.cs.cornell.edu/~rugina/papers/pldi07.pdf) (Cherem, Princehouse) PLDI'07

###Brief Introduction


Facing the frequent problems of leaking memory and double free, the paper proposes the static value-flow analysis to detect the memory leakage. Firstly, the author transforms the problem to source-sink problem, that every source of memory allocation must flow to exactly one sink of deallocation. Thus, by parsing and processing program to get value-flow graph, they attach guards to edge, denoting the in which condition the path would follow that way. Finally, the analysis engine calculates the possible conditions meeting the guards that would lead to none, one, or more deallocation sink. They presents the memory management error by listing the possible conditions(in text) or visual branches(in HTML) that leading to leaking.

In this process, to simplify the problem, the paper also does some other prepared work, such as allocator function's identification, formula simplification, post-dominator in CFG.

In the end, the paper shows its tool's experimental data, and compare its precision, speed, false positive ratio to other similar tools. 




###Advantages

####1) Simple but rigorous mathematical model

By the transformation of the problem to source-sink reachability, the problem becomes more clear. And the edges conditions' computation is described by mathematical recurrence formula. 

In this model, the reachability condition could be easily desribed by set operations, such as conjuction, disconjunction. It could not only give support for its thesis, but also faciliates understanding.

####2) Apply many techniques to speed up analysis process

In this paper, we could discover a lot of techniques to optimize the analysis engine. Allocator function's detection, simplification of testing condition, narrowing down the range of slice nodes.

What's more, in the process of computing cguard(n -> m), it uses the dynamic programming, which could largely reduce the algorithm complexity. 



###Defects

####1)  capability of detection is limited 

The leaking detection is based on value-flow analysis, which is flow-insensitive  and static. That limits its precision and power compared with other general approaches. From its experimental data, we could find that even though it does well in many aspects, such as speed, false positive ration, it generates too much unknown condition. And when compared with other tools, such as Saturn Figure 9, its detected bug count falls far behind, 63 vs 455.  

This problem is due to two aspects, its incapability to analyze aggregate global data structure, such as recursive structure. Another reason is its pure static directed approach as the complicated testing condition equivalence relationship specific to program may be hard to obtain.

What's more, I think the analysis should utilize the compiler's optimization, like redundancy elimination, loop unrolling, instead of build itself from parsing module. Taking following case as example,

	int i, a=0;

	for ( i = 0; i < 5; i++) {
		a+=3;
	}
	if (a>10){
		free(...)
	}

In its analysis engine, the free() sink would be guarded by condition `a>10`, and generate a `may memory leak` problem.
But if this code is optimized by compiler, the testing condition would be removed as a's value must be greater than 10.

After reading the third paper, KLEE Unassisted Test Cases Generator, this paper's drawback is more obvious. Because it is totally based on static analysis.


####2) Bug Report Message

The paper argues that the bug message is very concise, easily understood as biggest slice node number is just 11. But the small size of the slice  does not merely come from its good algorithm.

In section 5.6.3, the paper says, when the guard formula becomes very large in complicated and unstructured control-flow(maybe due to `goto` statement), they just stop analysis and mark it unknown. It could also partly explain why so many unknown leaking problem in experiment data. So conciseness of message is partly due to removing of complicated guards. 


###Ideas

####1) About the memory leaking detection

The method in this paper is basically relied on static analysis. On the contrary, In Java/C# languages, they adopt the approach of reference counting, to collect memory automatically. Inspired from that, I have the idea, why not also equips C with such facility? Not meaning changing C language itself, we could still get some benefits from that.

Firstly, we could implement the reference counting system for C, by inserting several  code segments at corresponding program sites. Such as, add reference count when coming across the pointer assignments, decrease the count when pointer is assigned to another address or it's out of life-range. Then run the program, and do calculations of reference counting to determine existence of memory leaking. If we found one, give the traces of problem to programmer, and they could remove reference counting code after fixing leaking bug. So it would not low down the final program's performance.

There are two potential problems of this solution. Firstly, to detect leaking problem in runtime environment, it has to cover large possible paths. But this problem could be alleviated by some test tools, that generate nice testing samples(Paper “KLEE” is on this topic). Another problem is difficulty of reference count construction due to C's capability of direct operation on memory. For example, “memcpy()” may copy the pointer.


####2) Deal with Program's Revision Code

From the paper's figure, we find Saturn analyzer, even though could beat FastCheck system in number of bugs reported, the speed is not promising. But considering this,  for a project with 130, 000 lines code, Saturn would take 130, 000 / 50 / 60 = 43 min to process. That is still acceptable. 

But if the analysis has to be reconstructed from beginning for another 43 min after adding or removing several lines of codes, it would be disappointing. So it would be better that the leaking detection system is designed to adapt to  code revision, and just compute the changing part. Then the speed advantage of FastCheck would not seem to be so much important. This is very useful in practice, programmers could rerun the analyzer after make some revisions conveniently.


####3) Including Dangling pointer Error Report

As the paper has worked out the memory leaking problem, why not also include the further analysis  on dangling pointer error? Firstly we could get the reaching definition for pointers.  And for each pointer's using point, compute the guards along the path from “free()” point to pointer's using point, then this task is also be transformed to a similar problem: identify the conditions that would lead free() pint flowing to not-redefined pointers using point. So we could still solve this problem by the aid of FastCheck system.

####4) Trade-off between efficiency and precision

In the paper's experiment section, we could notice, some existing similar leaking detection tool, for instance,  Saturn, outperform than this tool. The only drawback is its low speed. If we put some techniques, such as traversal depth, applied algorithm as parameters, then the tool is configurable and user-friendly. 



###Questions & Understanding



####1) About Region Nodes

In the paper, region node stands for a block of allocated memory. But how to calculate it, what kind of program segment corresponds to the partition? 

####2) Recursive Formula to Calculate Guard Condition

<p>
	<img src="/images/2012-09-16-1.png"/>
</p>

At first, I do not understand why cguard(n → m) = cg(x,n,m,empty)
After thinking , I realize that `Empty` means the none of edges have been visited so all of edges would be deemed as future available path.
