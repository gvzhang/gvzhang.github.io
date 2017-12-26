---
title: git协同开发
categories:
 - 开发工具
tags:
 - git
 - 协同开发
---

### 简介

![Linus](https://zgjian-pic.oss-cn-beijing.aliyuncs.com/markdown/LinuxCon_Europe_Linus_Torvalds_03.jpg)
林纳斯（Linus）·本纳第克特·托瓦兹

同生活中的许多伟大事物一样，Git 诞生于一个极富纷争大举创新的年代。

Linux 内核开源项目有着为数众广的参与者。绝大多数的 Linux 内核维护工作都花在了提交补丁和保存归档的繁琐事务上（1991－2002年间）。到 2002 年，整个项目组开始启用一个专有的分布式版本控制系统 Bit Keeper来管理和维护代码。

到了 2005 年，开发 Bit Keeper 的商业公司同 Linux 内核开源社区的合作关系结束，他们收回了 Linux 内核社区免费使用 Bit Keeper 的权力。这就迫使 Linux 开源社区（特别是 Linux 的缔造者 Linux Torvalds）基于使用Bit Kcheper 时的经验教训，开发出自己的版本系统。他们对新的系统制订了若干目标：
- 速度
- 简单的设计
- 对非线性开发模式的强力支持（允许成千上万个并行开发的分支）
- 完全分布式
- 有能力高效管理类似 Linux 内核一样的超大规模项目（速度和数据量）

自诞生于 2005 年以来，Git 日臻成熟完善，在高度易用的同时，仍然保留着初期设定的目标。它的速度飞快，极其适合管理大项目，有着令人难以置信的非线性分支管理系统。

整个Windows都从sd搬到git，创造了全球最大的Git储存库，内含容量高达300GB的350万个档案，同时宣布将GVFS释出成为开源专案。

### 版本控制系统
##### 本地版本控制系统
![本地版本控制系统](https://zgjian-pic.oss-cn-beijing.aliyuncs.com/markdown/local-version-control.png)

##### 集中化版本控制系统
![集中化版本控制系统](https://zgjian-pic.oss-cn-beijing.aliyuncs.com/markdown/svn-version-control.png)
（权限、联网、灾难）

##### 分布式版本控制系统
![分布式版本控制系统](https://zgjian-pic.oss-cn-beijing.aliyuncs.com/markdown/git-version-control.png)

### 常用命令
#### 新建代码库
```bash
# 在当前目录新建一个Git代码库
$ git init

# 新建一个目录，将其初始化为Git代码库
$ git init [project-name]

# 下载一个项目和它的整个代码历史
$ git clone [url]
```

#### 配置
```bash
# 显示当前的Git配置
$ git config --list

# 编辑Git配置文件
$ git config -e [--global]

# 设置提交代码时的用户信息
$ git config [--global] user.name "[name]"
$ git config [--global] user.email "[email address]"
```

#### 增加/删除文件
```bash
# 添加指定文件到暂存区
$ git add [file1] [file2] ...

# 添加指定目录到暂存区，包括子目录
$ git add [dir]

# 添加当前目录的所有文件到暂存区
$ git add .

# 添加每个变化前，都会要求确认
# 对于同一个文件的多处变化，可以实现分次提交
$ git add -p

# 删除工作区文件，并且将这次删除放入暂存区
$ git rm [file1] [file2] ...

# 停止追踪指定文件，但该文件会保留在工作区
$ git rm --cached [file]

# 改名文件，并且将这个改名放入暂存区
$ git mv [file-original] [file-renamed]
```

#### 代码提交
```bash
# 提交暂存区到仓库区
$ git commit -m [message]

# 提交暂存区的指定文件到仓库区
$ git commit [file1] [file2] ... -m [message]

# 提交工作区自上次commit之后的变化，直接到仓库区
$ git commit -a

# 提交时显示所有diff信息
$ git commit -v

# 使用一次新的commit，替代上一次提交
# 如果代码没有任何新变化，则用来改写上一次commit的提交信息
$ git commit --amend -m [message]

# 重做上一次commit，并包括指定文件的新变化
$ git commit --amend [file1] [file2] ...
```

#### 分支
```bash
# 列出所有本地分支
$ git branch

# 列出所有远程分支
$ git branch -r

# 列出所有本地分支和远程分支
$ git branch -a

# 新建一个分支，但依然停留在当前分支
$ git branch [branch-name]

# 新建一个分支，并切换到该分支
$ git checkout -b [branch]

# 新建一个分支，指向指定commit
$ git branch [branch] [commit]

# 新建一个分支，与指定的远程分支建立追踪关系
$ git branch --track [branch] [remote-branch]

# 切换到指定分支，并更新工作区
$ git checkout [branch-name]

# 切换到上一个分支
$ git checkout -

# 建立追踪关系，在现有分支与指定的远程分支之间
$ git branch --set-upstream [branch] [remote-branch]

# 合并指定分支到当前分支
$ git merge [branch]

# 选择一个commit，合并进当前分支
$ git cherry-pick [commit]

# 删除分支
$ git branch -d [branch-name]

# 删除远程分支
$ git push origin --delete [branch-name]
$ git branch -dr [remote/branch]
```

#### 查看信息
```bash
# 显示有变更的文件
$ git status

# 显示当前分支的版本历史
$ git log

# 显示commit历史，以及每次commit发生变更的文件
$ git log --stat

# 搜索提交历史，根据关键词
$ git log -S [keyword]

# 显示某个commit之后的所有变动，每个commit占据一行
$ git log [tag] HEAD --pretty=format:%s

# 显示某个commit之后的所有变动，其"提交说明"必须符合搜索条件
$ git log [tag] HEAD --grep feature

# 显示某个文件的版本历史，包括文件改名
$ git log --follow [file]
$ git whatchanged [file]

# 显示指定文件相关的每一次diff
$ git log -p [file]

# 显示过去5次提交
$ git log -5 --pretty --oneline

# 显示所有提交过的用户，按提交次数排序
$ git shortlog -sn

# 显示指定文件是什么人在什么时间修改过
$ git blame [file]

# 显示暂存区和工作区的差异
$ git diff

# 显示暂存区和上一个commit的差异
$ git diff --cached [file]

# 显示工作区与当前分支最新commit之间的差异
$ git diff HEAD

# 显示两次提交之间的差异
$ git diff [first-branch]...[second-branch]

# 显示今天你写了多少行代码
$ git diff --shortstat "@{0 day ago}"

# 显示某次提交的元数据和内容变化
$ git show [commit]

# 显示某次提交发生变化的文件
$ git show --name-only [commit]

# 显示某次提交时，某个文件的内容
$ git show [commit]:[filename]

# 显示当前分支的最近几次提交
$ git reflog
```

#### 远程同步
```bash
# 下载远程仓库的所有变动
$ git fetch [remote]

# 显示所有远程仓库
$ git remote -v

# 显示某个远程仓库的信息
$ git remote show [remote]

# 增加一个新的远程仓库，并命名
$ git remote add [shortname] [url]

# 取回远程仓库的变化，并与本地分支合并
$ git pull [remote] [branch]

# 上传本地指定分支到远程仓库
$ git push [remote] [branch]

# 强行推送当前分支到远程仓库，即使有冲突
$ git push [remote] --force

# 推送所有分支到远程仓库
$ git push [remote] --all
```

#### 撤销
```bash
# 恢复暂存区的指定文件到工作区
$ git checkout [file]

# 恢复某个commit的指定文件到暂存区和工作区
$ git checkout [commit] [file]

# 恢复暂存区的所有文件到工作区
$ git checkout .

# 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
$ git reset [file]

# 重置暂存区与工作区，与上一次commit保持一致
$ git reset --hard

# 重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变
$ git reset [commit]

# 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致
$ git reset --hard [commit]

# 重置当前HEAD为指定commit，但保持暂存区和工作区不变
$ git reset --keep [commit]

# 新建一个commit，用来撤销指定commit
# 后者的所有变化都将被前者抵消，并且应用到当前分支
$ git revert [commit]
```

### 技巧
#### 交互式暂存
在开发完成一个版本后，我们会提交一分干净的代码到分支上，然后新建一个分支，去开发新的功能。

但是，天有不测风云，在新功能做了一部分的时候，还没开发完成，Boss告知，上一个版本有个问题，需要修改。此时，新功能还未开发完成，不想提交。那怎么办？

```bash
# 暂时将未提交的变化移除，稍后再移入
$ git stash
$ git stash pop
```

#### 打标签
Git 也可以对某一时间点上的版本打上标签。人们在发布某个软件版本（比如 v1.0 等等）的时候，经常这么做。

##### 轻量级的(lightweight)
实际上它就是个指向特定提交对象的引用。

##### 含附注标签
实际上是存储在仓库中的一个独立对象，它有自身的校验和信息，包含着标签的名字，电子邮件地址和日期，以及标签说明，标签本身也允许使用GNU Privacy Guard(GPG)来签署或验证。

一般我们都建议使用含附注型的标签，以便保留相关信息；当然，如果只是临时性加注标签，或者不需要旁注额外信息，用轻量级标签也没问题。

#### 标签
##### GNU 风格的版本号管理策略
1. 项目初版本时 , 版本号可以为 0.1 或 0.1.0, 也可以为 1.0 或 1.0.0, 如果你为人很低调 , 我想你会选择那个主版本号为 0 的方式 
2. 当项目在进行了局部修改或 bug 修正时 , 主版本号和子版本号都不变 , 修正版本号加 1
3. 当项目在原有的基础上增加了部分功能时 , 主版本号不变 , 子版本号加 1, 修正版本号复位为 0, 因而可以被忽略掉 
4. 当项目在进行了重大修改或局部修正累积较多 , 而导致项目整体发生全局变化时 , 主版本号加 1
```bash
# 列出所有tag
$ git tag

# 新建一个tag在当前commit
$ git tag [tag]

# 新建一个tag在指定commit
$ git tag [tag] [commit]

# 删除本地tag
$ git tag -d [tag]

# 删除远程tag
$ git push origin :refs/tags/[tagName]

# 查看tag信息
$ git show [tag]

# 提交指定tag
$ git push [remote] [tag]

# 提交所有tag
$ git push [remote] --tags

# 新建一个分支，指向某个tag
$ git checkout -b [branch] [tag]
```

#### 搜索
Git 提供了一个 grep 命令，你可以很方便地从提交历史或者工作目录中查找一个字符串或者正则表达式。 
```bash
# 会自动map所有包含foo的文件
$ git grep foo

# 显示行号
$ git grep -n foo

# 只显示文件名
$ git grep --name-only foo

# 可以查看每个文件里有多个匹配内容
$ git grep -c foo
```

日志查找
```bash
# 仅显示最近的 n 条提交
$ git log -(n)

# 仅显示指定时间之后的提交。
$ git log --since, --after

# 仅显示指定时间之前的提交。
$ git log --until, --before

# 仅显示指定作者相关的提交。
$ git log --author

# 仅显示指定提交者相关的提交。
$ git log --committer

# 仅显示含指定关键字的提交
$ git log --grep

# 仅显示添加或移除了某个关键字的提交
$ git log -S
```

#### 重写历史
```bash
$ git commit --amend
```
如果你完成提交后又想修改被提交的快照，增加或者修改其中的文件，可能因为你最初提交时，忘了添加一个新建的文件，这个过程基本上一样。你通过修改文件然后对其运行git add或对一个已被记录的文件运行git rm，随后的git commit --amend会获取你当前的暂存区并将它作为新提交对应的快照。

使用这项技术的时候你必须小心，因为修正会改变提交的SHA-1值。这个很像是一次非常小的rebase——不要在你最近一次提交被推送后还去修正它。

### Git Flow
Git 最强大之处在于它的分支管理，在本地你可以随意拉取分支，只要你喜欢，而且拉取的速度很快，它实际只是修改了一下指针。SVN 虽然也有分支管理，但它创建一条分支却是先在远程上创建，然后你才能 checkout 到本地，实际就是复制了一份一模一样的代码，项目大的话，checkout 也得花点时间。

实际上，Git 有几条分支在自己的电脑上都是保留着一个目录的代码，随着你切换分支，目录里的代码会相应进行改变。SVN 则是你有几条分支则本地电脑保存着几份项目的代码，偶尔可能找错代码。

利用git的分支，可以非常方便地进行开发和测试，如果使用git没有让你感到轻松和愉悦，那是因为你还没有学会使用分支。**不把分支用出一点翔来，不要轻易跟别人说你用过git。**

#### 主分支Master
首先，代码库应该有一个、且仅有一个主分支。所有提供给用户使用的正式版本，都在这个主分支上发布。

#### 开发分支Develop
主分支只用来分布重大版本，日常开发应该在另一条分支上完成。我们把开发用的分支，叫做Develop。

#### 临时性分支
前面讲到版本库的两条主要分支：Master和Develop。前者用于正式发布，后者用于日常开发。其实，常设分支只需要这两条就够了，不需要其他了。

但是，除了常设分支以外，还有一些临时性分支，用于应对一些特定目的的版本开发。临时性分支主要有三种：
- 功能（feature）分支
- 预发布（release）分支
- 修补bug（fixbug）分支
这三种分支都属于临时性需要，使用完以后，应该删除，使得代码库的常设分支始终只有Master和Develop。

#### 功能分支
接下来，一个个来看这三种"临时性分支"。
第一种是功能分支，它是为了开发某种特定功能，从Develop分支上面分出来的。开发完成后，要再并入Develop。

#### 预发布分支
第二种是预发布分支（与预发布环境关联），它是指发布正式版本之前（即合并到Master分支之前），我们可能需要有一个预发布的版本进行测试。
预发布分支是从Develop分支上面分出来的，预发布结束以后，必须合并进Develop和Master分支。它的命名，可以采用release-*的形式。

#### 修补bug分支
最后一种是修补bug分支。软件正式发布以后，难免会出现bug。这时就需要创建一个分支，进行bug修补。
修补bug分支是从Master分支上面分出来的。修补结束以后，再合并进Master和Develop分支。它的命名，可以采用fixbug-*的形式。

![分支管理](https://zgjian-pic.oss-cn-beijing.aliyuncs.com/markdown/git-model@2x.png)

#### 优缺点
Git flow的优点是清晰可控，缺点是相对复杂，需要同时维护两个长期分支。大多数工具都将master当作默认分支，可是开发是在develop分支进行的，这导致经常要切换分支，非常烦人。

更大问题在于，这个模式是基于"版本发布"的，目标是一段时间以后产出一个新版本。但是，很多网站项目是"持续发布"，代码一有变动，就部署一次。这时，master分支和develop分支的差别不大，没必要维护两个长期分支。

### Github Flow
![Github Flow](https://zgjian-pic.oss-cn-beijing.aliyuncs.com/markdown/bg2015122305.jpg)
#### 优缺点
Github flow 的最大优点就是简单，对于"持续发布"的产品，可以说是最合适的流程。

问题在于它的假设：master分支的更新与产品的发布是一致的。也就是说，master分支的最新代码，默认就是当前的线上代码。

可是，有些时候并非如此，代码合并进入master分支，并不代表它就能立刻发布。比如，苹果商店的APP提交审核以后，等一段时间才能上架。这时，如果还有新的代码提交，master分支就会与刚发布的版本不一致。另一个例子是，有些公司有发布窗口，只有指定时间才能发布，这也会导致线上版本落后于master分支。

上面这种情况，只有master一个主分支就不够用了。通常，你不得不在master分支以外，另外新建一个production分支跟踪线上版本。

### 扩展
#### 钩子

### 资源
- Pro Git
- [使用git和github进行协同开发流程](https://github.com/livoras/blog/issues/7)
- [Git分支管理策略](http://www.ruanyifeng.com/blog/2012/07/git.html)
- [Git分支管理最佳实践](https://www.ibm.com/developerworks/cn/java/j-lo-git-mange/)
