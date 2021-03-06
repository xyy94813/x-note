# 正则表达式的悲观回溯

正则表达式中`*`表示匹配日前面的子表达式0次或多次（且尽可能多的匹配）。

---

假设有正则表达式`/^(a*)b$/a`和字符串`aaaaab`。

二者匹配，此时捕获组`(a*)`捕获到的字符串为`aaaaa`。

---

改写正则表达式为`/^(a*)ab$/`。

此时与字符串依然匹配，但是捕获到的字符串为`aaaa`。

---

仔细剖析这个表达式`/^(a*)ab$/`的匹配过程：

1. 匹配开始，`(a*)`匹配尽可能多的字符`a`；
2. `(a*)`一直捕获，直到遇到字符`b`。此时`(a*)`已经捕获了`aaaaa`；
3. 正则表达式继续执行`(a*)`之后的`ab`匹配，此时字符串仅剩一个`b`，导致无法完成匹配；
4. 从`(a*)`当前捕获的字符串中”吐出“一个字符`a`，这时捕获结果为`aaaa`，剩余字符串为`ab`;
5. 重新执行正则中的`ab`匹配。完成匹配。捕获组捕获结果为`aaaa`。

---

暂时的无法匹配并不会立即导致整体匹配的失败。而是会从捕获组中吐出字符后继续尝试，直到成功或者完全失败。这个“吐出“的过程就叫做**回溯**。

假设这个表达式`/^(a*)b$/`匹配字符串`aaaaaa`.

> 尽管能够一眼看出来无法匹配，但是正则表达式依旧会逐一回溯所有可能性，才能确定最终结果。这中“傻傻的”回溯过程就叫做_**悲观回溯**_。

这种悲观回溯，很可能会导致性能问题。

## 解决方案

### 1. 禁止回溯

有两种语法可以防止回溯

* 有限量词
* 原子分组

不过在 JavaScript 中均不被支持

### 2. 避免导致性能问题的回溯

以下两种模式的正则表达式很可能导致有性能问题的回溯

* 前后重复模式，例如`/x*x*/`；
* 嵌套量词，例如`/(x*)*/`。

### 3. 不使用正则

例如匹配字符串`aaaaa,bbbbbb,ccccc`中`,`分割的部分，使用`split()`对字符串进行切割即可，还能保证代码的性能是线性的。

### 4. 避免性能问题影响页面响应

在必须使用正则表达式，且这个表达式有潜在的性能问题的情况下。可以吧匹配操作放到 Service Worker 中进行。尽量的不影响页面响应

