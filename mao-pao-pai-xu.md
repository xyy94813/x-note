# 冒泡排序

冒泡排序是最经典的排序算法之一，重复地走访要排序的数列，依次比较两个元素，如果顺序错误就把它们交换过来，直到排序完成。

* 平均时间复杂度：O($$ n^2 $$)
* 最好时间复杂度：O(n)
* 最坏时间复杂度：O\($$ n^2 $$)
* 空间复杂度：O(1)
* 稳定性：稳定
* 排序方式：原地（In-place）

运作方式：

1. 比较相邻的元素。如果第一个比第二个大，交换；
2. 对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对；
3. 针对所有的元素重复以上的步骤；
4. 持续每次对越来越少的元素重复上述步骤，直到没有任何一组数字需要比较。

具体实现：

```js
function bubbleSort(arr) {
    const len = arr.length;
    for (let i = 0; i < len - 1; ++i) {
        for (let j = 0; j < len - 1 - i; ++j) {
            if (arr[j] > arr[j+1]) {
                swap(arr, j, j+1);
            }
        }
    }
    return arr;
}

// 交换数组的两个索引的值
function swap(arr, i, j) {
    const tmp = arr[i];
    arr[i] = arr[j];
    arr[j] = tmp;
}
```

冒泡排序虽然是简单的排序算法之一，但是大多数人多实现的方式都是错误的，最常见的错误的冒泡排序：

```js
function errorBubbleSort(arr) {
    const len = arr.length;
    for (let i = 0; i < len; ++i) {
        for (let j = 0; j < len; ++j) {
            if (arr[i] < arr[j]) {
                swap(arr, i, j);
            }
        }
    }
    return arr;
}
errorBubbleSort([5, 3, 1, 2, 4]); // [1, 2, 3, 4, 5]

function errorBubbleSort2(arr) {
    const len = arr.length;
    for (let i = 0; i < len; ++i) {
        for (let j = i + 1; j < len; ++j) {
            if (arr[i] > arr[j]) {
                swap(arr, i, j);
            }
        }
    }
    return arr;
}
errorBubbleSort2([5, 3, 1, 2, 4]); // [1, 2, 3, 4, 5]
```



