title: Python for/while else
tags:
- Python
---

和常见的语言不同，`Python` 的 `for/while` 可以配合 `else` 使用。简单来说，当 `for/while` 循环体中没有执行 `break` 时，就会执行 `else` 中的代码。假设需要判断数组中是否存在某个数，如果不存在的话则抛出异常，一种可能的写法是：

```py
target = 10
found = False

for i in range(5):
    if i == target:
        found = True

        break

if not found:
    raise Exception('not found')
```

借助 `for/while else` 可改写成：

```py
target = 10

for i in range(5):
    if i == target:
        break
else:
    raise Exception('not found')
```

虽然代码少了几行，但是对于不熟悉该语法特性的人来说无法一眼看穿代码的意图。

参考：

- [Why does python use 'else' after for and while loops?](https://stackoverflow.com/questions/9979970/why-does-python-use-else-after-for-and-while-loops)
