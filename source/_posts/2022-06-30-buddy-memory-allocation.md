title: 'Buddy Memory Allocation'
tags:
- Memory Allocation
---

## 介绍
`Buddy Memory Allocation` 是内存分配算法的一种，它假定内存的大小为$2^N$（`N` 为整数），并且总是以2的幂次方为单位分配或者释放内存。

## 算法
假设某个线程需要申请 `m` 字节内存，`Buddy Memory Allocation` 会先在当前所有的空闲空间中找到最小的空间满足$2^k \geq m$，如果$2^k$的一半依然大于等于 `m`，说明当前分配的空间过大，则继续将$2^k$对半分，直到找到最小的 `p`（$p \leq k$）满足$2^{p - 1} < m \leq 2^p$。

下图描述了从16字节中分配3字节的过程：

1. 初始状态整个内存只有16字节，是可分配的最小空间；不过由于16字节的一半大于3字节，所以将16字节拆分为两个8字节
2. 同理一个8字节的一半依然大于3字节，继续将其中一个8字节拆分为两个4字节
3. 4字节的一半比3字节小，所以4字节就是可分配的最小内存空间

![alt](/images/buddy-1.png)

## 参考
* [Buddy Memory Allocation](https://www.kuniga.me/blog/2020/07/31/buddy-memory-allocation.html)
* [Buddy Methods](https://opendsa-server.cs.vt.edu/ODSA/Books/Everything/html/Buddy.html#buddy-methods)