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

对于第一种方法，类似于往单链表的中间插入节点，既要更新前继节点的 `next` 指针，又要将新的节点的 `next` 指针指向下一个节点；而对于第二种方法，只需要将新节点挂载到目标节点的左子节点或右子节点即可，实现上较为简洁，可分为非递归和递归两种解法。

### 非递归
整体算法分为两步：

1. 找到要挂载的叶子节点
2. 将新节点挂载到该叶子节点的左子节点或右子节点上

第一步等同于二叉搜索树的查找，从根节点开始，将目标值和当前节点的值进行比较，如果当前节点的值比目标值小，说明要找的节点在右子树中，移动到右子节点中查找；如果当前节点的值比目标值大，说明要找的节点在左子树中，移动到左子节点中查找。

找到目标叶子节点后，比较该叶子节点的值和目标值的大小，来决定新节点是作为左子节点还是右子节点插入。

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
def dfs(root):
    if not root:
        # 处理基础情况
    
    if some condition:
        dfs(root.left)
    else:
        dfs(root.right)
    
    return root
```

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
