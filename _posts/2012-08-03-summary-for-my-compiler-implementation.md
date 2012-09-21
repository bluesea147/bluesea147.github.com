---
layout: post
category : 
tags : [compiler]
---

The long time preparation for GRE nearly strains me up, and I want to do some coding work. After disscussing with my classmate, Gabriel,
I got the idea to implement a compiler.

Compiler, just as the dragon in the `Dragon Book`, is unimaginable at first. But after digesting some resources, it's more touchable.
Following material may help,

+	Dragon Book
+	Linkers and Loaders
+	Professional Assembly Language
+	Intel x86 developer manual 

Then the process becomes clearer. Define the AST data structure, using the LL(1) technique to build parser, code generation for each kind
of AST, convert assembly code to binary, link.

I once want to build the compiler independly, doing linking and binary generation by myself. Well, after get known about the detail of linking
and x86 instruction, it's not funny to handle this trouble work. Finally, I use as and ld to solve that.

There is not much to say about parser as I apply LL(1). Code generation for loop, branch is also easy. Only interesting thing is code generation for
expression. I refer the approach in Dragon Book, that recursive generation on expression eshov tree.
So I decide to transform the expression to posix expr, which naturaly forms a tree. Handling parsing work, and posix transformation at the same time
is troublesome. So i add another module, posix.c.

To give a straightforward effect, the IO function is necesssary. I add io module to implement io function, implemented in assembly.
Not much to say, it's interesting to do it from without by oneself. :-)
