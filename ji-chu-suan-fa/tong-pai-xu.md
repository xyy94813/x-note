# 桶排序

**桶排序（Bucket sort）**又称之为**箱排序**，将数组中的元素分到有限数量的桶里，再对桶内的元素进行排序，最终将非空的桶内的已排序数组合并。

> 对桶内元素进行排序时，可以采取其它的排序算法。

* 平均时间复杂度：$$ \mathcal{O}(n + k) $$
* 最好时间复杂度：$$ \mathcal{O}(n + k) $$
* 最坏时间复杂度：$$ \mathcal{O}(n^2) $$
* 空间复杂度：$$ \mathcal{O}(n * k) $$
* 稳定性：对桶内元素采取的排序算法导致稳定性无法确定
* 排序方式：非原地（not-in-place）

![](https://upload.wikimedia.org/wikipedia/commons/thumb/6/61/Bucket_sort_1.svg/311px-Bucket_sort_1.svg.png)

![](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e3/Bucket_sort_2.svg/311px-Bucket_sort_2.svg.png)

步骤：

1. 设置一个定量的数组当作空桶
2. 遍历序列，将元素放入对应的桶里
3. 对每个不是空的桶进行排序
4. 从非空的桶里将元素放回原来的序列中。

JavaScript 实现：

```js
function bucketSort(arr, step = 1) {
    const minVal = Math.min(...arr);
    const maxVal = Math.max(...arr);
    
    // 查找元素所属的桶的下标
    const findElementBucketIndex = item => Math.floor((item - minVal) / step);

    const bucketNum = findElementBucketIndex(maxVal) + 1; // 桶的数量    
    const buckets = []; // 所有桶的集合
    // const buckets = new Array(bucketNum);
    
    // 初始化桶
    for (let i = 0; i < bucketNum; ++i) {
        buckets[i] = [];
    }
        
    // 将元素放入指定的桶中
    arr.forEach((item) => {
        const bucketIndex = findElementBucketIndex(item); // 元素所属的桶的下标
        buckets[bucketIndex].push(item);
    })
    
    arr.splice(0); // 清空数组，也可以使用新的空间进行处理

    buckets
        .filter(item => item.length > 0) // 过滤空的桶
        .forEach(item => {
            // 对桶内元素排序
            // 此处可采取任意排序算法，因此采用系统默认排序算法替代
            item.sort(); 
            arr.push(...item);
        })
            
    return arr;    
}
```