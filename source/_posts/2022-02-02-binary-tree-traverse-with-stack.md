title: 二叉树非递归遍历
tags:
- Data Structure
- Algorithm
- Binary Tree
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

虽然前序遍历的非递归方案不适用于中序遍历，不过中序遍历的递归方案可略微修改适用于前序遍历，只需将 `values.append(current.val)` 放在不断入栈左子树的循环中即可：

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
        stack = []
        current = root

        while current or stack:
            while current:
                values.append(current.val)
                stack.append(current)
                current = current.left

            current = stack.pop()
            current = current.right

        return values
```

## 后序遍历
后序遍历和中序遍历相同，最先访问的都是最左下方的节点，所以对左子树不断入栈这段逻辑不变，不同的是当出栈时，当前出栈的节点有可能存在右子树，而右子树还还没有被访问，所以当前节点还不能出栈。因此，需要先判断栈顶的节点是否存在右子树，以及右子树是否被访问过，如果存在右子树且未被访问则转向右子树重复上述流程，否则可弹出栈顶节点。而判断栈顶的右子树是否被访问可通过比较栈顶的右子树和上一个被访问的节点来实现，如果两者相等，说明栈顶的右子树刚被访问过，否则未被访问过：

```py
from typing import List


# Definition for a binary tree node.
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right


class Solution:
    def postorderTraversal(self, root: TreeNode) -> List[int]:
        prev = None
        current = root
        stack = []
        values = []

        while current or stack:
            while current:
                stack.append(current)
                current = current.left

            top = stack[-1]

            if top.right and prev != top.right:
                current = top.right
            else:
                current = stack.pop()
                values.append(current.val)
                prev = current
                current = None

        return values
```

## 通用模板
上述各非递归方案各不相同，是否存在和递归方案类似的通用模板方案？[这里](https://leetcode.com/problems/binary-tree-preorder-traversal/discuss/1736072/Java-This-simple-template-can-be-used-for-3-traversals) 给出了一种通用方案，首先需要额外引入一个数据结构来标记节点是否被访问过：

```java
private class Pair {
    boolean visited;
    TreeNode node;

    Pair(TreeNode node, boolean visited) {
        this.node = node;
        this.visited = visited;
    }
}
```

在 `Python` 中，可简单通过元组来实现，对应模板代码为：

```py
from typing import List


# Definition for a binary tree node.
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right


class Solution:
    def xxxTraversal(self, root: TreeNode) -> List[int]:
        if not root:
            return []
        
        stack = [(root, False)]
        values = []
        
        while stack:
            current, visited = stack.pop()
            
            if visited:
                values.append(current.val)
            else:
                # 在这里处理左子树，右子树，根节点的入栈顺序
                pass
        
        return values
```

对于三种遍历方式，上述模板方法仅在处理左子树，右子树，根节点的入栈顺序上不同，实际入栈顺序和遍历顺序相反：

```py
# 前序遍历
if current.right:
    stack.append((current.right, False))

if current.left:
    stack.append((current.left, False))

stack.append((current, True))

# 中序遍历
if current.right:
    stack.append((current.right, False))

stack.append((current, True))

if current.left:
    stack.append((current.left, False))

# 后序遍历
stack.append((current, True))

if current.right:
    stack.append((current.right, False))

if current.left:
    stack.append((current.left, False))
```

从出栈的角度来说，上述方法和理论遍历顺序并不一致，每个节点会入栈两次，第二次入栈时才会设置 `visited` 为 `True`，但从 `visited` 的角度来说顺序是和理论遍历顺序一致的。

## 参考

- [[Java] This simple template can be used for 3 traversals](https://leetcode.com/problems/binary-tree-preorder-traversal/discuss/1736072/Java-This-simple-template-can-be-used-for-3-traversals)
- [Preorder, Inorder, and Postorder Iteratively Summarization](https://leetcode.com/problems/binary-tree-postorder-traversal/discuss/45551/Preorder-Inorder-and-Postorder-Iteratively-Summarization)