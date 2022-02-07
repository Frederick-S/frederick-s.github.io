title: 二叉搜索树插入和删除
tags:
- Data Structure
- Algorithm
- Binary Search Tree
---

## 插入节点
往一个二叉搜索树中插入一个节点后的结果并不唯一，例如对于下面的二叉搜索树：

![alt](/images/bst-bst-1.png)

如果要插入节点2，可以将2作为3的左子节点：

![alt](/images/bst-bst-2.png)

或者将2作为1的右子节点：

![alt](/images/bst-bst-3.png)

对于第一种方法，类似于往单链表的中间插入节点，既要更新前继节点的 `next` 指针，又要将新的节点的 `next` 指针指向下一个节点；而对于第二种方法，只需要将新节点挂载到目标节点的左子节点或右子节点即可，实现上较为简洁。

### 非递归
```py
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right


class Solution:
    def insertIntoBST(self, root: TreeNode, val: int) -> TreeNode:
        if not root:
            return TreeNode(val)

        prev, current = None, root

        while current:
            prev = current

            if current.val < val:
                current = current.right
            else:
                current = current.left

        if prev.val > val:
            prev.left = TreeNode(val)
        else:
            prev.right = TreeNode(val)

        return root
```

### 递归
```py
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right


class Solution:
    def insertIntoBST(self, root: TreeNode, val: int) -> TreeNode:
        if not root:
            return TreeNode(val)

        if root.val < val:
            root.right = self.insertIntoBST(root.right, val)
        else:
            root.left = self.insertIntoBST(root.left, val)

        return root
```

## 删除节点
