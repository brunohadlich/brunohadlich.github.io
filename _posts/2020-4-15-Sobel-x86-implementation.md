---
layout: post
title: Implementing Sobel algorithm in x86-64 assembly
---

Recently I decided to improve my knowldge about x86-64 assembly, one thing I realized is that assembly as many other things
in life is not that easy to learn just by reading, practicing is the best way to in fact achieve a faster and sharper
reasoning. With this is mind I decided to implement an small algorithm called sobel, I considered this to be a good
choice because it is small and easy to understand, the idea behind this algorithm is putting in evidence the borders of an
image, so I will start by presenting the algorithm written in C.

```c
volatile void *sobel(uint8_t *input, uint8_t *output, uint16_t rows, uint16_t cols) {
  int kernel_x[] = {
    -1, 0, 1,
    -2, 0, 2,
    -1, 0, 1
  };
  int kernel_y[] = {
    -1, -2, -1,
    0, 0, 0,
    1, 2, 1
  };
  int r, c, a, b, gx, gy, kernel_idx, input_idx, input_value;
  for (r = 1; r < rows - 1; r++) {
    for (c = 1; c < cols - 1; c++) {
      gx = gy = 0;
      for (a = 0; a < 3; a++) {
        for (b = 0; b < 3; b++) {
          kernel_idx = a * 3 + b;
          input_idx = ((r - 1 + a) * cols + c - 1) + b;
          input_value = input[input_idx];
          gx += input_value * kernel_x[kernel_idx];
          gy += input_value * kernel_y[kernel_idx];
        }
      }
      output[r * cols + c] = sqrt(gx * gx + gy * gy);
    }
  }
}
```

This is the assembly version.

```nasm
	.global sobel

	.text
sobel:
	pushq %rbp
	movq %rsp, %rbp

	pushq %rax
	pushq %rbx
	pushq %rcx
	pushq %rsi

	movq %rdi, input
	movq %rsi, output
	movw %dx,  rows
	movw %cx,  cols

	decw cols
	decw rows

	r0:
	movl $1, r
	r1:

	c0:
	movl $1, c
	c1:

	movl $0, gx
	movl $0, gy

	a0:
	movl $0, a
	a1:

	b0:
	movl $0, b
	b1:

	//int kernel_idx = a * 3 + b;
	movl $3, %eax
	mull a
	addl b, %eax
	movl %eax, kernel_idx
	//int kernel_idx = a * 3 + b;

	//int input_idx = ((r - 1 + a) * cols + c - 1) + b;
	movl r, %eax
	decl %eax
	addl a, %eax
	movl %eax, %ecx
	movzwl cols, %ebx
	mull %ebx
	addl %ecx, %eax
	addl c, %eax
	decl %eax
	addl b, %eax
	movl %eax, input_idx
	//int input_idx = ((r - 1 + a) * cols + c - 1) + b;

	//input_value = input[input_idx]
	movq $0, %rsi
	movl input_idx, %esi
	movq input, %rbx
	movzbl (%rbx, %rsi), %eax
	movl %eax, input_value
	//input_value = input[input_idx]

	//gx += input_value * kernel_x[kernel_idx] 
	movq $0, %rsi
	movl kernel_idx, %esi
	movq $kernel_x, %rbx
	movl (%rbx,%rsi,4), %eax
	imull input_value, %eax
	addl %eax, gx
	//gx += input_value * kernel_x[kernel_idx]

	//gy += input_value * kernel_y[kernel_idx]
	movq $0, %rsi
	movl kernel_idx, %esi
	movq $kernel_y, %rbx
	movl (%rbx,%rsi,4), %eax
	imull input_value, %eax
	addl %eax, gy
	//gy += input_value * kernel_y[kernel_idx]

	incl b
	cmpl $3, b
	jl b1

	incl a
	cmpl $3, a
	jl a1

	//output[r * cols + c] = sqrt(gx * gx + gy * gy);
	//gx * gx
	movl gx, %eax
	mull %eax
	movl %eax, gx
	//gy * gy
	movl gy, %eax
	mull %eax
	//eax = gx * gx + gy * gy
	addl gx, %eax

	//sqrt(gx * gx + gy * gy)
	cvtsi2sd	%eax, %xmm0
	sqrtsd		%xmm0, %xmm0
	cvttsd2si	%xmm0, %ecx

	//r * cols + c
	movl r, %eax
	movzwl cols, %ebx
	mull %ebx
	addl r, %eax
	addl c, %eax
	movq $0, %rsi
	movl %eax, %esi

	//output[r * cols + c] = sqrt(gx * gx + gy * gy);
	movq output, %rbx
	movb %cl, (%rbx, %rsi)

	incl c
	movzwl cols, %ecx
	cmpl %ecx, c
	jl c1

	incl r
	movzwl rows, %ecx
	cmpl %ecx, r
	jl r1

	popq %rsi
	popq %rcx
	popq %rbx
	popq %rax

	movq %rbp, %rsp
	popq %rbp

	ret

	.data
kernel_x:	.long	-1, 0, 1, -2, 0, 2, -1, 0, 1
kernel_y:	.long	-1, -2, -1, 0, 0, 0, 1, 2, 1

	.bss
.lcomm input, 8
.lcomm output, 8
.lcomm rows, 2
.lcomm cols, 2
.lcomm kernel_idx, 4
.lcomm input_idx, 4
.lcomm input_value, 4
.lcomm r, 4
.lcomm c, 4
.lcomm a, 4
.lcomm b, 4
.lcomm gx, 4
.lcomm gy, 4
```

Now I want to talk about some issues that I faced during the development, the first one was declaring variables, you can see
they are splitted in two segments, data and bss, my first try was to put all variables into data segment and define them
like:

input: .quad  
output: .quad  
rows: .word   
...  
b: .long  
gx: .long  
gy: .long  

The issue is that as I was not defining a value for these variables, consequently the compiler assumed they all referred
to the same memory position, it may sound absurd but it took me hours to understand that I should have defined my variables
differently to make the compiler reserve space for each of them separately.

After fixing the issue with the variables the code was not presenting the same result as the C code yet.

The second issue was related with memory indexing, when I had to multiply ```input_value * kernel_x[kernel_idx]```, I moved
kernel_x[kernel_idx] to eax register with instruction ```movl (%rbx,%rsi), %eax``` instead of ```movl (%rbx,%rsi,4), %eax```
the problem of not adding the scale factor of 4 into account is that my kernel_x variable is an array of data type long(4
bytes) and in this case I was moving completely wrong values to eax causing the result to be a huge mess.

This assembly code was done with the purpose of only matching the same results of the C code, not to be fast, in fact the C
code compiled with flag O3 is ~10x faster the the assembly I wrote. Now that I did achieve my goal of making an assembly
working version, next step is to optimize it and make it faster by using more registers to reduce memory accesses, removing
redundant code, maybe using loop unrolling and other kinds of optimization techniques.

If you would like to test this code, clone the [repo](https://github.com/brunohadlich/image_processing_tests), run ```make```
to generate 'main' executable or ```make asm``` to generate 'asm_main' executable. Both files when executed will read 
images/Full_HD_1920x1080.bmp and generate the sobel output in images/img_mod.bmp based.
