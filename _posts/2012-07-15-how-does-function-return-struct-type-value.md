---
layout: post
category : 
tags : [assembly]
---

After learning assembly language, I get to know compiler would use eax register to store returning value.
But what if the returning type is a compact struct whose size is greater than the size of register eax? And how to deal with usage of the returning value as assignment expression's right operands.  

To figure out answer, I have done simple experiments.
Fristly, prepare the following C code:

	#include <stdio.h>
	struct A{
		int a;
		char c;
	};

	struct A add(int a, int b){
		struct A t;
		t.a = a*b;
		return t;
	}

	int main(){
		struct A t = add(3, 4);
		printf("%c\n", t.c);
		return 0;
	}

And the assembly source generated by gcc is like following:

	.globl add
		.type    add, @function
	add:
		pushl    %ebp
		movl    %esp, %ebp
		subl    $16, %esp

		8(%ebp) is the last argument's address, storing address of the variable assigned by returning value, 
		namely the eax value pushed in main()

		# unit 8(%ebp) contains last argument, storing the address of local varaible to be assigned,   
		namely the value in eax in main()
		movl    8(%ebp), %ecx 

		#extract second argument
		movl    12(%ebp), %eax 

		extract first argument
		imull    16(%ebp), %eax 

		# -8(%ebp) to (%ebp) corresponds to each filed of struct
		movl    %eax, -8(%ebp) 

		# begin to assign each field to caller's local variable, ecx containing address
		movl    -8(%ebp), %eax 
		movl    -4(%ebp), %edx
		movl    %eax, (%ecx) 
		movl    %edx, 4(%ecx)
		movl    %ecx, %eax
		leave
		ret    $4
	.LC0:
		.string    "%c\n"
	.globl main
		.type    main, @function
	main:
		leal    4(%esp), %ecx
		andl    $-16, %esp
		pushl    -4(%ecx)
		pushl    %ebp
		movl    %esp, %ebp
		pushl    %ecx
		subl    $36, %esp 
		leal    -16(%ebp), %eax

		#put arguments onto stack, using mov instruction after  
		 reserving space for arguments
		movl    $4, 8(%esp) 
		movl    $3, 4(%esp)
	    #put address of local vairable assigned onto stack, callee function would use it to implement assignemnt
		movl    %eax, (%esp) 
		call    add
		subl    $4, %esp
		movzbl    -12(%ebp), %eax
		movsbl    %al,%edx
		movl    $.LC0, %eax
		movl    %edx, 4(%esp)
		movl    %eax, (%esp)
		call    printf
		movl    $0, %eax
		movl    -4(%ebp), %ecx
		leave
		leal    -4(%ecx), %esp
		ret

From above analysis, we could find returning struct-type value of function and its assignment is implemented by pass local variable's 
address to callee in eax register.  
A friend has pointed out M$ compiler behaves differently, it would reallocate temporary space in callee's stack and done once copy.  
Compared with that way, gcc's approach is more conveient and efficient.
