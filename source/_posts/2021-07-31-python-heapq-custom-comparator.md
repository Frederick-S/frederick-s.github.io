title: Python heapq 自定义比较器
tags:
- Python
---

使用 `Python` 的 `heapq` 模块时，如果处理的是较为复杂的数据结构，则需要实现自定义比较器来比较两个元素的大小。

## 使用元组
如果 `heapq` 中放入的是元组，那么元组的第一个元素会用于大小比较。假设有这样一个问题，给定一个数组，返回前 `k` 小的数字所在数组中的位置。`Top k` 的问题的一个解法是使用堆，但是这里要求的是数字在数组中的位置而不是数字本身，所以不能直接将数组堆化，可以先将数组中的每个数字转换成一个包含2个元素的元组，元组的第一个元素是数字本身，第二个元素则是数字在数组中的位置。

```py
import heapq

def top_k(numbers, k):
  heap = [(n, i) for i, n in enumerate(numbers)]
  heapq.heapify(heap)

  return list(map(lambda x: heapq.heappop(heap)[1], range(k)))

if __name__ == '__main__':
  print(top_k([5, 4, 3, 2, 1], 3)) # [4, 3, 2]
```

参考：

- [heapq with custom compare predicate](https://stackoverflow.com/questions/8875706/heapq-with-custom-compare-predicate/8875823)
