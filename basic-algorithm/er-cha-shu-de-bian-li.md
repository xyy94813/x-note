#

二叉树的遍历

> 二叉树（Binary Tree）每个节点最多有两个子节点的树型数据结构。

```js
class TreeNode {
  constructor(val, left = null, right = null) {
    this.val = val;
    this.left = left;
    this.right = right;
  }
}
```

二叉树的遍历方式主要分为 **深度优先（Deep-first Search, DFS）** 和 **广度优先（Breadth-first Search，BFS）**。

其中深度优先遍历有以下几种方法

1. **前序遍历**
2. **中序遍历**
3. **后序遍历**

## 深度优先

深度优先顾名思义就是先纵向搜索，在进入下一个兄弟节点之前，尽可能的向下搜索每一个子节点。
该算法从根节点开始（在图的情况下选择一些任意节点作为根节点）并在回溯之前尽可能地沿着每个分支进行探索。

### 前序遍历

前序遍历（pre-order）的访问顺序是 DLR

> L 为左子树，R 为右子树，D 为根节点

![](https://user-gold-cdn.xitu.io/2018/6/20/1641c591b79998b9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

上图的前序遍历顺序为: ABDGHCEIF

#### 前序遍历的递归方式

```js
function preOrder(root) {
  if (!root) {
    return;
  }
  // do something
  // console.log(root.val)
  preOrder(root.left);
  preOrder(root.right);
}
```

#### 前序遍历的非递归方式

对于任一结点 `P`：

1. 访问结点 `P`，并将结点 `P` 入栈;

2. 判断结点 `P` 的左孩子是否为空，若为空，则取栈顶结点并进行出栈操作，并将栈顶结点的右孩子置为当前的结点 `P`，循环至 1; 若不为空，则将 `P` 的左孩子置为当前的结点 `P`;

3. 直到 `P` 为 `null` 并且栈为空，则遍历结束。

```js
function preOrderWithoutRecursion(root) {
  const stack = [];
  let node = root;
  while (node || stack.length > 0) {
    if (node) {
      // do something
      // console.log(node.val);
      stack.push(node);
      node = node.left;
    } else {
      node = stack.pop().right;
    }
  }
}
```

### 中序遍历

中序遍历（in-order）的顺序是 LDR

![](https://user-gold-cdn.xitu.io/2018/6/20/1641c591b78015a9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

上图的中序遍历顺序为: GDHBAEICF

#### 中序遍历的递归方式

```js
function inOrder(root) {
  if (!root) {
    return;
  }
  inOrder(root.left);
  // do something
  // console.log(root.val);
  inOrder(root.right);
}
```

#### 中序遍历的非递归方式

对于任一结点 `P`：

1. 若其左孩子不为空，则将 `P` 入栈并将 `P` 的左孩子置为当前的 `P`，然后对当前结点 `P` 再进行相同的处理；
2. 若其左孩子为空，则取栈顶元素并进行出栈操作，访问该栈顶结点，然后将当前的 `P` 置为栈顶结点的右孩子；
3. 直到 `P` 为 `null` 并且栈为空则遍历结束

```js
function inOrderWithoutRecursion(root) {
  const stack = [];
  let node = root;
  while (node || stack.length > 0) {
    if (node) {
      stack.push(node);
      node = node.left;
    } else {
      const lastNode = stack.pop();
      // do something
      // console.log(lastNode.val);
      node = lastNode.right;
    }
  }
}
```

### 后序遍历

后序遍历（post order）的顺序是 LRD

![](https://user-gold-cdn.xitu.io/2018/6/20/1641c591b791e58b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

上图的后序遍历顺序为: GHDBIEFCA

#### 后序遍历的递归方式

```js
function postOrder(root) {
  if (!root) {
    return;
  }

  postOrder(root.left);
  postOrder(root.right);
  // do something
  // console.log(root.val);
}
```

#### 后序遍历的非递归方式

```js
function postOrderWithoutRecursion(root) {
  if (!root) {
    return;
  }
  const stack = [];
  let node = root;
  let last = node;
  stack.push(node);
  while (stack.length > 0) {
    node = stack[stack.length - 1];
    if (
      (!node.left && !node.right) ||
      (!node.right && last === node.left) ||
      last === node.right
    ) {
      // doing something
      // console.log(node.val);
      last = node;
      stack.pop();
    } else {
      if (node.right) {
        stack.push(node.right);
      }
      if (node.left) {
        stack.push(node.left);
      }
    }
  }
}
```

## 广度优先遍历

广度优先遍历，也可以称之为层级遍历，先访问兄弟节点，再访问子节点。

![](https://upload.wikimedia.org/wikipedia/commons/thumb/d/d1/Sorted_binary_tree_breadth-first_traversal.svg/220px-Sorted_binary_tree_breadth-first_traversal.svg.png)

上图的后序遍历顺序为: FBGADICEH 

```js
function bfs(root) {
  if (!root) {
    return;
  }
  const queue = [root];

  while (queue.length > 0) {
    const node = queue.shift();
    // do something
    // console.log(node.val)
    if (node.left) {
      queue.push(node.left);
    }
    if (node.right) {
      queue.push(node.right);
    }
  }
}
```
