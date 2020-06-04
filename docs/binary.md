# 二叉树的学习
---

二叉树的按照遍历可以分为**前序遍历**，**中序遍历**，**前序后历**。

## 递归遍历

对于三种遍历，如果使用递归时，有个方程巧妙的通用模板:

```Python
def reverseBinaryTree(root: TreeNode):
    if not root: return
    # Preorder
    reverseBinaryTree(root.left)
    # Inorder
    reverseBinaryTree(root.right)
    # Lastorder
```

## 迭代遍历

和递归不同的是，迭代遍历需要借助`栈`或`队列`辅助结构，使用的具体情况为：

* 宽度优先 + 队列
* 深度优先 + 栈

DFS + Stack，以中序遍历为例：

```Python
def dfs(root: TreeNode):
    if not root: return
    res = []
    stack = []
    cur = root
    while stack or cur:
        # 遍历左子树
        while cur:
            stack.append(cur)
            cur = cur.left
        # 左子树遍历完毕
        top = stack.pop()
        # 加入父节点
        res.append(top.val)
        # 遍历右子树
        cur = top.right
```

BFS + Queue 按层遍历树

```Python
def bfs(root: TreeNode):
    if not root: return
    queue = collections.deque()
    queue.append(root)
    res = []
    while queue:
        tmp = []
        for _ in range(len(queue)):
            cur = queue.popleft()
            if not cur: continue
            tmp.append(cur.val)
            queue.append(cur.left)
            queue.append(cur.right)
        if tmp:
            res.append(tmp)
    print(res)
```