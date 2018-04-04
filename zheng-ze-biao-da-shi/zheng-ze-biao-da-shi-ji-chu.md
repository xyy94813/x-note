# 正则表达式基础

正则表达式是由普通字符（例如字符 a 到 z）以及**特殊字符（称为"元字符"）**组成的文字模式。

### 元字符

> 表达式中具有特殊含义的字符

**常用元字符：**

| 符号 | 含义 |
| :--- | :--- |
| `.` | 匹配除换行符以外的任意字符 |
| `\w` | 匹配字母或数字或下划线或汉字 |
| `\s` | 匹配任意的空白符 |
| `\d` | 匹配数字 |
| `\b` | 匹配单词的开始或结束 |
| `^` | 匹配字符串开始 |
| `$` | 匹配字符串的结束 |

**字符转义**

| 符号 | 含义 |
| :--- | :--- |
| `\` | 转义特殊字符 |

**重复**

| 符号 | 含义 |
| :--- | :--- |
| `*` | 重复零次或更多次 |
| `+` | 重复一次或更多次 |
| `?` | 重复零次或一次 |
| `{n}` | 重复 n 次 |
| `{n,}` | 重复 n 次或更多次 |
| `{n,m}` | 重复 n 次到 m 次 |

**字符类**

| 符号 | 含义 |
| :--- | :--- |
| \[str\] | 匹配限定字符 |

```js
const reg = /^[a-zA-Z0-9]$/;
reg.test('b'); // true
reg.test('B'); // true
reg.test('2'); // true
reg.test('!'); // false
```

**分支条件**

> 使用分支条件时，要注意各个条件的顺序

| 符号 | 含义 |
| :--- | :--- |
| \` | 将不同规则分割开，满足任意一种都能成功 |

```js
const reg = /(^a)|(b$)/; // a 开头或 b 结尾
reg.test('asdfghjkl;zxcv'); // true
reg.test('sdfghjkl;zxcvb'); // true
reg.test('sdfghjkl;zxcv'); // false
```

**分组**

用小括号来指定**子表达式（也叫做分组）**，然后就可以指定这个子表达式重复的次数。每个分组都会自动拥有一个组号，规则是：从左向右，以分组的左括号为标志，第一个出现的分组组号为`1`，第二个为`2`，一次类推。分组`0`对应整个表达式。

> 实际上，组号分配的过程要从左向右扫描两边：第一遍只给未命名组分配组号。第二遍只给命名组分配。因此所有命名组的组号都大于未命名的组号

| 符号 | 含义 |
| :--- | :--- |
| `(exp)` | 匹配表达式 exp，并捕获文本到自动命名的组 |
| `(?<name>exp)`或`(?'name'exp)` | 匹配表达式 exp，并捕获文本到名称为name的组里 |
| `(?:exp)` | 匹配表达式 exp，不捕获匹配的文本，也不给此组分配组号 |

**反义**

| 符号 | 含义 |
| :--- | :--- |
| `\W` | 匹配任意不是字母，数字，下划线，汉字的字符 |
| `\S` | 匹配任意不是空白符的字符 |
| `\D` | 匹配任意非数字的字符 |
| `\B` | 匹配任意非数字的字符 |
| `[^str]` | 匹配除了str以外的任意字符 |

```js
const reg = /^[^a-zA-Z0-9]$/;
reg.test('b'); // false
reg.test('B'); // false
reg.test('2'); // false
reg.test('!'); // true
```

**后向引用**

后向引用用于重复搜索前面某个分组匹配的文本。

| 符号 | 含义 |
| :--- | :--- |
| `\(组名)` | 匹配某个分组匹配的文本 |

```js
const reg = /^(\d)\1$/; // `\1` 代表分组 1 匹配的文本
reg.test('11'); // true
reg.test('12'); // false
```

**零宽断言**

用于查找在某些内容（但不包括这些内容）之前或之后的东西，也就是说它们像`\b` ，`^` ，`$`那样用于指定某个位置，这个位置应该满足一定条件（断言），即为零宽断言。

| 符号 | 含义 |
| :--- | :--- |
| `(?=exp)` | 匹配表达式 exp 前面的位置 |
| `(?<=exp)` | 匹配表达式 exp 后面的位置 |
| `(?!exp)` | 匹配后面跟的不是表达式 exp 的位置 |
| `(?<!exp)` | 匹配前面跟的不是表达式 exp 的位置 |

**注释**

| 符号 | 含义 |
| :--- | :--- |
| `(?#comment)` | 注释 |

**贪婪与懒惰**

当正则表达式中包含能接受重复的限定符时，通常的行为是（在使整个表达式能得到匹配的前提下）匹配尽可能多的字符。以`/a.*b/`为例，它将匹配最长的以 a 开始，以 b 结束的字符串。如果用它来匹配 `aabab` 的话，它会匹配整个字符串 `aabab`。这就是贪婪匹配。

| 符号 | 含义 |
| :--- | :--- |
| `*?` | 重复任意次，尽可能少的重复 |
| `+?` | 重复 1 次或更多次，尽可能少的重复 |
| `??` | 重复 0 次或 1 次，尽可能少的重复 |
| `{n,}?` | 重复 n 次以上，尽可能少的重复 |
| `{n,m}?` | 重复 n 到 m 次，尽可能少的重复 |

**平衡组/递归匹配**

| 符号 | 含义 |
| :--- | :--- |
| `(?'group')` | 把捕获的内容命名为 group，并压入堆栈 |
| `(?'-group')` | 从堆栈上弹出最后压入堆栈的名为group的捕获内容，如果堆栈本来为空，则本次分组的匹配失败 |
| `(?(group)yes|no)` | 如果堆栈上存在以名为 group 的捕获内容的话，继续匹配 yes 部分，否则匹配 no 部分 |
| `(?!)` | 零宽负向先行断言，由于没有后缀表达式，试图匹配总是失败 |

** 其他**

| 符号 | 含义 |
| :--- | :--- |
| `\a` | 报警字符（打印它的效果是电脑“嘀”一声） |
| `\b` | 通常是单词分界位置 |
| `\t` | 制表符 |
| `\r` | 回车 |
| `\v` | 竖向制表符 |
| `\f` | 换页符 |
| `\n` | 换行符 |
| `\e` | Escape |
| `\0nn` | ASCLL 代码为 nn 的字符 |
| `\xnn` | ASCLL 代码中十六进制代码为 nn 的字符 |
| `\unnnn` | Unicode 代码中十六进制代码为 nnnn 的代码 |
| `\cN` | ASCLL 控制字符。比如 \cC 代表 Ctrl+C |
| `\A` | 字符串开头，类似`^`，但不受处理多行选项的影响 |
| `\Z` | 字符串结尾或行尾，不受多行选项的影响 |
| `\z` | 字符串结尾（类似`$`，但不受处理多行选项的影响） |
| `\G` | 当前搜索的开头 |
| `\p{name}` | Unicode 中命名为 name 的字符串类，例如 `\p{IsGreek}` |
| `(?>exp)` | 贪婪子表达式 |
| `(?<x>-<y>exp)` | 平衡组 |
| `(?im-nsx:exp)` | 在子表达式后面的部分改变处理选项 |
| `(?im-nsx)` | 为表达式后面的部分改变处理选项 |
| `(?(exp)yes|no)` | 把 exp 当作零宽正向现行断言，如果在这个位置能匹配，使用yes 作为词组的表达式，否则使用 no |
| `(?(exp)yes)` | 同上，只是使用空表达式作为 no |
| `(?(name)yes|no)` | 如果命名为 name 的组捕获到了内容，使用 yes 作为表达式；否则使用 no |
| `(?(name)yes)` | 同上，只是使用空表达式作为 no |

### 处理选项

> 各个语言提供的处理选项有细微的出入

* IgnoreCase —— 忽略大小写

* Multiline —— 多行模式。更改`^`和`$`的含义，使它们分别在任意一行的行首和行尾匹配，而不仅仅在整个字符串的开头和结尾匹配。（在此模式下，`$`的精确含义是：匹配`\n`之前的位置以及字符串结束前的位置）

* Singleline —— 单行模式。更改`.`的含义，使它与每一个字符匹配。\(包括换行符`\n`\)

* IgnorePatternWhitespace —— 忽略表达式中的非转义空白并启用由`#`标记的注释

* ExplicitCapture —— 仅捕获已被显示命名的组



JavaScript RegExp 对象中提供的处理选项

* g —— 全局匹配
* i —— 忽略大小写
* m —— 多行模式
* u —— 将模式视为 Unicode 序列点点序列
* y —— 粘性匹配；仅匹配目标字符串中此正则表达式的 lastIndex 属性指示的索引，并且不尝试从任何后续的索引匹配。

```js
const reg = /^a$/i;
reg.test('a'); // true
reg.test('A'); // true
```

## 参考

* [正则表达式30分钟入门教程](https://deerchao.net/tutorials/regex/regex.htm)


