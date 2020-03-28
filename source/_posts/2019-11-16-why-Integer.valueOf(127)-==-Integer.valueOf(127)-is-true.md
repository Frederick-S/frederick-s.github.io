title: 为什么 Integer.valueOf(127) == Integer.valueOf(127) 为 True
tags:
- Java
---

`Integer` 类的内部维护了一个 `IntegerCache` 的静态类，默认缓存了-128到127的 `Integer` 对象，而 `java.lang.Integer#valueOf(int)` 方法执行时会判断参数是否在-128到127之间，如果在这个区间，则返回 `IntegerCache` 内部的缓存对象，所以 `Integer.valueOf(127) == Integer.valueOf(127)` 为 `True`。

不过，整型缓存只适用于自动装箱的情况，不适用于通过构造函数创建的 `Integer` 对象间的比较，在下述代码中，最终的输出结果是 `integer1 == integer2` 和 `integer3 != integer4`。

```java
Integer integer1 = 3;
Integer integer2 = 3;

if (integer1 == integer2) {
    System.out.println("integer1 == integer2");
} else {
    System.out.println("integer1 != integer2");
}

Integer integer3 = new Integer(3);
Integer integer4 = new Integer(3);

if (integer3 == integer4) {
    System.out.println("integer3 == integer4");
} else {
    System.out.println("integer3 != integer4");
}
```

`Integer integer1 = 3` 发生了自动装箱，编译器会将其等价转换为 `Integer integer1 = Integer.valueOf(3)`，所以 `integer1 == integer2`。而通过构造函数创建的 `Integer` 对象属于不同的对象，指向不同的内存地址，所以 `integer3 != integer4`。

另外，可以通过增加虚拟机的参数 `-XX:AutoBoxCacheMax=size` 来设置整型缓存的最大值，如 `-XX:AutoBoxCacheMax=500` 表示-128到500的整型会被缓存。

参考：

* [Java Integer Cache — Why Integer.valueOf(127) == Integer.valueOf(127) Is True](https://medium.com/@njnareshjoshi/java-integer-cache-why-integer-valueof-127-integer-valueof-127-is-true-e5076824a3d5)
* [Java Integer Cache](https://javapapers.com/java/java-integer-cache/)