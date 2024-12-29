title: 为什么 Python 的负整数除法结果和 C 不同
tags:
- Python
---

对于整数-5除以2，在 `Python` 中的结果是-3，但是在 `C` 中是-2。如果扩展到其他几种常见的语言，可以看到和 `C` 一致的比较多：

|语言   |结果   |
|---|---|
|C   |-2   |
|C++   |-2   |
|Java   |-2   |
|C#   |-2   |
|Rust   |-2   |
|Go   |-2   |
|Python   |-3   |
|Ruby   |-3   |

区别在于对于结果-2.5是选择向0取整还是向负无穷取整，`Python` 和 `Ruby` 选择了后者。

对于整数 `a` 和 `n`，记 `a` 除以 `n` 的结果是 `q`，余数是 `r`，则有：`a = n * q + r`，其中 `|r| < |n|`。在数论中，`r` 始终是正数，但是不同的编程语言各自有不同的实现。

> In number theory, the positive remainder is always chosen, but in computing, programming languages choose depending on the language and the signs of a or n.

## 编程语言实现
### Truncated division
很多语言采用这种实现，约定 $q = trunc(\frac{a}{n})$，其中 `trunc` 表示向0取整，代表语言如 `Java`。

### Floored division
`Donald Knuth` 提倡这种实现，约定 $q = \lfloor \frac{a}{n} \rfloor$，即向下取整，代表语言如 `Python`。

### Euclidean division
`Raymond T. Boute` 则提倡这种实现，约定：
$$
q = sgn(n)\lfloor \frac{a}{|n|} \rfloor =     
    \begin{cases}
      \lfloor \frac{a}{n} \rfloor & \text{if n > 0}\\
      \lceil \frac{a}{n} \rceil & \text{if n < 0}
    \end{cases}
$$

即根据 `n` 的正负号来判断是向下取整还是向上取整，代表语言如 `ABAP`。

### Rounded division
这是 `Common Lisp` 和 `IEEE 754` 采用的实现，约定 $q = round(\frac{a}{n})$，其中 `round` 使用 `rounding half to even`，即对于1.5，2.5，x.5这样的数字取整到最近的偶数，例如6.5取整到6，7.5取整到8。

### Ceiling division
这是 `Common Lisp` 提供的另一种实现，约定 $q = \lceil \frac{a}{n} \rceil$，即向上取整。

## Python
回到 `Python`，很难说上述哪种实现一定最优，`Python` 的作者提到采用 `floored division` 是因为对于某些应用来说，如果取模运算返回负数没有意义。例如，给定一个 `POSIX timestamp`，如何返回该天的时间部分，即时分秒？因为一天有86400秒，假设时间戳是 `t`，那么 `t % 86400` 就表示该天过了多少秒，就可以进一步转化为时分秒。而对于在 `1970-01-01T00:00:00Z` 之前的日期，`t` 则是负数，采用 `floored division` 的情况下 `t % 86400` 依然返回正数，并且结果也是正确的，而 `truncated division` 则返回负数，需要应用程序进一步处理。

不过，一种编程语言中不一定只提供一种实现，其他实现可以借助函数库。例如，`Python` 中 `-5 % 2` 结果是1，实现方式为 `floored division`，但是 `math.fmod(-5, 2)` 结果是-1，实现方式为 `truncated division`。

## 参考
* [Why Python's Integer Division Floors](https://python-history.blogspot.com/2010/08/why-pythons-integer-division-floors.html)
* [Modulo](https://en.wikipedia.org/wiki/Modulo)
* [WHAT IS NEGATIVE NUMBER REPRESENTATION ?](https://www.computersciencecafe.com/14-negative-number-representation.html)
