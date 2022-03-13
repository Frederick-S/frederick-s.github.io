title: CF 和 OF 标志位
tags:
- CPU
- Assembly Language
---

看汇编语言时看到，标志寄存器中 `CF` 标志位表示无符号数运算时是否向最高有效位外的更高位产生进位或借位，而 `OF` 标志位表示有符号数运算时是否产生溢出。这里存在两个疑问：

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

最后的 `int $0x80` 中的 `int` 表示 `interrupt`，即中断，当发生一个中断时会有一个与之对应的中断处理程序来处理，这里的 `$0x80` 就是声明由哪个中断处理程序处理，在 `Linux` 中，`$0x80` 对应的是操作系统内核，用于发起一个系统调用，而具体发起哪个系统调用则由 `eax` 中的值决定，这就是 `movl $1, %eax` 的作用，1对应的系统调用是 `exit`，用于退出程序，而程序退出时会伴有一个状态码，这个状态码的值来自于 `ebx`，也就是 `movl $0, %ebx` 的作用，这里使用0来表示程序正常退出。

接下来我们借助 `gdb` 来观察程序运行时 `CF` 的值的变化。首先将上述代码保存为 `demo.s` 后进行编译：

```
as demo.s -o demo.o -gstabs+
```

这里的 `-gstabs+` 表示生成机器码时同时生成调试信息，如果没有这个选项后续 `gdb` 加载时会提示 `(No debugging symbols found in ./demo)`。

然后进行链接：

```
ld demo.o -o demo
```

这个时候就可以通过 `gdb` 加载生成的可执行文件：

```
gdb ./demo
```

![alt](/images/cf-of-1.png)

然后输入 `break 4` 在代码第四行设置一个断点，即 `mov $255, %al` 处，最后输入 `run` 开始调试执行：

![alt](/images/cf-of-2.png)

此时可输入 `layout reg` 来观察各寄存器内的值，我们需要关注的是 `eflags` 寄存器，它展示了哪些标志位生效了：

![alt](/images/cf-of-3.png)

或者通过执行 `info registers eflags` 来查看 `eflags` 的值：

```
(gdb) info registers eflags
eflags         0x202               [ IF ]
```

目前只有一个 `IF` 标志位，它用于表示是否响应中断。

接着，输入 `next` 来执行当前断点所在处的指令，可以看到，执行后 `rax` 寄存器内的值变成了255（`rax` 是64位 `CPU` 下的一个通用寄存器，32位 `CPU` 下对应为 `eax`）：

![alt](/images/cf-of-4.png)

再输入一次 `next` 来执行加法运算，此时 `rax` 中的值变为了0（实际的二进制结果应该是100000000，因为 `al` 寄存器最多只能表示8位，所以最高位的1无法表示，最终结果为0），`eflags` 中出现了 `CF` 标志位，说明发生了进位：

![alt](/images/cf-of-5.png)

`rax` 中的值为0也说明了加法运算后产生的进位并不会体现在比参与运算的寄存器位数更多的寄存器中，否则 `rax` 中的值应该是256。

再来看借位，将程序稍加修改执行一个 `1 - 2` 的运算：

```
.section .text
.globl _start
_start:
    mov $1, %al
    sub $2, %al
    movl $1, %eax
    movl $0, %ebx
    int $0x80
```

最后 `rax` 中的值为255（存在高位借位的情况下最后的二进制结果为11111111，解释为无符号数为255），`eflags` 中同样出现了 `CF` 标志位。

![alt](/images/cf-of-6.png)

所以，`CF` 的标记取决于两个二进制数的运算是否产生进位或借位。

## OF
有符号数的溢出分两种情况，一种是运算结果应该是正数却返回负数，另一种是运算结果应该是负数却返回正数。

首先来看两个正数运算得到负数的例子，同样对代码稍加修改实现 `127 + 1` 的运算：

```
.section .text
.globl _start
_start:
    mov $127, %al
    add $1, %al
    movl $1, %eax
    movl $0, %ebx
    int $0x80
```

最后 `rax` 中的值为128（对应二进制表示为10000000，以有符号数的角度来看，其值为-128，即两个正数相加得到一个负数），`eflags` 中出现了 `OF` 标志位，说明发生了溢出：

![alt](/images/cf-of-7.png)

从有符号数的角度来看，参与运算的两个数的符号位都是0，相加后符号位却是1，所以 `OF` 设置为1。

再来看两个负数运算得到正数的例子，再次对代码稍加修改实现 `-128 - 1` 的运算，-128的二进制补码表示为10000000，即无符号数角度下的128，-1的二进制补码表示为11111111，即无符号数角度下的255：

```
.section .text
.globl _start
_start:
    mov $128, %al
    add $255, %al
    movl $1, %eax
    movl $0, %ebx
    int $0x80
```

最后 `rax` 中的值为127（对应二进制表示为01111111，以有符号数的角度来看，其值为127，即两个负数相加得到一个正数），`eflags` 中出现了 `OF` 标志位，说明发生了溢出：

![alt](/images/cf-of-8.png)

从有符号数的角度来看，参与运算的两个数的符号位都是1，相加后符号位却是0，所以 `OF` 设置为1。

所以，`OF` 的标记取决于运算结果的符号位是否发生变化，这里说的变化关心的是

参考：

* [The CARRY flag and OVERFLOW flag in binary arithmetic](http://teaching.idallen.com/dat2343/10f/notes/040_overflow.txt)
* [Eflags Registers](https://niranjanmr.wordpress.com/2016/01/20/eflags-registers/)
* [What does "int 0x80" mean in assembly code?](https://stackoverflow.com/questions/1817577/what-does-int-0x80-mean-in-assembly-code)
* 汇编语言（第4版），王爽