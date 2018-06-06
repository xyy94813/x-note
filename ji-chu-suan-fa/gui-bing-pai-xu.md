# 归并排序

**归并排序（Merge Sort）**是一种基于归并操作的排序算法，核心思想是将两个已经排序的序列合并成一个序列。

* 平均时间复杂度：$$O({n}\log_{2}{n})$$
* 最好时间复杂度：$$O(n)$$
* 最坏时间复杂度：$$O({n}\log_{2}{n})$$
* 空间复杂度：$$O(n)$$
* 稳定性：稳定
* 排序方式：非原地（Not-in-place）

![](https://upload.wikimedia.org/wikipedia/commons/c/cc/Merge-sort-example-300px.gif)

归并算法又分为**自顶下下（Top-down）**和**自底向上（Bottom-up）**两种实现方式

## 自顶向下实现

1. 申请一大小为俩个已排序序列之和的空间
2. 始比较两个序列中的第一个元素，相对较小的元素放入之前申请的空间
3. 取出较小元素所在队列的下一个元素，然后与另一队列之前取出的元素进行比较
4. 重复步骤 2~3 直到某个队列提前遍历完成
5. 将另一序列的所有元素直接放入申请的空间尾部

![](http://youngjd.com/images/2017-02-20-2.png)

具体实现：

```js
// 以下 JavaScript 版本的实现利用了基础库 Array 的特有 API
// 并未考虑达到到最优的空间和时间，仅为了体现核心思想
function mergeSortTopDown(arr) {
    function core(arr1, arr2) {
        const rest = [];
        while (arr1.length && arr2.length) {
            rest.push(arr1[0] <= arr2[0] ? arr1.shift() : arr2.shift());
        }
        return rest.concat(arr1.concat(arr2));
    }
    const len = arr.length;
    if (len < 2) {
        return arr;
    }
    let mid = parseInt(len / 2);
    return core(mergeSortTopDown(arr.slice(0, mid)), mergeSortTopDown(arr.slice(mid)));
}
```

## 自底向上实现

自底向上就是把序列当作是由 n 个长度为 1 的子序列组成的序列，然后以 2 个序列为单元循环来回地合并子序列

1. 将长度为 n 的序列拆分成 n 个子序列
2. 每两个相邻的子序列进行归并操作，n 个子序列变成了 $$ceil(\dfrac{n}{2})$$ 个新的子序列
3. 对所有新的子序列，重复步骤 2，知道最终得到一个长度为 n 的子序列

![](http://youngjd.com/images/2017-02-20-3.png)

具体实现：

```js
function mergeSort(arr) {
  const len = arr.length;
  if(arr.length < 2){
    return;
  }
  
  for (let step = 1; step < len; step *= 2) {
    for (let i = 0; i < len; i = i + 2 * step) {
      mergeArrays(
        arr, 
        i, 
        Math.min(i + step, len), 
        Math.min(i + 2 * step, len)
      );
    };
  }
  
  //对左右序列进行排序
  function mergeArrays(arr, left, right, end) {
    let n = left,
        m = right;
    const currentSort = [];
    
    for (let j = left; j < end; j++) {
      if ( n < right && ( m >= end || arr[n] < arr[m] )) {
        currentSort.push(arr[n]);
        n++;
      } else {
        currentSort.push(arr[m]);
        m++;
      }
    }
    //     
    currentSort.forEach(function(item, index) { 
      arr[left + index] = item; 
    });
  }
  
  return arr;
}
```



