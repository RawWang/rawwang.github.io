# 迭代和递归
---

对于数据结构而言，最基本的操作就是**遍历**和**修改**，针对基本操作的方法又分为**线性**和**非线性**。

* 线性，就是以`迭代`的形式操作，比如用`for`,`while`循环操作。
* 非线性，对应的就是`递归`

> 链表遍历 -> 二叉树遍历 -> N叉树遍历 -> 图遍历

### 单链表-迭代

```Python
def traverseLinkedList(head: ListNode):
    while head:
        head = head.next
        # Do Something
```
### 单链表-递归

```Python
def reverseLinkedList(head: ListNode):
    if not head: return
    # Do something
    reverse(head.next)
```

单链表可以理解成只有一个孩子节点的树，因此二叉树和N叉树的遍历，可以总结为：

### 二叉树递归遍历

```Python
def reverseBinaryTree(root: TreeNode):
    if not root: return
    # Do something
    reverseBinaryTree(root.left)
    reverseBinaryTree(root.right)
```

### N叉树递归遍历
```Python
def reverseTree(root: TreeNode):
    if not root: return
    for child in root.children:
        reverseTree(child)
```

而图可以理解成多个**N叉树**结合体。

