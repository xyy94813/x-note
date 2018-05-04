# Git 工作流

Git 的主流工作流主要分为三种工作流程：

* Git Flow
* Github Flow
* Gitlab Flow

## Git 标准工作流（Git Flow）

Git Flow 中包含两个长期分支

* 主分支（master）
* 开发分支（develop）

主分支用于存放对外发布的版本，任何时候在这个分支拿到的，都是稳定的发布版本

开发分支用于日常开发，存放最新的开发版本

其次，项目存在三种短期分支

* 功能分支（feature branch）
* 补丁分支（hotfix branch）
* 预发布分支（release branch）

一旦完成开发，这三种短期分支会被合并进开发分支或主分支中，然后被删除。

![](https://docs.gitlab.com/ee/workflow/gitdashflow.png)

Git Flow 的优势是清晰可控，但是需要同时维护两个长期分支。

## Github Flow

Github Flow 只有一个长期分支 —— 主分支（master）。

Github 官方[推荐流程](https://guides.github.com/introduction/flow/index.html)图

![](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015122305.png)

1. 从主分支拉取新分支，新分支不区分功能分支或补丁分支；
2. 新分支开发完成或需要讨论时，向主分支发起 pull request；
3. pull request 既是一个通知，让别人注意到你的请求，又是一种对话机制，在这个过程中，还可以不断提交代码；
4. pull request 被接受，合并进主分支，重新部署后，之前拉取的分支被删除

Github Flow 十分的简单，清晰，但是不适应于复杂的发布环境

## Gitlab Flow

Gitlab Flow 是对于 Git Flow 和 Github Flow 的结合，吸取了二者的优点。

[Gitlab Flow](https://docs.gitlab.com/ee/workflow/gitlab_flow.html) 的最大原则是“上游优先（upstream first）”，只存在一个主分支，它是所有分支的“上游”。只有上游分支采纳的代码变化，才能应用到其它分支。

### Gitlab Flow 的 CI/CD

Gitlab Flow 能够支持持续继承和持续发布。
如果你无法控制具体到发布时间，例如，IOS 应用需要等待 App Store 的检查。对于这种情况，Gitlab 建议包含两个长期分支：

* 主分支（master）
* 生产分支（production）

功能分支和补丁分支先合并到主分支中，主分支没有问题再合并到生产分支中。

![](https://docs.gitlab.com/ee/workflow/production_branch.png)

### Gitlab Flow 的环境区分

Gitlab Flow 能够支持按环境区分流程，假设项目存在两个环境 —— 预生产环境和生产环境，对于这种情况 Gitlab Flow 建议根据环境建立两个分支：

* 预生产分支（pre-production）
* 生产分支（production）

主分支（master）是预生产分支的“上游”，预生产分支又是生产分支的“上游”。

![](https://docs.gitlab.com/ee/workflow/environment_branches.png)

所有的开发基于主分支，开发完成后，合并进主分支。没有问题后，主分支再发布到预生产分支。预生产分支也没有问题再从预生产分支发布到生产分支

### Gitlab Flow 版本分支

有些项目需要同时发布并维护多个版本，对于这类型的项目，Gitlab 建议每一个稳定版本都要从主分支中拉取出一个分支。例如：

* 2-3-stable
* 2-4-stable
* ……

![](https://docs.gitlab.com/ee/workflow/release_branches.png)

假如某个版本出现了 bug，首先需要新建一个 hotfix 分支，开发完后合并进主分支，确认没有问题后再`cherry-pick`到出现问题的版本分支中，并且更新小版本号。



