---
layout: post
category : 
tags : [assembly]
---

This is our Network Security Class's experiement, also the most interesting one in my all colledge experiments assignments.  
The problem description:  
Given a segment of code, change the returning instruction to execute after function calling.  

My code is following:  

	#include <stdio.h>
	
	void foo(){
		 int a, *p;
		 p = (int*)((int)&a + 8);
		 *p += 12;
	}

	int main(){
		foo();
		printf("First printf call\n");
		printf("Second printf call\n");
		return 0;
	}

After compiling, the output is merely `Second printf call`, instead of displaying two output strings both.  
The reason is quite simple,	revising calling site's returning address in function foo()'s body.

Detailed process:  
Firstly, compile source to assembly source by `gcc -S main.c`. And you could get following assembly segment:  

	080483e4 <foo>:
	 80483e4:    55                       push   %ebp
	 80483e5:    89 e5                    mov    %esp,%ebp
	 80483e7:    83 ec 10                 sub    $0x10,%esp
	 80483ea:    8d 45 fc                 lea    -0x4(%ebp),%eax
	 80483ed:    83 c0 08                 add    $0x8,%eax
	 80483f0:    89 45 f8                 mov    %eax,-0x8(%ebp)
	 80483f3:    8b 45 f8                 mov    -0x8(%ebp),%eax
	 80483f6:    8b 00                    mov    (%eax),%eax
	 80483f8:    8d 50 0c                 lea    0xc(%eax),%edx
	 80483fb:    8b 45 f8                 mov    -0x8(%ebp),%eax
	 80483fe:    89 10                    mov    %edx,(%eax)
	 8048400:    c9                       leave  
	 8048401:    c3                       ret    

	08048402 <main>:
	 8048402:    55                       push   %ebp
	 8048403:    89 e5                    mov    %esp,%ebp
	 8048405:    83 e4 f0                 and    $0xfffffff0,%esp
	 8048408:    83 ec 10                 sub    $0x10,%esp
	 804840b:    e8 d4 ff ff ff           call   80483e4 <foo>
	 8048410:    c7 04 24 f0 84 04 08     movl   $0x80484f0,(%esp)
	 8048417:    e8 fc fe ff ff           call   8048318 <puts@plt>
	 804841c:    c7 04 24 02 85 04 08     movl   $0x8048502,(%esp)
	 8048423:    e8 f0 fe ff ff           call   8048318 <puts@plt>
	 8048428:    b8 00 00 00 00           mov    $0x0,%eax
	 804842d:    c9                       leave  
	 804842e:    c3                       ret    
	 804842f:    90                       nop

From above assembly, we could find the address of instruction right after foo() calling is 0x8048410, and the second 
printf() calling's address is 0x804841c. The value difference is 12, so we should add 12 to returing address.

Before go on next section, let's review some basic knowledge about function calling detail.  
<p>
	<img src="/images/2012-05-19-1.png"/>
</p>

The procedure in C calling convention is:  
1.	caller push arguments from right to left into stack  
2.	CALL instruction is invoked.  
And call instruction would push the returing address onto stack, then revise the eip register, which stores the Program Counter.

And we notice in above code segment,  

	80483ea: 8d 45 fc lea -0x4(%ebp),%eax


	
Value of memory unit locating at -4(%ebp) was assigned to eax register. Corresponding C code is `p = &a;`
so we now know that the local varaible `a` in foo() is stored at -4(%ebp), that is the position of Local Variable 1 in above picture.  
Returning address is 8 byte offset higher(in x86 arch), relative to local variable a.  
Expression `p = &a + 8` calculates the unit that stores returning address.   
At last, it's no wonder `*p += 12` changes returning address.(12 is analyzed from above section).

We finish all analysis. :-)

