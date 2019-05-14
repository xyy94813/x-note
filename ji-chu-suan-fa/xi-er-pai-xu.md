# 希尔排序

**希尔排序（Shell Sort）**，也称作**递减增量排序算法**，以起设计者 Donald Shell 的名字命名，是插入排序的一种更高效的改进版本。

希尔排序通过将比较的全部元素分为几个区域来提升插入排序的性能。这样可以让一个元素可以一次性地朝最终位置前进一大步。然后算法再取越来越小的步长进行排序，算法的最后一步就是普通的插入排序，但是到了这步，需排序的数据几乎是已排好的了（此时插入排序较快）。

* 平均时间复杂度：根据步长序列的不同而不同
* 最好时间复杂度：$$ \mathcal{O}({n}\log_{2}{n}) $$
* 最坏时间复杂度：$$ \mathcal{O}({n}^{2}) $$
* 空间复杂度：$$ \mathcal{O}(n) $
* 稳定性：不稳定
* 排序方式：原地（in-place）

步长的选择是希尔排序的重要部分。只要最终步长为1任何步长序列都可以工作。算法最开始以一定的步长进行排序。然后会继续以一定步长进行排序，最终算法以步长为1进行排序。当步长为1时，算法变为插入排序，这就保证了数据一定会被排序。 Donald Shell 最初建议步长选择为 $$ floor(\dfrac{n}{2}) $$，并且对步长取半直到步长达到1。虽然这样取可以比 $$ \mathcal{O}(n^{2}) $$ 类的算法（插入排序）更好，但这样仍然有减少平均时间和最差时间的余地。

比如，如果一个数列以步长 5 进行了排序然后再以步长 3 进行排序，那么该数列不仅是以步长 3 有序，而且是以步长 5 有序。如果不是这样，那么算法在迭代过程中会打乱以前的顺序，那就不会以如此短的时间完成排序了。

一般实现：

```js
function shellSort(arr) {
    if (!Array.isArray(arr)) {
        throw TypeError('arr must be an array')
    }
    const len = arr.length;
    if (len <= 1)  {
        return arr;
    }
    // len >> 1 === Math.floor(len / 2)
    for (let step = len >> 1; step > 0; step >>= 1) {
        for (let i = step; i < len; ++i) {
            let tmp = arr[i];
            let j;
            for (j = i - step; j >= 0 && arr[j] > tmp; j -= step) {
                arr[j + step] = arr[j];
            }
            arr[j + step] = tmp;
        }
    }
    return arr;
}
```



