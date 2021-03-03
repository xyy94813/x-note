# Git 内部原理

从根本上来讲 Git 是一个内容寻址（content-addressable）文件系统，并在此之上提供了一个版本控制系统的用户界面

## Git 目录

当在一个新目录或已有目录执行 git init 时，Git 会创建一个 .git 目录。
这个目录包含了几乎所有 Git 存储和操作的东西。

```
$ ls -F1 .git
config
description
HEAD # 指向目前被检出的分支
hooks/
info/
objects/ # 存储所有数据内容
refs/ # 存储指向数据（分支、远程仓库和标签等）的提交对象的指针

# 随着 Git 使用，目录下可能还会包含其他内容

index # 暂存区信息
...

# 同样，随着版本的不同，该目录下可能还会包含其他内容

```

## Git Object

Git 是一个内容寻址文件系统，这意味着，Git 的核心部分是一个简单的键值对数据库（key-value data store）。
可以向 Git 仓库中插入任意类型的内容，它会返回一个唯一的 key，通过该键可以在任意时刻再次取回该内容。

_这个 key 一般为一个长度为 40 的 SHA-1 哈希值 —— 一个将待存储的数据外加一个头部信息（header）一起做 SHA-1 校验运算而得的校验和。_

Git 存储内容的方式 —— 一个文件对应一条内容， 以该内容加上特定头部信息一起的 SHA-1 校验和为文件命名

```
$find .git/objects -type f

.git/objects/61/3f5f0f3bf9224ababb22f5d5db9574f29c7a7d
.git/objects/61/b8f2edc772597698bfad430f6c8926ff8eecf6
```

**校验和的前两个字符用于命名子目录，余下的 38 个字符则用作文件名。**
最后，Git 将内容存经 zlib 压缩后，存入入对应地址

一旦将内容存储在了对象数据库中，那么可以通过 `cat-file` 命令从 Git 那里取回数据。

```
$git cat-file -p 613f5f0f3bf9224ababb22f5d5db9574f29c7a7d

# 将会展示 613f5f0f3bf9224ababb22f5d5db9574f29c7a7d 对应的文件内容
# 613f5f0f3bf9224ababb22f5d5db9574f29c7a7d 为一个
```

```
$ git cat-file -t 613f5f0f3bf9224ababb22f5d5db9574f29c7a7d
commit
```

### Git Object Type

Git Object 存在以下几种类型

- blob
- tree
- commit
- tag

### Blob Object

**数据对象（blob object）**，用于存储文件内容。
当 git 添加一个文件，例如 `example_file.txt` 时，git 会创建一个包含文件内容的 blob-object。

### Tree Object

Git 以一种类似于 UNIX 文件系统的方式存储内容，但作了些许简化。
所有内容均以树对象和数据对象的形式存储，其中树对象对应了 UNIX 中的目录项，数据对象则大致上对应了 inodes 或文件内容。

Git 使用 **树对象（tree object）**，解决文件名保存，以及多个文件组织的问题。

一个 tree-object 包含了一条或多条**树对象记录（tree entry）**，
每条记录含有一个指向 blob-object 或者 sub tree-object 的 SHA-1 指针，以及相应的模式、类型、文件名信息。

```
$ git cat-file -p master^{tree}
100644 blob a906cb2a4a904a152e80877d4088654daad0c859      README
040000 tree 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0      lib

$ git cat-file -p 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0
100644 blob ec2394f9a74cc49be02a1551d49f179d73943bd2      lib-file.md
```

![Git 的数据内容结构](/images/simple-version-of-the-git-data-model.png)

### Commit Object

**提交对象（Commit Object）**包含目录树对象哈希，父提交哈希，作者，提交者，日期和消息。

```
$ git cat-file -p 84979b887f1f551b933f83c62e35763e92802532
tree 8ea9f4a5cf15af8e4918ec418a53fc7eabc4d332
parent 8f29c536730b7909d25cc033a65e8ba7e4cffe7f
author Author Name <author@email.com> 1614089208 +0800
committer Author Name <author@email.com> 1614089208 +0800

commit message
```

![Git 目录下所有可达的对象](/images/the-content-structure-of-your-current-git-data.png)

### Tag Object

**标签对象（tag object）** 非常类似于一个提交对象，它包含一个标签创建者信息、一个日期、一段注释信息，以及一个指针。
主要的区别在于，标签对象通常指向一个提交对象，而不是一个树对象。
它像是一个永不移动的分支引用，永远指向同一个提交对象，只不过给这个提交对象加上一个更友好的名字罢了。

```
$ git tag -a first-commit -m "Tag pointing to first commit"
$ git cat-file -t f793addc4a909bb148a0547178a9ec130f2bf95b
tag
```

```
$ git cat-file -p f793addc4a909bb148a0547178a9ec130f2bf95b
object 7810a8e30b6142d51541655d66d1ad294c908e90
type commit
tag first-commit
tagger Tagger Name <tagger@email.com> 1601974623 +0800

Tag pointing to first commit
```

PS: 标签对象并非必须指向某个提交对象；可以对任意类型的 Git 对象打标签。

## 如何计算 SHA-1

假设某文件内容为 "what is up, doc?"

```js
// Git 首先会以识别出的对象的类型作为开头来构造一个头部信息，本例中是一个“blob”字符串。
const header = `${objectType} ${content.length}\0`; // "blob 16\u0000"
const store = header + content; // "blob 16\u0000what is up, doc?"
const sha1 = computeSha1(store); // "bd9dbf5aae1a3862dd1526723246b20206e5fc37"
```

## Git 引用

在 Git 中，使用一个文件来保存 SHA-1 值，而该文件有一个名字，这种名字被称为“**引用（references，或简写为 refs）**”。
这些文件被存储在 refs 文件目录中.

Git 分支的本质就是 Refs，例如，master 分支。

```
$find .git/refs
.git/refs
.git/refs/heads
.git/refs/heads/master
.git/refs/heads/feature/doSomething
.git/refs/tags
.git/refs/remotes
.git/refs/remotes/origin
.git/refs/remotes/origin/HEAD
.git/refs/remotes/origin/master
.git/refs/remotes/upstream
.git/refs/remotes/upstream/HEAD
.git/refs/remotes/upstream/master
```

![包含分支引用的 Git 目录对象](../images/git-directory-objects-with-branch-head-references-included.png)

### HEAD 引用

HEAD 文件通常是一个**符号引用（symbolic reference）**，指向目前所在的分支。
所谓符号引用，表示它是一个指向其他引用的指针。

```
$cat .git/head
ref: refs/heads/feature/git-internals
```

在某些罕见的情况下，HEAD 文件可能会包含一个 git 对象的 SHA-1 值。
例如，merge 失败，解决冲突的过程中。

本质上，通过 HEAD 最终的得到的也是一个 git 对象。

### 标签引用

存在两种类型的标签：附注标签和轻量标签。

创建一个轻量标签：

```
$ git update-ref refs/tags/v1.0 cac0cab538b970a37ea1e769cbbde608743bc96d
```

这就是轻量标签的全部内容，一个固定的引用。

附注标签则更复杂一些。
若要创建一个附注标签，Git 会创建一个标签对象，并记录一个引用来指向该标签对象，而不是直接指向提交对象。

```
$ git tag -a v1.1 1a410efbd13591db07496601ebc7a059dd55cfe9 -m 'test tag'
```

```
$ cat .git/refs/tags/v1.1
9585191f37f7b0fb9444f35a9bf50de191beadc2
```

```
$ git cat-file -p 9585191f37f7b0fb9444f35a9bf50de191beadc2
object 1a410efbd13591db07496601ebc7a059dd55cfe9
type commit
tag v1.1
tagger Author <author@email.com> Sat May 23 16:48:58 2020 +0800

test tag
```

### 远程引用

如果添加了一个远程版本库并对其执行过推送操作，Git 会记录下最近一次推送操作时每一个分支所对应的值，并保存在 refs/remotes 目录下。
**远程引用（remote reference）**和分支（位于 refs/heads 目录下的引用）之间最主要的区别在于，远程引用是只读的。

## 包文件

每次提交文件，Git 都会用一个全新的对象来存储新的文件内容，就算只是在一个 400 行的文件后面加入一行新内容。

如果 Git 只完整保存其中一个，再保存另一个对象与之前版本的差异内容，岂不更好？

Git 最初向磁盘中存储对象时所使用的格式被称为 **“松散（loose）”**对象格式。
但是，Git 会时不时地将多个这些对象打包成一个称为 **“包文件（packfile）”** 的二进制文件，以节省空间和提高效率。
当版本库中有太多的松散对象，或者手动执行 git gc 命令，或者向远程服务器执行推送时，Git 都会这样做。

Git 打包对象时，会查找命名及大小相近的文件，并只保存文件不同版本之间的差异内容。

打包对象之后原始的版本将以差异方式保存的，这是因为大部分情况下需要快速访问文件的最新版本。

## 引用规范

**引用规范（refspec）**的格式为 `(+)<src>:<dst>`。
`<src>` 代表远程版本库中的引用；
`<dst>` 是本地跟踪的远程引用的位置；
`+` 号告诉 Git 即使在不能快进的情况下也要（强制）更新引用。

`<src>`和`<dst>` 均支持以表达式表达。

```
$cat .git/config
...

[remote "origin"]
        url = https://github.com/xyy94813/x-note.git
        fetch = +refs/heads/*:refs/remotes/origin/*

...
```

如果想让 Git 每次只拉取远程的 master 分支，而不是所有分支， 可以把（引用规范的）获取那一行修改为只引用该分支：

```
fetch = +refs/heads/master:refs/remotes/origin/master
```

## 传输协议

Git 可以通过两种主要的方式在版本库之间传输数据： “dumb” 协议和 “smart” 协议。

### Dumb 协议

如果正在架设一个基于 HTTP 协议的只读版本库，一般而言这种情况下使用的就是 Dumb 协议。
这个协议之所以被称为 “Dumb 协议”，是因为在传输过程中，服务端不需要有针对 Git 特有的代码；
抓取过程是一系列 HTTP 的 GET 请求，这种情况下，客户端可以推断出服务端 Git 仓库的布局。

Dumb 协议步骤：

1. 获取服务器端 info/refs
2. 基于 info/refs，检出对应的 ref，一般包含一个 Tree-Object 和一个 Blob-Object
3. 获取对象信息，
   1. 如果服务器端存在，从服务器端获取；
   2. 否则，检查所有替代版本库。如果派生项目存在，从派生项目获取；
   3. 如果替代版本库找不到该对象，检查服务端有哪些可用的包文件，从包文件中查找对象信息
4. 重复步骤 3，直到获取完所有所需的目录信息

Dumb 协议虽然很简单但效率略低，且它不能从客户端向服务端发送数据。

### Smart 协议

智能协议是更常用的传送数据的方法，但它需要在服务端运行一个进程，而这也是 Git 的智能之处 —— 它可以读取本地数据，理解客户端有什么和需要什么，并为它生成合适的包文件。
总共有两组进程用于传输数据，它们分别负责上传和下载数据。

#### 上传数据，通过 SSH

为了上传数据至远端，Git 使用 `send-pack` 和 `receive-pack` 进程。

运行在客户端上的 `send-pack` 进程连接到远端运行的 `receive-pack` 进程。

Git 运行 `send-pack` 进程，通过 SSH 连接 Git 服务器，然后在服务端执行命令。

类似于：

```
$ ssh -x git@server "git-receive-pack 'simplegit-progit.git'"
00a5ca82a6dff817ec66f4437202690a93763949 refs/heads/master report-status \
	delete-refs side-band-64k quiet ofs-delta \
	agent=git/2:2.1.1+github-607-gfba4028 delete-refs
0000
```

#### 上传数据，通过 HTTP

上传过程在 HTTP 上，与 SSH 几乎是相同的，除了握手阶段有一点小区别。
连接是从下面这个请求开始的：

```
=> GET http://server/simplegit-progit.git/info/refs?service=git-receive-pack
001f# service=git-receive-pack
00ab6c5f0e45abd7832bf23074a333f739977c9e8188 refs/heads/master report-status \
	delete-refs side-band-64k quiet ofs-delta \
	agent=git/2:2.1.1~vmg-bitmaps-bugaloo-608-g116744e
0000
```

接下来客户端发起另一个请求，这次是一个 POST 请求，这个请求中包含了 `send-pack` 提供的数据。

```
=> POST http://server/simplegit-progit.git/git-receive-pack
```

这个 POST 请求的内容是 `send-pack` 的输出和相应的包文件。 服务端在收到请求后相应地作出成功或失败的 HTTP 响应。

#### 下载数据，通过 SSH

客户端启动 `fetch-pack` 进程，连接至远端的 `upload-pack` 进程，以协商后续传输的数据。

如果通过 SSH 使用抓取功能，fetch-pack 会像这样运行：

```
$ ssh -x git@server "git-upload-pack 'simplegit-progit.git'"
```

在 `fetch-pack` 连接后，`upload-pack` 会返回类似下面的内容：

```
00dfca82a6dff817ec66f44342007202690a93763949 HEAD multi_ack thin-pack \
	side-band side-band-64k ofs-delta shallow no-progress include-tag \
	multi_ack_detailed symref=HEAD:refs/heads/master \
	agent=git/2:2.1.1+github-607-gfba4028
003fe2409a098dc3e53539a9028a94b6224db9d6a6b6 refs/heads/master
0000
```

这与 `receive-pack` 的响应很相似，但是这里还包含 `HEAD` 引用所指向内容`（symref=HEAD:refs/heads/master）`，
这样如果客户端执行的是克隆，它就会知道要检出什么。

#### 下载数据，通过 HTTP

抓取操作的握手需要两个 HTTP 请求。
第一个是向和 Dumb 协议中相同的端点发送 GET 请求：

```
=> GET $GIT_URL/info/refs?service=git-upload-pack
001e# service=git-upload-pack
00e7ca82a6dff817ec66f44342007202690a93763949 HEAD multi_ack thin-pack \
	side-band side-band-64k ofs-delta shallow no-progress include-tag \
	multi_ack_detailed no-done symref=HEAD:refs/heads/master \
	agent=git/2:2.1.1+github-607-gfba4028
003fca82a6dff817ec66f44342007202690a93763949 refs/heads/master
0000
```

这和通过 SSH 使用 `git-upload-pack` 是非常相似的，但是第二个数据交换则是一个单独的请求：

```
=> POST $GIT_URL/git-upload-pack HTTP/1.0
0032want 0a53e9ddeaddad63ad106860237bbf53411d11a7
0032have 441b40d833fdfa93eb2908e52742248faf0ee993
0000
```

## 维护与数据恢复

有的时候，需要对仓库进行清理 —— 使它的结构变得更紧凑，或是对导入的仓库进行清理，或是恢复丢失的内容。

### 维护

Git 会不定时地自动运行一个叫做 `auto gc` 的命令。
如果有太多松散对象（不在包文件中的对象）或者太多包文件，Git 会运行一个完整的 `git gc` 命令。

这个命令会做以下事情：

- 收集所有松散对象并将它们放置到包文件中， 将多个包文件合并为一个大的包文件，移除与任何提交都不相关的陈旧对象。
- 打包引用到一个单独的文件

大约需要 7000 个以上的松散对象或超过 50 个的包文件才能让 Git 启动一次真正的 gc 命令。
可以通过修改 `gc.auto` 与 `gc.autopacklimit` 的设置来改动这些数值。

如果更新了引用，Git 并不会修改这个文件，而是向 `refs/heads` 创建一个新的文件。
为了获得指定引用的正确 SHA-1 值，Git 会首先在 `refs` 目录中查找指定的引用，然后再到 `packed-refs` 文件中查找。

### 数据恢复

Git 会默默地记录每一次改变 HEAD 时它的值。
每一次提交或改变分支，引用日志都会被更新。
可以通过 `git reflog` 找回丢失的提交信息。

如果引用日志信息也丢失

可以通过 `git fsck --full` 显示出所有没有被其他对象指向的对象，从而找到丢失的 commit。

### 移除对象

git clone 会下载整个项目的历史，包括每一个文件的每一个版本。
然而，如果某个人在之前向项目添加了一个大小特别大的文件，即使将这个文件从项目中移除了，每次克隆还是都要强制的下载这个大文件。
之所以会产生这个问题，是因为这个文件在历史中是存在的，它会永远在那里。

**所以，Git 项目执行 CI 时，可以考虑只获取最新的历史，gitlab ci 就提供了这样的功能。**

## 环境变量

Git 总是在一个 bash shell 中运行，并借助一些 shell 环境变量来决定它的运行方式。

具体环境变量参考：[Git 内部原理 - 环境变量](https://git-scm.com/book/zh/v2/Git-%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86-%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F)
