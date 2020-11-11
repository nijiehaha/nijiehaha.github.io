---
title: Git 压缩提交
date: 2020-06-10 10:25:30
tags: 杂七杂八
categories: 杂七杂八
---

# 前言

对于有点强迫症的我来说，Git 分支上过多冗余的 commit 记录是我心中的痛，于是「压缩提交」就成了一个必须要解决的问题。

通过各种搜索，查资料，总结下来，大概下面几种办法。

# 方法一

删掉 `.git` 文件夹，重新 `git init`，从零开始，吧啦吧啦。。。

简单粗暴～

如果项目只有一个分支的话，可以这么做，如果还有其他的，建议不要这样做，因为这样就把其他分支的 commit 记录 都删了

# 方法二

使用 `git rebase -i`，合并 commit 。。。

优雅～

# 方法三

使用 `git reset --soft`, 吧啦吧啦。。。

优雅～

# 方法四

假设我们要压缩的是 master 上的 提交记录

+ 使用 `git checkout --orphan new_branch` , 基于当前分支创建一个独立的分支 new_branch

+ 添加所有文件变化至暂存空间, `git add -A`

+ 提交并添加提交记录, `git commit -m 'commit message'`

+ 删除要覆盖的分支 master, `git branch -D master`

+ 重新命名当前独立分支为 master, `git branch -m master`

# 覆盖远程分支提交记录

使用 `git push -f` 强制覆盖

> 这个 `-f` 指令仅适用个人分支，团队协作的分支，要谨慎使用，因为会强制把远程仓库的 HEAD 指向你个人本地的 HEAD，覆盖掉同时进行的其他协作者的代码，不推荐使用！

# 如何在当前分支 git rebase 第一个 commit

> git rebase -i --root

[How do I git rebase the first commit?](https://stackoverflow.com/questions/22992543/how-do-i-git-rebase-the-first-commit/23000315)

# 参考

[Git 文档](https://git-scm.com/book/zh/v2)

[如何清空所有的commit记录](https://juejin.im/post/5be995c25188250fa8358f9d)

[如何优雅地合并多个 Commit](https://github.com/Jisuanke/tech-exp/issues/13)

