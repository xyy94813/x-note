# 选择排序

**选择排序（Selection Sort）**每次从未排序的序列中找出最小（大）的元素，然后放入排序序列的末尾，然后继续寻找最小（大）大元素。

* 平均时间复杂度：$$ O(n^2) $$
* 最好时间复杂度：$$ O(n^2) $$
* 最坏时间复杂度：$$ O(n^2) $$
* 空间复杂度：$$ O(1) $$
* 稳定性：不稳定
* 排序方式：原地（In-place）

选择排序的主要优势是，每次交换一对元素，至少有一个被移动到其最终位置上。

![](https://upload.wikimedia.org/wikipedia/commons/9/94/Selection-Sort-Animation.gif)

具体实现：

```js
function swap(arr, i, j) {
    let tmp = arr[i];
    arr[i] = arr[j];
    arr[j] = tmp;
}

function selectSort(arr) {
    const len = arr.length;
    for (let i = 0; i < len; ++i) {
        let minIndex = i;
        for (let j = i; j < len; ++j) {
            if (arr[i] > arr[j]) {
                minIndex = j;
            }
        }
        if (i !== minIndex) {
            swap(arr, i, minIndex);
        }
    }
    return arr;
}
```



