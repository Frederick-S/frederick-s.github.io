title: 'MIT 6.824 Lab 1 (1) - MapReduce: Simplified Data Processing on Large Clusters'
tags:
- Paper
- MIT 6.824
---

## 介绍
`MapReduce: Simplified Data Processing on Large Clusters` 是 [6.824: Distributed Systems](https://pdos.csail.mit.edu/6.824/) 中所介绍的第一篇论文。它提出了一种针对大数据处理的编程模型和实现，使得编程人员无需并行和分布式系统经验就可以轻松构建大数据处理应用。该模型将大数据处理问题拆解为两步，即 `Map` 和 `Reduce`，`Map` 阶段将一组输入的键值对转化为中间结果键值对，`Reduce` 阶段对中间结果键值对按照相同的键进行值的合并，从而得到最终的结果。

## 背景
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

最后，`MapReduce` 框架将上述聚合后的数据分发给 `reduce` 函数执行，即：

```
reduce("it", ["1", "1"])
reduce("was", ["1", "1"])
reduce("the", ["1", "1"])
reduce("best", ["1"])
reduce("worst", ["1"])
reduce("of", ["1", "1"])
reduce("times", ["1", "1"])
```

`reduce` 函数执行时会遍历 `values`，将每个字符串转换为整型后累加，然后作为 `reduce` 的结果返回，最终得到所有单词出现的次数：

```
it 2
was 2
the 2
best 1
worst 1
of 2
times 2
```

实际执行 `reduce` 函数时，并不会将 `values` 一次性传给某个 `reduce` 函数，因为有可能数据量太大无法完全载入内存，所以 `values` 在实现时是个迭代器，`reduce` 函数能以流式的形式获取值。

另外，虽然在上述的例子中 `map` 和 `reduce` 处理的都是字符串类型的数据，但也可以支持其他类型的数据，`map` 和 `reduce` 处理的数据类型遵循如下的模式：

```
map (k1, v1) -> list(k2, v2)
reduce (k2, list(v2)) -> list(v2)
```

可以看到，`map` 产生的中间结果的数据类型和最终结果的数据类型是一致的。对整个框架来说，最初的输入和最终的输出都是某种形式的字节流或字符串，因此在 `Google` 的 `C++` 实现中，提供了专门的数据转换接口，用户可实现该接口用于字符串和 `map`、`reduce` 需要的数据类型之间转换。

## 实现
`MapReduce` 的具体实现视硬件环境的不同而不同，论文中描述的实现是针对 `Google` 内部广泛使用的硬件环境，即通过交换以太网相连的大量廉价 `PC` 组成的集群：

1. 每台机器的配置一般为双核 `x86` 处理器，2-4 `GB` 内存，运行 `Linux` 系统
2. 使用廉价网络硬件，带宽一般为 `100 Mbit/s` 或 `1 Gbit/s`，不过平均来说会小于 `bisection bandwidth`（`bisection bandwidth` 指当某个网络被分成两部分时，这两部分间的带宽）
3. 一个集群一般由几百上千台机器组成，所以机器异常是家常便饭
4. 存储使用的是廉价的 `IDE` 硬盘，并直接装载到了机器上。不过 `Google` 内部实现了一套分布式文件存储系统来管理这些硬盘上的数据，并通过数据冗余作为在不可靠的硬件上实现可用性和可靠性的手段。
5. 用户向调度系统提交一组任务，每个任务包含多个子任务，调度系统会为每个任务分配一批集群内的机器执行。

### 执行概览
在 `Map` 执行阶段，框架会自动将输入数据分为 `M` 片，从而将 `Map` 任务分发到多台机器上并行执行，每台机器只处理某一片的数据。同样的，在 `Reduce` 阶段，框架首先将中间结果数据根据分片函数（例如 `hash(key) mod R`）拆分为 `R` 片，然后继续将 `Reduce` 任务分发执行，用户可自行指定 `R` 的值和实现具体的分片函数。

下图展示了 `Google` 所实现的 `MapReduce` 框架的整体执行流程：

![alt](/images/map-reduce.png)

当用户提交 `MapReduce` 任务后，框架会执行以下一系列流程（下文中的序号和上图中的序号对应）：

1. 首先 `MapReduce` 框架将输入数据分为 `M` 片，每片数据大小一般为 `16 MB` 至 `64 MB`（具体大小可由用户入参控制），然后将 `MapReduce` 程序复制到集群中的一批机器上运行。
2. 在所有的程序拷贝中，某台机器上的程序会成为主节点（`Master`），其余称为工作节点（`Worker`），由主节点向工作节点分派任务，一共有 `M` 个 `Map` 任务和 `R` 个 `Reduce` 任务需要分派。主节点会选择空闲的工作节点分派 `Map` 或 `Reduce` 任务。
3. 如果某个工作节点被分派了 `Map` 任务则会读取当前的数据分片，然后将输入数据解析为一组键值对后传递给用户自定义的 `Map` 函数执行。`Map` 函数产生的中间结果键值对会暂存在内存中。
4. 暂存在内存中的中间结果键值对会周期性的写入到本地磁盘中，注意前文提到这些磁盘上的数据由 `Google` 内部的分布式文件存储系统所管理，所以当前工作节点产生的中间结果键值对并不是直接写入到当前机器的磁盘上，而是会由某个分片函数将这些数据写入到文件存储系统下的 `R` 个区，这样所有键相同的中间结果数据最终会分发到相同的区中。同时，这些数据写入后的地址会回传给 `Master` 节点，`Master` 节点会将这些数据的地址发送给相应的 `Reduce` 节点。
5. 当 `Reduce` 节点接收到 `Master` 节点发送的中间结果数据地址通知后，将通过 `RPC` 请求根据数据地址读取 `Map` 节点生成的数据。在所有中间结果数据都读取完成后，`Reduce` 节点会先将所有中间结果数据按照键进行排序，这样所有键相同的数据就聚合在了一起。之所以要排序是因为一个 `Reduce` 节点会分发处理多个键下的中间结果数据。如果中间结果数据量太大不足以完全载入内存，则需要使用外部排序。
6. `Reduce` 节点执行时会先遍历排序后的中间结果数据，每遇到一个新的键就会将该键及其对应的所有中间结果数据传递给用户自定义的 `Reduce` 函数执行。`Reduce` 函数执行的结果数据会追加到当前 `Reduce` 节点的最终输出文件里。
7. 当所有 `Map` 任务和 `Reduce` 任务都执行完成后，`Master` 节点会唤醒用户程序，并将控制权交还给用户代码。

当成功结束 `MapReduce` 任务后，其执行结果就保存在了 `R` 个文件中（每个文件对应一个 `Reduce` 节点的产出，文件的名字由用户所指定）。一般来说，用户不必将这 `R` 个输出文件合并成一个，它们通常会作为另一个 `MapReduce` 任务的输入，或交由其他分布式应用处理。

### Master 节点数据结构
```
// 任务状态，空闲、进行中、完成
enum TaskState {
    Idle,
    InProgress,
    Completed
}

class MapTask {
    TaskState state;
}

class ReduceTask {
    TaskState state;
}

class Worker {
    int id;
}

class Master {

}
```

参考：

- [MapReduce: Simplified Data Processing on Large Clusters](https://research.google/pubs/pub62/)
- [Bisection bandwidth](https://en.wikipedia.org/wiki/Bisection_bandwidth)
- [Understanding bisection bandwidth](https://networkengineering.stackexchange.com/questions/28894/understanding-bisection-bandwidth)
