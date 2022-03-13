title: CF 和 OF 标志寄存器
tags:
- CPU
- Assembly Language
---

看汇编语言时看到，`CF` 标志寄存器表示无符号数运算时是否向最高有效位外的更高位产生进位或借位，而 `OF` 标志寄存器表示有符号数运算时是否产生溢出。这里存在两个疑问：

1. 对于 `CPU` 来说，它并不区分处理的是无符号数还是有符号数，那什么时候设置 `CF`，什么时候设置 `OF` 呢
2. `CF` 表示进位时也是一种溢出，能否和 `OF` 共用一个

## CF
首先来看 `CF` 进位的例子，这里我们以8位无符号数为例，其最大值为255，那么计算 `255 + 1` 则会产生进位。可以通过一段简单的汇编代码进行验证：

```
.section .text
.globl _start
_start:
    mov $255, %al
    add $1, %al
    movl $1, %eax
    movl $0, %ebx
    int $0x80
```

在上述代码中，`al` 是一个8位寄存器，是 `eax` 寄存器的低8位，这里首先将255放到 `al` 寄存器内，然后对 `al` 寄存器中的值加1并放回到 `al` 寄存器中，即实现 `255 +1` 的运算。

最后的 `int $0x80` 中的 `int` 表示 `interrupt`，即中断，当发生一个中断时会有一个与之对应的中断处理程序来处理，这里的 `$0x80` 就是声明由哪个中断处理程序处理，在 `Linux` 中，`$0x80` 对应的是操作系统内核，用于发起一个系统调用，而具体发起哪个系统调用则由 `eax` 中的值决定，这就是 `movl $1, %eax` 的作用，1对应的系统调用是 `exit`，从而退出了程序，而程序退出时会伴有一个状态码，这个状态码的值来自于 `ebx`，也就是 `movl $0, %ebx` 的含义，这里使用0来表示程序正常退出。

参考：

* [The CARRY flag and OVERFLOW flag in binary arithmetic](http://teaching.idallen.com/dat2343/10f/notes/040_overflow.txt)
* [Eflags Registers](https://niranjanmr.wordpress.com/2016/01/20/eflags-registers/)
* [What does "int 0x80" mean in assembly code?](https://stackoverflow.com/questions/1817577/what-does-int-0x80-mean-in-assembly-code)
* 汇编语言（第4版），王爽