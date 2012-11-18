---
layout: post
category : 
tags : [compiler]
---

###Background 

Our MPI trace compression research project needs to identify MPI function calling sequence
within loop. And to compress the MPI trace, we have to do instrumentation profiling by inserting
some function calls at certain position, such as the beginning and ending of loop, branch. This
static analysis work is based on LLVM. 

It requires me to obtain structural information of MPI callings, organized as AST. But the problem
is that, LLVM just provides one level IR, which is represented as connected basic blocks. Here is
the problem. 

###Introduction
After careful thinking and comparison, I found that LLVM control flow graph contains much
information to restore AST from that. See the pictures below.

<p>
	<img src="/images/2012-08-28-1.png"/>
</p>



Figure I shows the control flow graph for a typical loop. Let’s conduct the Breadth-First-Search on
the Control Flow Graph. And we found node 3 is connected with a previously visited node, Node 1. And Node 1 is Node 3’s ancestor. So we find a loop structure. 

Figure II shows the graph for typical if/else branch structure. Same as above, we notice Node 3
has the edge to Node 4, which is visited but not node 3’s ancestor. That’s the feature of the
branch in CFG. 

###Algorithm 
Above examples could be utilized to illustrate my idea. 

1.	Firstly conduct Breath-First-Search on Control Flow Graph. We divide into following two cases.  

	* A node is connected with a visited node, and the visited node is its ancestor. Mark the visited
	*__inLoop__*, such Node 1 in figure 1. And calculate *__outLoop__* node, such as Node 4 in Figure I.  

	* A node is connected to a visited node, but visited one is not its ancestor. We mark these two
	nodes nearest common ancestor node as *__inBranch__*. For example, Node 1 is common
	ancestor of 4 and 3 in figure II. We mark the visited one as *__outbranch__*, Node 4 in Figure II. 

2.	 We conduct basic block traversal on function like this. Keep the structural information as a tree consisting of TreeNode. After initializing the stack, we
	iterate the basic blocks in a function.  

	* If it’s marked as *__inLoop/inBranch__*, construct a new TreeNode, and make it the child of the top
	TreeNode on the stack, then push it into the Stack.  

	* If it’s marked as *__outLoop/outBranch__*, we pop the top TreeNode from the stack. 

3. For each instruction in the basic block, if it’s MPI calling instruction, add a new TreeNode
denoting the MPI callings and make it the child of the top TreeNode on the stack. 

4. When all of the basic blocks have been iterated, the last tree TreeNode left in the stack is the
root of the tree. 

###Some Details 

1. We could easily adapt it to inter procedure analysis. If the instruction in the basic block is call
instruction, we could traverse called function and append the result to the top node on stack. 

2.	As for calculation of the node marking as *__outLoop__*, it’s a little tricky. See figure I, you may
	assume *__outLoop__* node is another node connected with loop header (node 1). This works well
	for C programs compiled by LLVM. But it’s different for FORTRAN.  

	Compiler would convert `while{do}` to `if(cond){do{}while(cond)}` structure. So
	the node connected with *__outbranch__* node is at the end of the loop body.
	The more general way to deal with this problem could be this:   
	Let q is the current node, and is connected with p which is q’s ancestor.
	Each node which is both q’s ancestor and p’s sons must fall in the loop circuit.
	From these nodes, find the edge out of the loop circuit, like edge(u->v), v is not within loop. Then
	v could be marked as *__outLoop__*. 

###Implementation

I implement above idea as the form of two LLVM Passes, one for Breath-First-Search and another
for maintaining calling extraction stack. After finishing our project, I may consider to publish it on
github.

