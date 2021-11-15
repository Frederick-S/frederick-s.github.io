title: 'MIT 6.824 Lab 1 (1) - MapReduce: Simplified Data Processing on Large Clusters'
tags:
- Paper
- MIT 6.824
---

`MapReduce: Simplified Data Processing on Large Clusters` 是 [6.824: Distributed Systems](https://pdos.csail.mit.edu/6.824/) 中所介绍的第一篇论文。它提出了一种针对大数据处理的编程模型和实现，使得编程人员无需并行和分布式系统经验就可以轻松构建大数据处理应用。该模型将大数据处理问题拆解为两步，即 `Map` 和 `Reduce`，`Map` 阶段将一组输入的键值对转化为中间结果键值对，`Reduce` 阶段对中间结果键值对按照相同的键进行值的合并，从而得到最终的结果，这个过程涉及到三组数据：

1. 输入键值对
2. 中间结果键值对
3. 最终结果键值对

参考：

- [MapReduce: Simplified Data Processing on Large Clusters](https://research.google/pubs/pub62/)
