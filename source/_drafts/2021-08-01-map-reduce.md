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

如果每一个数据加工任务都需要独立去解决上述的问题，一方面会使得原本简单的代码逻辑变得庞大、复杂和难以维护，另一方面也是在重复工作。受 `Lisp` 等其他函数式编程语言中的 `map` 和 `reduce` 函数的启发，`Google` 的工程师们发现大部分的数据处理遵循如下的模式：

1. 对输入的每一条数据应用一个 `map` 函数产生一组中间结果键值对
2. 对中间结果键值对按照相同的键聚合后，应用 `reduce` 函数生成最终的衍生数据

因此，`Google` 的工程师们抽象出了 `MapReduce` 框架，使得应用开发人员可以专注于计算逻辑实现而无需关心底层运行细节，统一由框架层处理并行、容错、数据分发和负载均衡等系统问题。现在再来看前面提到的问题是如何解决的：

1. 如何使计算可并行：在 `Map` 阶段，对数据分发后，各任务间无依赖，可并行执行；在 `Reduce` 阶段，不同 `key` 的数据处理间无依赖，可并行执行
2. 如何分发数据：在 `Map` 阶段，可按执行 `Map` 任务的节点数量平均分发；在 `Reduce` 阶段，可按 `key` 相同的数据聚合后分发
3. 如何处理异常：重新执行某个节点上失败的 `Map` 或 `Reduce` 任务作为首要的容错手段

## 编程模型
假设需要统计一组文档中每个单词出现的次数，在 `MapReduce` 框架下用户需要编写 `map` 和 `reduce` 函数，近似的伪代码表示如下：

```
map(String key, String value):
    // key: document name
    // value: document contents
    for each word w in value:
        EmitIntermediate(w, "1");

reduce(String key, Iterator values):
    // key: a word
    // values: a list of counts
    int result = 0;
    for each v in values:
        result += ParseInt(v);
    Emit(AsString(result));
```

假设有两个文档 `hello.txt` 和 `world.txt`，其内容分别为：

```
hello.txt:
It was the best of times

world.txt:
it was the worst of times
```

对上述 `map` 和 `reduce` 函数来说，`map` 函数每次处理一个文档，`key` 为文档的名称，`value` 为文档的内容，即：

```
map("hello.txt", "It was the best of times")
map("world.txt", "it was the worst of times")
```

`map` 函数执行时会遍历文档的内容，对每个单词输出中间结果键值对（作为示例，这里省去了将文档内容拆分为单词的过程，同时也忽略了标点符号、大小写等与示例无关的内容），键为单词，值为 `"1"`，所有 `map` 函数执行完成后生成的中间结果为：

```
hello.txt:
it 1
was 1
the 1
best 1
of 1
times 1

world.txt:
it 1
was 1
the 1
worst 1
of 1
times 1
```

然后，`MapReduce` 框架对所有中间结果按照相同的键进行聚合，即：

```
it ["1", "1"]
was ["1", "1"]
the ["1", "1"]
best ["1"]
worst ["1"]
of ["1", "1"]
times ["1", "1"]
```

最后，`MapReduce` 框架将上述聚合后的数据分发给 `reduce` 节点执行，即：

```
reduce("it", ["1", "1"])
reduce("was", ["1", "1"])
reduce("the", ["1", "1"])
reduce("best", ["1"])
reduce("worst", ["1"])
reduce("of", ["1", "1"])
reduce("times", ["1", "1"])
```

`reduce` 函数执行时会遍历 `values`，对当前单词出现的次数进行累加，然后作为 `reduce` 的结果返回，最终得到所有单词出现的次数：

```
it 2
was 2
the 2
best 1
worst 1
of 2
times 2
```

参考：

- [MapReduce: Simplified Data Processing on Large Clusters](https://research.google/pubs/pub62/)
