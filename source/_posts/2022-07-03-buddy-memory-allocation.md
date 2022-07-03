title: 'Buddy Memory Allocation'
tags:
- Memory Allocation
---

## 介绍
`Buddy Memory Allocation` 是内存分配算法的一种，它假定内存的大小为$2^N$（`N` 为整数），并且总是以2的幂次方为单位分配或者释放内存。

## 算法
假设某个线程需要申请 `m` 字节内存，`Buddy Memory Allocation` 会先在当前所有的空闲空间中找到最小的空间满足$2^k \geq m$，如果$2^k$的一半依然大于等于 `m`，说明当前分配的空间过大，则继续将$2^k$对半分（分裂后的这两块内存区域就成为了互为兄弟关系（`buddies`）），不断重复上述操作，直到找到最小的 `p`（$p \leq k$）满足$2^{p - 1} < m \leq 2^p$。

下图描述了从16字节中分配3字节的过程（假设系统总共只有16字节内存）：

1. 初始状态整个内存只有16字节，是可分配的最小空间；不过由于16字节的一半大于3字节，所以将16字节拆分为两个8字节
2. 同理一个8字节的一半依然大于3字节，继续将其中一个8字节拆分为两个4字节
3. 4字节的一半比3字节小，所以4字节就是可分配的最小内存空间

![alt](/images/buddy-1.png)

当某个线程需要释放$2^k$的内存时，`Buddy Memory Allocation` 会尝试将这个内存空间及其相邻的兄弟空间一起合并得到一个$2^{k + 1}$大小的空间，然后一直重复此操作，直到某块内存空间无法和其兄弟空间合并，无法合并的情况有三种：

1. 当前分配的内存空间大小为整个内存空间的大小，所以也就没有兄弟空间
2. 兄弟空间已全部分配
3. 兄弟空间已局部分配

下图描述了从16字节中释放3字节的过程（假设系统总共只有16字节内存）：

1. 当前系统分配了一个2字节的空间和一个4字节的空间
2. 此时需要回收被占用的2字节，由于它的兄弟空间没有被占用，所以两个2字节的空间合并为一个4字节的空间
3. 合并后的4字节的空间的兄弟空间同样没有被占用，两个4字节的空间继续合并为1个8字节的空间
4. 合并后的8字节的空间的兄弟空间存在部分占用，无法继续合并

![alt](/images/buddy-2.png)

## 实现
### 内存
首先定义一个 `Memory` 类来表示内存，其内部使用一个 `byte` 数组来存储数据，数组的索引就是内存地址：

```java
public class Memory {
    private final byte[] memory;

    public Memory(int size) {
        if (size <= 0) {
            throw new IllegalArgumentException("size should be greater than zero");
        }

        this.memory = new byte[size];
    }
}
```

同时，`Memory` 类还支持 `bool` 和 `int32` 类型的数据读写，从实现的简化考虑，`bool` 值的读写以一个 `byte` 为单位；而 `int32` 的读写以4个 `byte` 为单位：

```java
// 在给定的地址设置布尔值，占据一字节
public void setBool(int address, boolean value) {
    checkAddress(address);

    this.memory[address] = value ? (byte) 1 : (byte) 0;
}

// 根据给定的地址读取布尔值，读取一字节
public boolean getBool(int address) {
    checkAddress(address);

    return this.memory[address] == (byte) 1;
}

// 在给定的地址设置 int32，占据4字节
public void setInt32(int address, int value) {
    checkAddress(address);

    byte[] bytes = ByteBuffer.allocate(Constant.INT32_SIZE).putInt(value).array();
    setByteArray(address, bytes);
}

// 根据给定的地址读取 int32，读取4字节
public int getInt32(int address) {
    checkAddress(address);

    if (address + Constant.INT32_SIZE > this.memory.length) {
        throw new IllegalArgumentException("address overflow");
    }

    byte[] bytes = new byte[Constant.INT32_SIZE];

    System.arraycopy(this.memory, address, bytes, 0, Constant.INT32_SIZE);

    return ByteBuffer.wrap(bytes).getInt();
}
```

### Block
定义 `Block` 表示系统所分配的内存块，其中 `address` 表示该 `Block` 的起始内存地址，同时 `Block` 借助 `Memory` 对内存实现读写操作：

```java
public class Block {
    private final int address;
    private final Memory memory;

    public Block(int address, Memory memory) {
        if (address < 0 || address >= memory.getSize()) {
            throw new IllegalArgumentException("invalid address");
        }

        Objects.requireNonNull(memory, "memory should not be null");

        this.address = address;
        this.memory = memory;
    }
}
```

下图展示了一个 `Block` 在内存中的布局：

![alt](/images/buddy-3.png)

一个 `Block` 除了包含用户数据外还需要保存元数据，所以每个 `Block` 占据的内存会大于用户实际申请的内存；元数据中的第一个字节表示当前内存块是否被使用；第2到5字节表示 `sizeClass`，用来计算当前内存块所占据的内存的大小，即$2^{sizeClass}$；第6到9字节表示前一个空闲内存块的地址；第10到13字节表示后一个空闲内存块的地址；从第14字节开始就是用户数据。当然，这只是一种很粗犷的布局方式，实际应用中的布局必然比这个精炼。

这里需要前一个/后一个空闲内存块的地址是因为将相同大小的内存块通过双向链表的方式串联在一起，从而能快速找到以及删除某个指定大小的内存块。因为 `Buddy Memory Allocation` 始终以$2^k$大小分配内存，假设系统的最大内存为$2^N$，则可以建立 `N` 个双向链表，每个双向链表表示当前大小下可用的内存块，如下图所示：

![alt](/images/buddy-4.png)

`Block` 通过 `Memory` 类提供的 `bool`，`int32` 数据的读写功能来实现对元数据的读写：

```java
// 将 block 标记为已使用
public void setUsed() {
    this.memory.setBool(this.address, true);
}

// 当前 block 是否已使用
public boolean isUsed() {
    return this.memory.getBool(this.address);
}

// 释放当前 block
public void setFree() {
    this.memory.setBool(this.address, false);
}

// 设置 sizeClass
public void setSizeClass(int sizeClass) {
    this.memory.setInt32(this.address + Constant.OFFSET_SIZE_CLASS, sizeClass);
}

// 获取当前 block 的 sizeClass
public int getSizeClass() {
    return this.memory.getInt32(this.address + Constant.OFFSET_SIZE_CLASS);
}

// 设置前一个空闲的 block
public void setPrev(Block block) {
    this.memory.setInt32(this.address + Constant.OFFSET_PREV, block.getAddress());
}

// 获取前一个空闲的 block
public Block getPrev() {
    int address = this.memory.getInt32(this.address + Constant.OFFSET_PREV);

    return address == -1 ? null : new Block(address, this.memory);
}

// 设置后一个空闲的 block
public void setNext(Block block) {
    this.memory.setInt32(this.address + Constant.OFFSET_NEXT, block.address);
}

// 获取后一个空闲的 block
public Block getNext() {
    int address = this.memory.getInt32(this.address + Constant.OFFSET_NEXT);

    return address == -1 ? null : new Block(address, this.memory);
}
```

### BlockList
`BlockList` 表示一个双向链表，用于存储某个 `sizeClass` 下的所有空闲内存块，为了实现方便，内部使用了一个哨兵头节点来作为双向链表的头节点，新节点的插入采用头插法的方式：

```java
package buddy;

import java.util.Objects;

public class BlockList {
    private final Block head;
    private final int sizeClass;

    public BlockList(int address, Memory memory, int sizeClass) {
        if (address < 0 || address >= memory.getSize()) {
            throw new IllegalArgumentException("invalid address");
        }

        Objects.requireNonNull(memory, "memory cannot be null");

        if (sizeClass <= 0) {
            throw new IllegalArgumentException("invalid sizeClass");
        }

        this.head = new Block(address, memory);
        this.head.setSizeClass(sizeClass);
        this.sizeClass = sizeClass;
    }

    // 清空列表，将头节点指向自身
    public void clear() {
        this.head.setNext(this.head);
        this.head.setPrev(this.head);
    }

    // 列表是否为空
    public boolean isEmpty() {
        return this.head.getNext().equals(this.head);
    }

    // 获取头节点的后一个节点
    public Block getFirst() {
        if (this.isEmpty()) {
            throw new RuntimeException("list must not be empty");
        }

        return this.head.getNext();
    }

    // 头插法插入一个 block
    public void insertFront(Block block) {
        this.head.insertAfter(block);
    }

    // 当前列表是否有空闲的内存块，以及该内存块是否能容纳 size 大小的数据（减去元数据占用的内存大小后）
    public boolean hasAvailableBlock(int size) {
        return !this.isEmpty() && Block.getActualSize(this.sizeClass) >= size;
    }

    // 所有空闲内存块的数量，不包含哨兵头节点
    public int length() {
        int length = 0;
        Block block = this.head.getNext();

        while (!block.equals(this.head)) {
            length += 1;
            block = block.getNext();
        }

        return length;
    }
}
```

由于需要通过哨兵头节点访问下一个可用的内存块，所以每个哨兵头节点就需要知道下一个 `Block` 的内存起始地址，因此同样需要将哨兵头节点的信息保存在内存中，对于内存大小为$2^N$的系统来说，一共需要保存 `N` 个哨兵头节点的信息，这里将内存分为两部分，前一部分保存所有的哨兵头节点，后一部分保存所有的 `Block`：

![alt](/images/buddy-5.png)

因此第一个 `Block` 的内存起始位置也就等于所有哨兵节点的大小之和。

### 内存管理
#### 初始化
定义 `Allocator` 负责内存的分配和回收，本质上是对 `Block` 的管理，即 `Block` 的分裂和合并：

```java
public class Allocator {
    private final Memory memory;
    private final BlockList[] blockLists;

    private static final int MIN_SIZE_CLASS = 4;
    private static final int MAX_SIZE_CLASS = 16;

    public Allocator() {
        int allHeadSentinelSize = this.getMemoryOffset();
        int maxMemorySize = (1 << MAX_SIZE_CLASS) + allHeadSentinelSize;
        this.memory = new Memory(maxMemorySize);
        this.blockLists = new BlockList[MAX_SIZE_CLASS];

        // 初始化空闲列表
        for (int i = 0; i < MAX_SIZE_CLASS; i++) {
            int sizeClass = i + 1;
            int headSentinelAddress = Constant.HEAD_SENTINEL_SIZE * i;
            this.blockLists[i] = new BlockList(headSentinelAddress, this.memory, sizeClass);
            this.blockLists[i].clear();
        }

        // The single full block
        Block block = new Block(allHeadSentinelSize, this.memory);
        block.setSizeClass(MAX_SIZE_CLASS);
        block.setFree();
        this.blockLists[MAX_SIZE_CLASS - 1].insertFront(block);
    }
}
```

在这个例子中，我们假设系统最大能支持的内存大小为$2^{16}$个字节，由于哨兵节点也需要占用一部分内存，所以在构造函数中初始化 `Memory` 的大小为所有哨兵节点占用的内存大小加上 $2^{16}$ 个字节。同时，系统可分配的 `Block` 的大小分别为$2^1$，$2^2$，...，$2^{15}$，$2^{16}$，对应需要初始化16个双向链表，这里简单的使用数组来保存这16个双向链表，并初始化对应哨兵头节点的内存起始地址。同时，整个系统在初始状态只有一个 `Block`，大小为$2^{16}$。

#### 内存分配
如前面所述，内存分配的第一步是找到满足用户内存需求的最小的 `Block`，然后如果 `Block` 过大则继续将 `Block` 进行分裂：

```java
public int alloc(int size) {
    Block block = null;

    for (int i = 0; i < MAX_SIZE_CLASS; i++) {
        BlockList blockList = this.blockLists[i];

        // 找到满足用户内存需求的最小的 Block
        if (!blockList.hasAvailableBlock(size)) {
            continue;
        }

        block = blockList.getFirst();

        // 尝试将 block 分裂
        block = this.split(block, size);

        // 将 block 标记为已使用
        block.setUsed();

        break;
    }

    if (block == null) {
        throw new RuntimeException("memory is full");
    }

    // 这里没有返回 block 的起始地址，因为 block 的起始地址指向的是元数据，实际需要返回用户数据的起始地址
    return block.getUserAddress();
}

// 将 Block 分裂（如果能分裂的话），返回分裂后的左兄弟
private Block split(Block block, int size) {
    int sizeClass = block.getSizeClass();

    // 只要 block 的一半（减去元数据占据的空间后）仍能容纳 size，则持续将 block 分裂
    // 由于 block 本身需要存储元数据，每个 block 至少需要 2^MIN_SIZE_CLASS 字节
    while (sizeClass > MIN_SIZE_CLASS && Block.getActualSize(sizeClass - 1) >= size) {
        int newSizeClass = sizeClass - 1;

        // 将 block 分裂为两个，取第一个继续分裂
        Block[] buddies = this.splitToBuddies(block, newSizeClass);
        block = buddies[0];
        sizeClass = newSizeClass;
    }

    // 将 block 从空闲链表中删除
    block.removeFromList();

    return block;
}

private Block[] splitToBuddies(Block block, int sizeClass) {
    block.removeFromList();
    Block[] buddies = new Block[2];

    for (int i = 0; i < 2; i++) {
        // 更新分裂后的 block 的起始地址和 sizeClass，并标记为可用
        int address = block.getAddress() + (1 << sizeClass) * i;
        buddies[i] = new Block(address, this.memory);
        buddies[i].setFree();
        buddies[i].setSizeClass(sizeClass);
    }

    // 这里从后往前遍历 buddies 插入到双链表中是因为最后返回给用户的是第一个 buddy
    for (int i = 1; i >= 0; i--) {
        this.blockLists[sizeClass - 1].insertFront(buddies[i]);
    }

    return buddies;
}
```

#### 内存回收
应用程序要求释放内存时，提交的是用户数据的起始地址，需要先将其转为 `Block` 的起始地址（减去 `Block` 元数据的占用空间大小即可），然后尝试将 `Block` 和其兄弟合并，并将合并后的 `Block` 加入到空闲列表中：

```java
public void free(int userAddress) {
    // 根据用户数据地址得到 Block 的起始地址
    Block block = Block.fromUserAddress(userAddress, this.memory);
    block.setFree();

    // 尝试将 block 和其兄弟合并
    this.merge(block);
}

public void merge(Block block) {
    int sizeClass = block.getSizeClass();

    // 最多只能合并到 MAX_SIZE_CLASS - 1
    while (sizeClass < MAX_SIZE_CLASS) {
        // 得到兄弟 block
        Block buddy = this.getBuddy(block, sizeClass);

        // 兄弟 block 正在被使用或者已分裂为更小的 block，则不能合并
        if (buddy.isUsed() || buddy.getSizeClass() != sizeClass) {
            break;
        }

        // 将兄弟 block 从空闲链表中删除
        buddy.removeFromList();

        // 如果兄弟 block 的起始地址比 block 的起始地址小，说明当前的 block 是右兄弟，由于合并后需要得到整个 block 的起始地址，因此将 block 指向 buddy
        if (block.getAddress() > buddy.getAddress()) {
            block = buddy;
        }

        sizeClass += 1;
    }

    // 设置合并后的 sizeClass
    block.setSizeClass(sizeClass);
    this.blockLists[sizeClass - 1].insertFront(block);
}
```

这里有个关键的问题在于如何根据 `block` 的地址知道其兄弟 `block` 的地址？因为一个 `block` 会被分为左兄弟和右兄弟两个内存块，如果当前 `block` 是左兄弟，则右兄弟的地址为 `block.getAddress() + 1 << sizeClass`，如果当前 `block` 是右兄弟，则左兄弟的地址为 `block.getAddress() - 1 << sizeClass`。然而由于缺失位置信息我们并不能知道一个 `block` 是左兄弟还是右兄弟。

原作者在这里巧妙的在不引入额外的元数据的情况下解决了这个问题。首先，对于某个 `sizeClass` 为 `k` 的内存块来说，它的起始地址一定是$C2^k$，其中 `C` 为整数。这里使用数学归纳法来证明，假设系统内存最多支持$2^N$个字节，则初始状态下整个系统只有一个内存块，`k` 就等于 `N`，该内存块的起始地址为0，满足$C2^k$，取 `C = 0` 即可。假设某个 `sizeClass` 为 `k` 的内存块的起始地址满足$C2^k$，则需要进一步证明分裂后的两个内存块的起始地址为$C'2^{k - 1}$。而分裂后的内存块的起始地址分别为$C2^k$和$C2^k + 2^{k - 1}$，又$C2^k = (2C)2^{k - 1}$，$C2^k + 2^{k - 1} = (2C+ 1)2^{k - 1}$，证明完毕。

然后我们就可以通过位运算来计算兄弟内存块的起始地址，对于$2^k$大小的内存块来说，两个兄弟内存块的起始地址分别为$C2^k$和$C2^k + 2^k = (C + 1)2^k$，由于$2^k$的二进制表示为1后面跟着 `k` 个0，所以对于$C2^k$来说，其二进制表示中从低位往高位数的第 `k + 1` 位有可能是1（如 `C = 1`），也有可能是0（如 `C = 2`），不管是哪种情况，$C2^k$再加上$2^k$后的二进制表示中从低位往高位数的第 `k + 1` 位必然和$C2^k$不同，也就是说两个兄弟内存块的地址的最低位不同。因此我们就可以将内存块的地址和 `1 << sizeClass`（也就是$2^k$）进行异或运算，得到的地址就是对应兄弟内存块的地址。

## 总结
以上仅作为 `Buddy Memory Allocation` 算法的示例，不具有实际应用意义，例如完全没有考虑线程安全。完整的代码可参考原作者的 [代码](https://github.com/kunigami/blog-examples/blob/master/buddy-algorithm/buddy_algorithm.py) 及 `Java` 版本的 [buddy-memory-allocation](https://github.com/Frederick-S/buddy-memory-allocation)。

## 参考
* [Buddy Memory Allocation](https://www.kuniga.me/blog/2020/07/31/buddy-memory-allocation.html)
* [Buddy Methods](https://opendsa-server.cs.vt.edu/ODSA/Books/Everything/html/Buddy.html#buddy-methods)