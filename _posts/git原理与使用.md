---
title: git原理与使用
categories:
  - 笔记
tags:
  - git
date: 2018-08-18 09:46:59
---
 这是摘要
 <!-- more -->

#git
## git使用
### 提交
1. 合并add和commit
`git commit -am "message"`
以上命令可以直接将tracked状态的文件提交
2. 追加提交
`git commit --amend`
3. commit和merge技巧
建议一个commit只修改一个问题,这样如果想在多个分支之间同步一个修改,
只需要用`git cherry-pick commitId`就可以完成
4. 使用https clone项目之后每次pull,push代码都要输入密码的解决办法(好像没用)
`git config --global credential.helper store`

### 回退
1. 回退单个文件
`git reset head filename`
以上命令可以将暂存区的文件回退到工作区,也就是将add过的文件变回add之前的状态
`git reset head`
这个命令就是将所有暂存区的文件回退到工作区
2. 回退版本
`git reset --hard head^`
以上命令可以将代码回退到当前最新的commit
`git reset --hard commit_id`
将代码回退到指定commit
3. 撤销commit
`git revert head`
还原已经提交的修改,是提交一个新的版本，将需要revert的版本的内容再反向修改回去，版本会递增，不影响之前提交的内容。
此次操作之前和之后的commit和history都会保留，并且把这次撤销作为一次最新的提交 
`git revert HEAD` 
撤销前一次 commit 
`git revert HEAD^` 
撤销前前一次 commit 
`git revert commit-id` 
撤销指定的版本，撤销也会作为一次提交进行保存

### 撤销操作
1. `git revert <SHA>`
场景: 你已经执行了 git push, 把你的修改发送到了 GitHub，现在你意识到这些 commit 的其中一个是有问题的，你需要撤销那一个 commit.
原理: git revert 会产生一个新的 commit，它和指定 SHA 对应的 commit 是相反的（或者说是反转的）。任何从原先的 commit 里删除的内容会在新的 commit 里被加回去，任何在原先的 commit 里加入的内容会在新的 commit  里被删除。此操作是安全的,因为产生了新的commit,历史记录还在
2. `git commit --amend`
场景: 你在最后一条 commit 消息里有个笔误，已经执行了 git commit -m "Fxies bug #42"，但在 git push 之前你意识到消息应该是 “Fixes bug #42″
原理: git commit --amend 会用一个新的 commit 更新并替换最近的 commit ，这个新的 commit 会把任何修改内容和上一个 commit 的内容结合起来。如果当前没有提出任何修改，这个操作就只会把上次的 commit 消息重写一遍。
3. `git checkout -- <bad filename>`
场景: 一只猫从键盘上走过，无意中保存了修改，然后破坏了编辑器。不过，你还没有 commit 这些修改。你想要恢复被修改文件里的所有内容 — 就像上次 commit 的时候一模一样。
原理: git checkout 会把工作目录里的文件修改到 Git 之前记录的某个状态。你可以提供一个你想返回的分支名或特定 SHA ，或者在缺省情况下，Git 会认为你希望 checkout 的是 HEAD，当前 checkout 分支的最后一次 commit。
记住：你用这种方法“撤销”的任何修改真的会完全消失。因为它们从来没有被提交过，所以之后 Git 也无法帮助我们恢复它们。你要确保自己了解你在这个操作里扔掉的东西是什么！（也许可以先利用 git diff 确认一下）
4. `git reset <last good SHA> 或 git reset --hard <last good SHA>`
场景: 你在本地提交了一些东西（还没有 push），但是所有这些东西都很糟糕，你希望撤销前面的三次提交 — 就像它们从来没有发生过一样。
原理: git reset 会把你的代码库历史返回到指定的 SHA 状态。 这样就像是这些提交从来没有发生过。
	1. --soft,工作区和暂存区都不动,只是撤销了commit
	Does not touch the index file or the working tree at all.
	2. 缺省情况下(--mixed)， **git reset 会保留工作目录。**这样，提交是没有了，但是修改内容还在磁盘上。这是一种安全的选择，
	Resets the index but not the working tree.
	3. 但通常我们会希望一步就“撤销”提交以及修改内容 ,这就是 --hard 选项的功能。
	Resets the index and working tree. 
5. 利用分支的另一种做法
场景: 你进行了一些提交，然后意识到你开始 check out 的是 master 分支。你希望这些提交进到另一个特性（feature）分支里。
方法: `git branch feature, git reset --hard origin/master, and git checkout feature`
原理: 你可能习惯了用 `git checkout -b <name> `创建新的分支 — 这是创建新分支并马上 check out 的流行捷径 — 但是你不希望马上切换分支。这里，` git branch feature` 创建一个叫做 feature 的新分支并指向你最近的 commit，但还是让你 check out 在 master 分支上。
下一步，在提交任何新的 commit 之前，用 `git reset --hard` 把 master 分支倒回 origin/master 。不过别担心，那些 commit 还在 feature 分支里。
最后，用 git checkout 切换到新的 feature 分支，并且让你最近所有的工作成果都完好无损。
6. `git rm --cached application.log`
场景: 你偶然把 application.log 加到代码库里了，现在每次你运行应用，Git 都会报告在 application.log 里有未提交的修改。你把 `*.login`放到了 .gitignore 文件里，可文件还是在代码库里 — 你怎么才能告诉 Git “撤销” 对这个文件的追踪呢？
原理: 虽然 .gitignore 会阻止 Git 追踪文件的修改，甚至不关注文件是否存在，但这只是针对那些以前从来没有追踪过的文件。一旦有个文件被加入并提交了，Git 就会持续关注该文件的改变。类似地，如果你利用` git add -f` 来强制或覆盖了 .gitignore， Git 还会持续追踪改变的情况。之后你就不必用-f  来添加这个文件了。
如果你希望从 Git 的追踪对象中删除那个本应忽略的文件， `git rm --cached `会从追踪对象中删除它，但让文件在磁盘上保持原封不动。因为现在它已经被忽略了，你在  git status 里就不会再看见这个文件，也不会再偶然提交该文件的修改了。

### 分支管理
1. 查看当前分支基于那个分支

2. 将主线代码合并到自己的分支
    1. 切换到master分支
      　　`git checkout master`
    2. 将remote master同步到local master
      　　`git pull origin master`
    3. 切换到的local开发分支
      　　`git checkout dev-mine`
    4. 合并 local master 到 local的开发分支
         `git merge master`
    5. 推送更新到gitlab，使gitlab同步更新显示
       　 `git push origin dev-minw`
3. 创建分支
    1. 基于远程分支创建本地分支
    `git checkout -b dev-local origin/dev-remote`
    这样创建的分支是有关联的,然后基于这个分支再拉一个分支做开发,开发完之后push到远程,发起pull request(github)或者merge request(gitlab)
    2. 建立本地分支和远程分支的关联
    `git branch --set-upstream-to origin/dev-remote`
    2. 基于当前分支创建分支(假设是master)
    `git checkout -b dev-basedon-master`
    或者是
    `git checkout -b dev-basedon-master master`
    3. 重命名本地分支
    `git branch -m dev-old dev-new`
    4. 删除远程分支
    `git push origin :dev-remote`
    5. 查看分支
    `git branch`
    查看本地分支
    `git branch -r`
    查看远程分支
    `git branch -vv`
    查看本地分支和远程分支的关联
    6. 删除本地分支
    `git branch -d branch-name`
    -D是强制删除

### 暂时保存工作进度
`[]`方括号中内容为可选，`[<stash>]`里面的stash代表进度的编号形如：`stash@{0}`, `<>`尖括号内的必填
`git stash`  对当前的暂存区和工作区状态进行保存。 
`git stash save "work about something on branch-xxx"`最好写个记录
`git stash list`  列出所有保存的进度列表。 
`git stash pop [--index] [<stash>]` 恢复工作进度
如：以下命令恢复编号为0的进度的工作区和暂存区
`git stash pop --index stash@{0}`
`git stash apply [--index] [<stash>]` 不删除已恢复的进度，其他同git stash pop 
`git stash drop [<stash>]` 删除某一个进度，默认删除最新进度 
`git stash clear` 删除所有进度 
`git stash branch <branchname> <stash>` 基于进度创建分支

### 查看记录
1. 查看某次commit的所有文件改动内容
`git show commit_id` 
2. 查看历史commit
`git log [--pretty=oneline]`

## git原理
参考地址:https://www.jianshu.com/p/619122f8747b
从根本上来讲，Git是一个内容寻址文件系统,Git 和其他版本控制系统的主要差别在于，Git 只关心文件数据的整体是否发生变化，而大多数其他系统则只关心文件内容的具体差异。这类系统（CVS，Subversion，Perforce，Bazaar 等等）每次记录有哪些文件作了更新，以及都更新了哪些行的什么内容：
{% asset_img 其他版本系统.png 其他版本系统%}
Git 并不保存这些前后变化的差异数据。实际上，Git 更像是把变化的文件作快照后，记录在一个微型的文件系统中。每次提交更新时，它会纵览一遍所有文件的指纹信息并对文件作一快照，然后保存一个指向这次快照的索引。为提高性能，若文件没有变化，Git 不会再次保存，而只对上次保存的快照作一链接。Git 的工作方式就如下图所示。
{% asset_img git版本系统.png git版本系统%}

### 工作区和版本库
{% asset_img 工作区和版本库.png 工作区和版本库%}
1. 修改了工作区的文件
2. 对修改的文件进行快照,然后存到版本库的暂存区(index目录)
3. 提交更新后,将暂存区的文件快照永久保存到objects目录

>参考地址:https://juejin.im/post/5af26c4d5188256728605809

{% asset_img git工作区域.png %}
{% asset_img git文件生命周期.png %}


### `.git`目录结构
{% asset_img 目录结构.png 目录结构%}
description文件：仅供GitWeb程序使用
config文件：包含项目特有的配置选项
info目录：包含一个全局性排除(global exclude)文件，用以放置那些不希望被记录在 .gitignore文件中的忽略模式(ignored patterns)
hooks目录：包含客户端或服务端的钩子脚本(hook scripts)
HEAD文件：指示目前被检出的分支
index文件：保存暂存区信息
objects目录：存储所有数据内容
refs 目录：存储指向数据（分支）的提交对象的指针

### merge的过程
{%asset_img merge的原理.png merge%}
想要merge master分支和other分支,会将master分支当前节点,other当前节点,以及两个分支的公共祖先节点,三者进行比较合并

### git底层数据结构
>参考地址:https://www.jianshu.com/p/fa31ef8814d2

1. Blob
Git 把版本库中的每一个文件都转换为一个 blob 对象进行存储，
2. Tree
而用 tree 对象来表达文件的层次结构。
3. Commit
Commit 对象代表了一次提交操作，它包含了当前的项目快照以及提交人和提交日期等诸多信息。所有的 commit 对象串接起来，组成一个有向无环图。从版本控制的角度看，这些 commit 对象构成了一个完整的版本提交记录；从项目开发的角度看，它们描述了项目是如何从无到有一点一滴地构建起来的。
4. Tag
Tag 对象指向一个 commit 对象，我们可以通过 tag 对象快速访问到项目的某一次特征提交。
























