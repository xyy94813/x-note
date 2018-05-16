# 快速排序

**快速排序（Quick Sort）**又称为**划分交换排序（partition-exchange sort）**是一种有效率的排序算法。

* 平均时间复杂度：$$O(n \log n)$$
* 最好时间复杂度：$$O(n \log n)$$
* 最坏时间复杂度：$$O(n^2)$$
* 空间复杂度：根据实现方式不同为$$O(n)$$或$$O(\log n)$$
* 稳定性：不稳定
* 排序方式：原地（In-place）

步骤：

1. 从序列中挑出一个元素作为**基准（Pivot）**，一般选取序列中的第一个；
2. 从后向前遍历剩余元素，找到第一个比基准小的元素的索引；
3. 从前向后遍历剩余元素，找到第一个比基准数大的元素的索引；
4. 交换两个元素；
5. 重复 2~4，直到从后向前的游标和从前向后的游标重叠；
6. 交换基准与当前两个游标重叠的位置的元素，此时，所有比基准值小的元素放在基准前面，所有比基准大的放在基准后面（相同的可放在前面或后面）；
7. 划分结束后，以基准所在位置划分左右两个分区；
8. 对新划分的分区执行 1~5，直到无法划分分区。

可以简单理解为，从每个的分区中，取一个基准值，然后找到这个基准值所在的位置。

![](https://upload.wikimedia.org/wikipedia/commons/6/6a/Sorting_quicksort_anim.gif)

递归实现：

```js
function swap(arr, i, j) {
    let tmp = arr[i];
    arr[i] = arr[j];
    arr[j] = tmp;
}

function quickSort(arr, left, right) {
    const len = arr.length;
    if (left > right) {
        return arr;
    }
    let pivot = arr[left];
    let i = left;
    let j = right;
    while(i != j) {
        while (arr[j] >= pivot && i < j) {
            j--;
        }
        while (arr[i] <= pivot && i < j) {
            i++;
        }
        if (i < j && arr[i] !== arr[j]) {
            swap(arr, i, j)
        }
    }

    if (left !== i && arr[left] !== arr[i]) {
        swap(arr, left, i);  
    }

    quickSort(arr, left, i-1);
    quickSort(arr, i+1, right);

    return arr;
}

let arr = [3, 7, 8, 5, 2, 1, 9, 5, 4];
quickSort(arr, 0, arr.length - 1); // [1, 2, 3, 4, 5, 5, 7, 8, 9]
```



