title: CF 和 OF 标志寄存器
tags:
- CPU
- Assembly Language
---

看汇编语言时看到，`CF` 标志寄存器表示无符号数运算时是否向最高有效位外的更高位产生进位或借位，而 `OF` 标志寄存器表示有符号数运算时是否产生溢出。这里存在两个疑问：

1. 对于 `CPU` 来说，它并不区分处理的是无符号数还是有符号数，

参考：

* [The CARRY flag and OVERFLOW flag in binary arithmetic](http://teaching.idallen.com/dat2343/10f/notes/040_overflow.txt)
* [Eflags Registers](https://niranjanmr.wordpress.com/2016/01/20/eflags-registers/)
* [What does "int 0x80" mean in assembly code?](https://stackoverflow.com/questions/1817577/what-does-int-0x80-mean-in-assembly-code)
* 汇编语言（第4版），王爽