# 堆排序

**堆排序（Heapsort）**就是把最大（小）堆的最大（小）数取出，然后继续将剩余的元素重新构建最大（小）堆，直到剩余数只有一个时结束。

> **最大堆**：最大堆中的最大元素值出现在根结点（堆顶），堆中每个父节点的元素值都大于等于其孩子结点（如果存在）  
> **最小堆**：最小堆中的最小元素值出现在根结点（堆顶），堆中每个父节点的元素值都小于等于其孩子结点（如果存在）

* 平均时间复杂度：$$ \mathcal{O}({n}\log_{2}{n}) $$
* 最好时间复杂度：$$ \mathcal{O}({n}\log_{2}{n}) $$
* 最坏时间复杂度：$$ \mathcal{O}({n}\log_{2}{n}) $$
* 空间复杂度：$$ \mathcal{O}(1) $$，最大为 $$ \mathcal{O}(n) $$
* 稳定性：不稳定
* 排序方式：原地（in-place）

**堆（二叉堆）**是一种近似于**完全二叉树**的结构，这使得堆可以利用数组来表示。

![](http://bubkoo.qiniudn.com/heap-and-array-zero-based.png)

仔细观察能够轻易地出父节点与其孩子节点的下标之间的关系：

* 节点 i 的父节点坐标： $$ Parent(i) = floor(\dfrac{i - 1}{2}) $$ 
* 节点 i 的左节点坐标：$$ Left(i) = 2i + 1 $$ 
* 节点 i 的右节点坐标：$$ Right(i) = 2(i + 1) $$ 

> **完全二叉树**，在不考虑二叉树最后一层的情况下，其余层的节点都是满的。也就是说，具有 n 个节点的完全二叉树的深度为 $$ k = log_{2}n + 1 $$，深度为 k 的完全二叉树至少有 $$ {2}^{k} $$ 个节点，至多有 $$ {2}^{k + 1} $$ 个节点。

一般实现：

```js
/*
 * 数组元素交换函数
 */
function swap(arr, i, j){
    let tmp = arr[i];
    arr[i] = arr[j];
    arr[j] = tmp;
}

function heapSort(arr) {
  buildMaxHeap(arr);
  for (let i = arr.length - 1; i > 0; i--) {
    swap(arr, 0, i);
    maxHeapify(arr, 0, i);
  }  

  /**
   * 构建最大堆
   **/
  function buildMaxHeap(arr) {
    const heapSize = arr.length;
    let iParent = (heapSize - 1) >> 1; // Math.floor((heapSize - 1) / 2);
    for (let i = iParent; i >= 0; --i) {
      maxHeapify(arr, i, heapSize);
    }
  }

  /**
   * 从 index 开始检查并保持最大堆性质
   **/
  function maxHeapify(arr, index, heapSize) {
    let iMax = index,
        iLeft = 2 * index + 1,
        iRight = 2 * (index + 1);
    if (iLeft < heapSize && arr[index] < arr[iLeft]) {
      iMax = iLeft;
    }
    if (iRight < heapSize && arr[iMax] < arr[iRight]) {
      iMax = iRight;
    }
    if (iMax != index) {
      swap(arr, iMax, index);
      maxHeapify(arr, iMax, heapSize); // 递归调整
    }
  }
  return arr;
}
```



