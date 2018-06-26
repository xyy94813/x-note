# 基数排序

**基数排序（Radix Sort）**是一种非比较型整数排序算法。基数排序可以看作是桶排序的扩展，核心思想是将所有整数统一为位数相同的证书，位数较少的前面补零，然后按照位数切割，对每个位数分别进行比较和排序。

基数排序还分为了 **LSD（Least significant digital）** 和 **MSD（Most significant diagital）**两种实现方式。其中，LSD 先比较低位的数字，MSD 反之。

* 平均时间复杂度：$$\mathcal{O}(n * k)$$
* 最好时间复杂度：$$\mathcal{O}(n * k)$$
* 最坏时间复杂度：$$\mathcal{O}(n * k)$$
* 最坏空间复杂度：$$\mathcal{O}(n + k)$$
* 稳定性：稳定
* 排序方式：非原地（not-in-place）

LSD：
![](https://camo.githubusercontent.com/4376d13c2f2b6425681038a614ccdce1cf0c1893/68747470733a2f2f7777772e7265736561726368676174652e6e65742f7075626c69636174696f6e2f3239313038363233312f6669677572652f666967312f41533a36313432313434353234303432343040313532333435313534353536382f53696d706c69737469632d696c6c757374726174696f6e2d6f662d7468652d73746570732d706572666f726d65642d696e2d612d72616469782d736f72742d496e2d746869732d6578616d706c652d7468652e706e67)

LSD 实现：

```js
function radixSort(arr) {
    const maxVal = Math.max(...arr);
    const maxValLen = `${maxVal}`.length; // 最大的数的位数
    
    // 补零
    const _arr = arr.map((item) => `${item}`.padStart(maxValLen, '0'));
    
    // 从低位开始对每一位进行比较后排序
    for (let i = maxValLen - 1; i >=0; --i) {
        _arr.sort((a, b) => parseInt(a[i], 10) - parseInt(b[i], 10));
    } 
    
    // 赋值回 arr
    _arr.forEach((item, index) => {
        arr[index] = parseInt(item, 10);
    });
    
    return arr;
}
```