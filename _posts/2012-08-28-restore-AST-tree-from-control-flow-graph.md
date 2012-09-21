---
layout: post
category : 
tags : [compiler]
---

Recently I have come accross a little tough problem.
Our project has to extract the MPI function calling sequences from parallel program, and identify the callings in loop and branches.  
At the beginning, as the tool is LLVM whose function consists of basicblocks, my plan is to handle it from basciblock and calling sites:

1.  Traverse basicblock's instruction list. If it was mpi function calling, add to current BasicBlock's array. If it is a inner defined function
	calling, and that function is not visted, traverse that function.

2.	In each function's basicblock list, use breadth-first-search strategy to traverse the basicblock graph. When search the current node p's connected
	node, we found a node q that has been visted. It could be divided into two situations:  
	*	if q is the ancestor(direct/indirect) of p, then we found a loop starting from node q.
		<p><img src="/images/2012-08-28-1.png"/></p>
	*	if q is not the ancestor of p, then two we find the convergin point of two branches.
		<p><img src="/images/2012-08-28-2.png"/></p>

By such traverse, we could mark each node corresponding bit, such as inBranch, outBranch, inLoop, outLoop. 

After computing such information for all nodes in a function, we could traverse all the in its formal order(in fact, that's preorder of bfs).
If we comes to a basicblock with inLoop/inBranch mark, push it into stack, if we comes to a node with outLoop/outBranch mark, pop the top basicblock from
stack. And when we found the mpi function calling, add it into the current top basicblock's vector.

Finally we got the mpi function graph, organized as a ast-like tree.
