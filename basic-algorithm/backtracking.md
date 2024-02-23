# 回溯

回溯算法（Backtracking）也叫试探法，是一种能避免不必要搜索的穷举式的搜索算法。

基本思想是：从一条路往前走，能进则进，不能进则退回来，换一条路再试。

回溯算法实际上一个类似枚举的搜索尝试过程，主要是在搜索尝试过程中寻找问题的解，当发现已不满足求解条件时，就“回溯”返回，尝试别的路径。
满足回溯条件的某个状态的点称为“回溯点”。

回溯比暴力穷举法更高明的地方就是回溯算法使用剪枝函数，剪去一些不可能到达 最终状态（即答案状态）的节点，从而减少状态空间树节点的生成。

回溯的特点：

- 深度优先遍历。
- 求问题的所有解时，要回溯到根，且根结点的所有子树都已被搜索遍才结束。
- 求任一解时，只要搜索到问题的一个解就可以结束。

## 一般步骤

1. 针对所给问题，定义问题的解空间，它至少包含问题的一个（最优）解。
2. 确定易于搜索的解空间结构,使得能用回溯法方便地搜索整个解空间 。
3. 以深度优先的方式搜索解空间，并且在搜索过程中用剪枝函数避免无效搜索。

```js
const result = []; // 存放所欲符合条件结果的集合
const curPath = []; // 存放当前符合条件的结果

const backtracking = (eleList, curPath = [], result = []) => {
  if (遇到边界条件) {
    result.push(curPath);
  } else {
    // 枚举可选元素列表
    for (let ele of eleList) {
      curPath.push(ele);
      backtracking(eleList, curPath, result);
      path.pop(); // 回溯
    }
  }
};

// backtracking(eleList)
```

## 剪枝

某些问题如果其解空间过大，即使用回溯法进行计算也有很高的时间复杂度，因为回溯法会尝试解空间树中所有的分支。

优化剪枝策略，就是判断当前的分支树是否符合问题的条件，如果当前分支树不符合条件，那么就不再遍历这个分支里的所有路径。

启发式搜索策略指的是，给回溯法搜索子节点的顺序设定一个优先级，从该子节点往下遍历更有可能找到问题的解。

## 使用场景

### 分割回文串

[Leetcode 131](https://leetcode.com/problems/palindrome-partitioning/description/)

“给你一个字符串 s，请你将 s 分割成一些子串，使每个子串都是 回文串 。返回 s 所有可能的分割方案。”

```js
/**
 * @param {string} s
 * @return {string[][]}
 */
var partition = function (s) {
  const palindromeStrCache = new Map();
  const isPalindromeStr = (str, start, end) => {
    const key = `${start},${end}`;
    if (palindromeStrCache.has(key)) {
      return palindromeStrCache.get(key);
    }

    if (start >= end) {
      palindromeStrCache.set(key, true);
    } else if (str[start] === str[end]) {
      // 基于 DP 的计算是否是 Palindrome
      palindromeStrCache.set(key, isPalindromeStr(str, start + 1, end - 1));
    } else {
      palindromeStrCache.set(key, false);
    }

    return palindromeStrCache.get(key);
  };

  const core = (str, startIndex, curTmp, result) => {
    const strLen = str.length;
    if (startIndex < strLen) {
      for (let endIndex = startIndex; endIndex < strLen; ++endIndex) {
        if (isPalindromeStr(str, startIndex, endIndex)) {
          curTmp.push(str.slice(startIndex, endIndex + 1));
          // 执行具体的逻辑
          core(s, endIndex + 1, curTmp.slice(), result);
          // 回溯
          curTmp.pop();
        }
      }
    } else {
      // 达到终点，加入最终结果
      result.push(curTmp);
    }

    return result;
  };

  return core(s, 0, [], []);
};
```

### 排列组合

TODO

## 参考

[五大基本算法之回溯算法 backtracking](https://houbb.github.io/2020/01/23/data-struct-learn-07-base-backtracking)
[回溯算法知识](https://algo.itcharge.cn/09.Algorithm-Base/04.Backtracking-Algorithm/01.Backtracking-Algorithm/)
