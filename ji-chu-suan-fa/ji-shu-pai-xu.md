# 计数排序

**计数排序（Counting sort）**是一种稳定的线性时间排序算法。计数排序使用一个长度为`maxVal - minVal + 1`的额外的数据结构 $$\mathcal{C}$$，其中第 i 个元素是待排序数组 $$\mathcal{A}$$ 中值等于 i 的元素个数。然后根据数组 $$\mathcal{C}$$ 来将 $$\mathcal{A}$$ 的元素排到正确的位置。

> 计数排序本质上是一种特殊的桶排序，当桶的个数最大的时候，就是计数排序。

* 平均时间复杂度：$$ \mathcal{O}(n + k) $$
* 最好时间复杂度：$$ \mathcal{O}(n + k) $$
* 最坏时间复杂度：$$ \mathcal{O}(n + k) $$
* 空间复杂度：$$ \mathcal{O}(n + k) $$
* 稳定性：稳定
* 排序方式：非原地（not-in-place）

> $$ k = maxVal - minVal $$， 为最大值和最小值的差值。也就是说，当最大值和最小值相差过大，但是数组数量较小时，不应采用计数排序

算法步骤：  
1. 找出待排序的数组中最大和最小的元素  
2. 统计数组 $$\mathcal{A}$$ 中每个值为 i 的元素出现的次数，存入数组 $$\mathcal{C}$$ 的第 i 项  
3. 对所有的计数累加（从 $$\mathcal{C}$$ 中的第一个元素开始，每一项和前一项相加）  
4. 反向填充目标数组：将每个元素 i 放在新数组的第 $$\mathcal{C}[{i}]$$ 项，每放一个元素就将 $$\mathcal{C}[{i}]$$ 减去 1

JavaScript 实现：

```js
function countingSort(arr) {
    // JavaScript 数组本身就是动态分配的内存，
    // 对于 JavaScript 简略版本，完全可以不去考虑额外结构内存分配，    
    let minVal = Math.min(...arr); 
    let maxVal = Math.max(...arr); 
    const c = new Array(maxVal - minVal + 1);
    // const c = [];
    arr.forEach(item => {
        let index = item - minVal;
        let counter = c[index];
        c[index] = counter ? counter + 1 : 1;
    })
    
    arr.splice(0); // 清空数组
    c.forEach((item, index) => {
        while(item) {
            arr.push(index + minVal);
            item --;
        }
    });
    return arr;
}
```



