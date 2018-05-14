# 插入排序

**插入排序（Insertion Sort）**的工作原理是通过构建有序序列，对于未排序数据，在一排序序列中扫描，找到相应位置并插入。

* 平均时间复杂度：$$O(n^2)$$
* 最好时间复杂度：$$O(n)$$
* 最坏时间复杂度：$$O(n^2)$$
* 空间复杂度：$$O(1)$$
* 稳定性：稳定
* 排序方式：原地（In-place）

运作方式：

1. 将第一个元素放入有序序列中
2. 取出下一个元素，（从后向前）扫描已排序的元素序列
3. 如果该元素大于新元素，将该元素移到下一个位置
4. 重复步骤 3，直到找到该元素在已排序数组中的位置
5. 将新元素插入到该位置
6. 重复步骤 2~5

![](https://upload.wikimedia.org/wikipedia/commons/thumb/0/0f/Insertion-sort-example-300px.gif/220px-Insertion-sort-example-300px.gif)

具体实现：

```js
function insertSort(arr) {
    for (let i = 1; i < arr.length; ++i) {
        let tmp = arr[i]; // 减少索引访问
        // 数组的 0~i-1 部分，即已排序序列 
        for (let j = i - 1; j >= 0; --j) {
            if (tmp >= arr[j]) {
                break;
            }
            arr[j + 1] = arr[j];
            arr[j] = tmp;
        }
    }
    return arr;    
}
```



