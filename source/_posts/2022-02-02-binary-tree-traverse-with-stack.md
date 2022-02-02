title: 二叉树非递归遍历
tags:
- Data Structure
- Algorithm
---

二叉树的遍历直观的解法是使用递归求解，不过同样也可使用非递归方式求解。

## 前序遍历
先来看前序遍历的递归求解：

```py
from typing import List


# Definition for a binary tree node.
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right


class Solution:
    def preorderTraversal(self, root: TreeNode) -> List[int]:
        values = []

        self._preorder_traversal(root, values)

        return values

    def _preorder_traversal(self, root: TreeNode, values: List[int]) -> None:
        if not root:
            return

        values.append(root.val)

        self._preorder_traversal(root.left, values)
        self._preorder_traversal(root.right, values)
```

对于如下的二叉树：

![alt](/images/binary-tree.jpg)

其调用链为：

```
_preorder_traversal(1)
    _preorder_traversal(2)
        _preorder_traversal(4)
        _preorder_traversal(5)
    _preorder_traversal(3)
```

可以看到越深的节点对应的函数调用越先返回，对应先进后出的模型，即栈，所以递归转非递归可借助栈实现。

由于前序遍历是先访问根节点，所以对于每个子树，可以先将根节点入栈，然后依次弹出栈顶的节点，从而实现先访问根节点，然后将左右子树的根节点入栈，由于左子树需要先于右子树被访问，所以右子树的根节点要先入栈，然后再入栈左子树的根节点：

```py
from typing import List


# Definition for a binary tree node.
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right


class Solution:
    def preorderTraversal(self, root: TreeNode) -> List[int]:
        if not root:
            return []

        values = []
        stack = [root]

        while stack:
            current = stack.pop()
            values.append(current.val)

            if current.right:
                stack.append(current.right)

            if current.left:
                stack.append(current.left)

        return values
```

## 中序遍历
在递归的方案下，前序遍历改为中序遍历只需改变 `values.append(root.val)` 的执行位置即可，而在非递归方案下，并不能通过直接改变 `values.append(current.val)` 的执行位置来实现，因为不管放到哪个位置，都会提前访问到根节点。

中序遍历下，最左下方的节点是最先被访问的，沿着左子树的根节点这条线，等同于一个单链表的倒序访问，单链表的倒序如果用栈来实现则是将单链表的所有节点从链表头开始遍历依次放入栈，然后再依次出栈，类似的，只要当前节点存在左子树，则持续将左子树的根节点压入栈，这样下次出栈时，就会先访问最左下方的节点。当某个节点出栈时，由于上述的操作，它必然是某个子树的最左下方的节点，此时需要转到该节点的右子树重复上述流程从而访问右子树的全部节点：

```py
from typing import List


class TreeNode:
    def __init__(self, x):
        self.val = x
        self.left = None
        self.right = None


class Solution:
    def inorderTraversal(self, root: TreeNode) -> List[int]:
        values = []
        stack = []
        current = root

        while current or stack:
            while current:
                stack.append(current)
                current = current.left

            current = stack.pop()
            values.append(current.val)
            current = current.right

        return values
```

## 通用模板
a

参考：

- [[Java] This simple template can be used for 3 traversals](https://leetcode.com/problems/binary-tree-preorder-traversal/discuss/1736072/Java-This-simple-template-can-be-used-for-3-traversals)
- [Preorder, Inorder, and Postorder Iteratively Summarization](https://leetcode.com/problems/binary-tree-postorder-traversal/discuss/45551/Preorder-Inorder-and-Postorder-Iteratively-Summarization)
