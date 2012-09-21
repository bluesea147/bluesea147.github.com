---
layout: post
category : 
tags : [assembly, compiler]
---

Ocassionaly, when I read Computer Science, a programmer's prospective, an interesting question appeals to me:  
`Is switch and if else statement's efficiency the same, or different?`  
But after google that, I found it is a common-known question. :-(  
Well, I could still take some notes. 

It could be divided into two senarios,

1.	the range of values in case label is very small, eg. \[a, b\] interval.   
	The compiler would substract the minimum value in all case number, to get offset. And construct the **jump table**, storing
	right address to jump at corresponding index position. For example, case values range is \[100, 110\], for `case 103`, it would
	do 103 - 100 = 3, and use `jmp 3(table_base)` to jump to right position.

	The absence of certain number in jump table would be filled with default label's address. And it explains why this method could 
	only applied in situation where the case number is basically continuous, not too discrete.

2.  When the range of values in case labels is large and values are discrete, above mechanics would not work. As it has to allocate size
	of the range for jumping table. In this case, **binary search** is applied. :-)  
	First sort the values, and then compare integer variable's value in binary search way, so we just need log(n) comparisions to determine which 
	label should jump to.



Equiped with this subtle implementation for switch statement, it is unlike if-else statement, just making comparision as source code directed.  
Now we know the differences. And I respect more towards people constructing compiler. :-)
