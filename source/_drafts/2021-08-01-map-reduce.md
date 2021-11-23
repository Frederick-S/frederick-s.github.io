title: 'MIT 6.824 Lab 1 (1) - MapReduce: Simplified Data Processing on Large Clusters'
tags:
- Paper
- MIT 6.824
---

## 介绍
`MapReduce: Simplified Data Processing on Large Clusters` 是 [6.824: Distributed Systems](https://pdos.csail.mit.edu/6.824/) 中所介绍的第一篇论文。它提出了一种针对大数据处理的编程模型和实现，使得编程人员无需并行和分布式系统经验就可以轻松构建大数据处理应用。该模型将大数据处理问题拆解为两步，即 `Map` 和 `Reduce`，`Map` 阶段将一组输入的键值对转化为中间结果键值对，`Reduce` 阶段对中间结果键值对按照相同的键进行值的合并，从而得到最终的结果。

对于 `Google` 来说，每天运行的系统会产生大量的原始数据，同时又要对这些原始数据进行加工产生各种衍生数据，虽然大部分数据加工的逻辑都较为简单，然而由于数据量过于庞大，为了在合理的时间内完成数据处理，通常需要将待处理的数据分发到几百或几千台机器上并行计算。这就存在几个问题要解决：

1. 如何使计算可并行
2. 如何分发数据
3. 如何处理异常

如果每一个数据加工任务都需要独立去解决上述的问题，一方面会使得原本简单的代码逻辑变得庞大、复杂和难以维护，另一方面也是在重复工作。因此，`Google` 的工程师们抽象出了 `MapReduce` 框架，对应用开发人员屏蔽了运行细节，由框架层统一处理并行、容错、数据分发和负载均衡。

## 编程模型

参考：

- [MapReduce: Simplified Data Processing on Large Clusters](https://research.google/pubs/pub62/)
