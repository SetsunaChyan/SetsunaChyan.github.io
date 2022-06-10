---
title: "CSAPP笔记 Machine Level Programming"
categories: [Notes]
tags: [CSAPP]
toc: true
classes: []
excerpt: "AT&T 汇编"
typora-root-url: ./
---

### Basics

#### Registers

x86-64 架构下，寄存器有 %rax(accumulate),  %rcx(counter), %rdx(data), %rbx(base), %rsi(source index), %rdi(destination index), %rsp(stack pointer), %rbp(base pointer) 以及 %r8 到 %r15 共 16 个。

每个寄存器为 64 位，前 8 个寄存器以 %rax 为例，低 32 位为 %eax; 低 16 位为 %ax; 16 位中的高八位为 %ah 第 8 位为 %al（仅前 4 个寄存器）。

寄存器命名有历史原因，现除 %rsp 外均为通用寄存器。除此之外还有寄存器 %rip(instruction pointer) 存储下一条指令的地址。



#### Operations

指令 `movq Src, Dest`

可以是 Immediate(如 \$0x400)，Register (如 %rax，但不要使用 %rsp)，Memory (如 (%rax) )。

Dest 不能是立即数，同时也不允许直接内存到内存，所以一共有 5 种可能。

```c
void swap(int *x,int *y) {
    int t1=*x;
    int t2=*y;
    *x=t2;
    *y=t1;
}
```

这个函数经过 `gcc -Og -S` 得到的汇编是

```assembly
movl    (%rdi), %eax
movl    (%rsi), %edx
movl    %edx, (%rdi)
movl    %eax, (%rsi)
ret
```

因为不能内存到内存，所以写经典的三行版 swap 在优化后也会得到同样的汇编。

一些其他算术运算指令：

+ `addq`, `subq`,  `imulq` 分别是加减乘
+ `salq`, `sarq`, `shrq` 分别是左移、算术右移（补符号位）、逻辑右移（补零）
+ `xorq`, `andq`, `orq` 分别是按位异或、与、或

主要注意运算顺序，所有的都是 `Dest = Dest op Src` ，与 Intel 是反过来的。



#### Memory Addressing Mode

`D(Rb,Ri,S)` 对应 `Mem[Reg[Rb]+S*Reg[Ri]+D]`

+ `D` Displacement，通常为 1, 2, 4 字节。

+ `Rb` Base Register，十六个基本寄存器均可。

+ `Ri` Index Register，除去 %rsp 均可。

+ `S` Scale，一般为 1, 2, 4, 8。

例子：假设 `%rdx=0xf000,%rcx=0x0100`，那么有：

+ `0x80(,%rdx,2)=2*0xf00+0x80=0x1e080` 
+ `(%rdx,%rcx,4)=0xf000+4*0x0100=0xf400`

指令 `leaq Src,Dst` 经常被利用来计算形如 $x+k\times y$ 的表达式。

```c
long m12(long x) {
    return x * 12;
}
```

经过优化后可能是

```assembly
leaq (%rdi,%rdi,2), %rax  # %rax = %rdi + 2 * %rdi
salq $2, %rax             # %rax <<= 2 
```

它与 `mov` 指令不同的是，`lea` 是将计算出来的值赋值给 `Dst` ，而 `mov` 是将计算出来的值作为地址，从内存中取出这个地址的值赋值给 `Dst`。



### Control

#### Condition Codes

有 4 个重要的 Single Bit Register，算数运算时会隐式地为它们赋值。

+ `CF` Carry Flag (for unsigned)，无符号是否进位
+ `ZF` Zero Flag，是否为 0
+ `SF` Sign Flag (for signed)，是否小于 0
+ `OF` Overflow Flag (for signed)，有符号是否溢出

不过注意 `lea` 这类并不会设置这些 flag。

`cmpq Src2, Src1` ，计算 `Src1 - Src2` 并为上述 flag 赋值。

`testq Src2, Src1` ，计算 `Src2 & Src1` 。

通过下表可以使用上述 flag，只会改变最低字节。

<img src="/assets/images/posts/CSAPP/lec6_condition_codes.png" style="zoom: 67%;" />

一个简单的例子

```c
int gt(long x, long y) {
    return x > y;
}
```

```assembly
# %rdi = x, %rsi = y
cmpq %rsi, %rdi
setg %al
movzbl %al, %eax
```

通常使用 `movzbl` 赋值，高位会被清零。此外 32 位赋值给 64 位时也会把高位清零。



#### Condition Branches

程序通过 jX 指令来体现不同的控制流，具体如下表

<img src="/assets/images/posts/CSAPP/lec6_jx_instructions.png" style="zoom: 67%;" />

一个简单的例子

```c
long absdiff(long x, long y) {
    long result;
    if (x > y) result = x - y;
    else result = y - x;
    return result;
}
```

```assembly
# -Og:
# %rdi = x, %rsi = y
        movq    %rdi, %rax
        cmpq    %rsi, %rdi
        jle     .L2           # x <= y 则跳转至 .L2
        subq    %rsi, %rax
        ret
.L2:
        subq    %rdi, %rsi
        movq    %rsi, %rax
        ret
# -O2:
        movq    %rdi, %rdx
        movq    %rsi, %rax
        subq    %rsi, %rdx
        subq    %rdi, %rax
        cmpq    %rsi, %rdi
        cmovg   %rdx, %rax
        ret
```

一般来说对于分支语句，会有两种转换形式

```c
	val = test ? Then_Expr : Else_Expr;
	Other_Expr;
/*-------------------*/
    int ntest = !test;
    if(ntest) goto Else;
    Then_Expr;
    goto Done;
Else:
	Else_Expr;
Done:
	Other_Expr;
/*-------------------*/
	int ntest = !test;
	val = Then_Expr;
	eval = Else_Expr;
	if(ntest) val = eval;
	Other_Expr;
```

通常第二种转换形式效率更高，原因在于充分利用 CPU 流水线与分支预测。但是以下几种情况不能优化：

+ `val = Test(x) ? Hard1(x) : Hard2(x)` 运算比较复杂
+ `val = p ? *p : 0` 不安全
+ `val = x > 0 ? x*=7 : x+=3` 有副作用



#### Loops

"Do-While" 模式 与 "Jump-to-middle" 模式。

感觉两者没本质区别。

```c
while (Test)
    Body;
/*-------------------*/
	goto test;
loop:
	Body;
test:
	if(test)
        goto loop;
```

对于 `do-while` 和 `for` 只需要对上述结构做一点小修改即可。



#### Switch Statements

```c
long switch_eg(long x, long y, long z) {
    long w = 1;
    switch (x) {
    case 11:
        w = y * z;
        break;
    case 13:
        w = y / z; /* Fall Through */
    case 14:
        w += z;
        break;
    case 15:
    case 17:
        w -= z;
        break;
    default:
        w = 2;
    }
    return w;
}
```

在 case 比较稠密时，会生成一个 Jump Table 维护每个 case 该调往哪个代码段。

前半段会变成这样：

```assembly
.LFB0:
	.cfi_startproc
	endbr64
	subq	$11, %rdi
	movq	%rsi, %rax
	movq	%rdx, %rcx
	cmpq	$6, %rdi
	ja	.L8
	leaq	.L4(%rip), %rsi
	movslq	(%rsi,%rdi,4), %rdx
	addq	%rsi, %rdx
	notrack jmp	*%rdx
	.section	.rodata
	.align 4
	.align 4
```

减去 11 是为了把标号从 $[11,17]$ 映射到 $[0,6]$ 。后面的 `cmpq` 与 `ja` 是个很巧妙的点，因为 `ja` 是无符号的，也就是说如果 `%rdi-6` 是负的或者大于 6 的都会跳转至 `.L8` 这个 default 段里。如果未跳转则根据跳转表跳转到相应的分支。这样跳转是 $O(1)$ 的。

```assembly
.L4:  # Jump Table
	.long	.L7-.L4
	.long	.L8-.L4
	.long	.L6-.L4
	.long	.L9-.L4
	.long	.L3-.L4
	.long	.L8-.L4
	.long	.L3-.L4
	.text
	.p2align 4,,10
	.p2align 3
```

由于并不是每个分支都需要给 `w` 赋值为 1，所以 `w=1` 的作用可能被延迟到需要的段中了。

比如 `x=13` 的分支就没有（这里编译器选择将 `x=14` 的部分复制了一份塞进 `x=13` 的分支里），但 `x=17` 或 `x=14` 的分支有。

```assembly
.L6: # x = 13 
	cqto
	idivq	%rcx
	addq	%rcx, %rax
	ret
.L9: # x = 14
	movl	$1, %eax
	addq	%rcx, %rax
	ret
.L3: # x = 15 or x = 17
	movl	$1, %eax
	subq	%rcx, %rax
	ret
```

当 case 的值稀疏时，会退化成 `if-else` ，编译器会恰当地构建决策树使得高度是 $O(\log n)$ 的。

