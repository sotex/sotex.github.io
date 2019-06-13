---
layout:  post
title:  "git rebase修改历史提交内容"
date:  2018-09-29
categories:  编程
tags:  编程
comments: 1
---
[TOC]
[博客园原文地址 http://www.cnblogs.com/oloroso/archive/2018/09/29/9723783.html](http://www.cnblogs.com/oloroso/archive/2018/09/29/9723783.html)



## 简述
git提交历史中有一次提交的内容是有问题，因为每隔一段时间就要发一次版本，所以必须修改这次提交的内容，以便其不影响已经发布的版本。
大概是这样子的
```
      A --- B ---- C ---- D ---- E  ----- F ------
                  |        \              \
                 有问题      \-----发布      \---- 发布
```
所以这里需要修改C这次提交的内容。

## 解决过程
相关的操作可以参考[7.6 Git 工具 - 重写历史](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E9%87%8D%E5%86%99%E5%8E%86%E5%8F%B2)
这里我创建了一个新的仓库，用来描述解决这个问题的过程。

**1、先看一下提交记录**

```bash
$ git log
commit aa3f6b723abf030b1692f9b573092ec782600d91
Author: solym <solym@sohu.com>
Date:   Sat Sep 29 14:34:36 2018 +0800

    第三次提交

commit e186c75c5431a6eb683d4640ac30c4b8900ba0c1
Author: solym <solym@sohu.com>
Date:   Sat Sep 29 14:34:11 2018 +0800

    第二次提交

commit ebcd3120d30c52125593601f296607c5dcc520a3
Author: solym <solym@sohu.com>
Date:   Sat Sep 29 14:33:48 2018 +0800

    第一次提交
```

这里假设是`第二次提交`的内容有问题，所以需要会到`e186c75c5431a6eb683d4640ac30c4b8900ba0c1`这个提交记录**之前一次**提交的位置来解决这个问题。

**2、先将当前的修改用stash存储一下，后面解决完之后再释放出来**

```bash
$ git stash
Saved working directory and index state WIP on master: aa3f6b7 第三次提交
HEAD is now at aa3f6b7 第三次提交
```

**3、将 HEAD 通过rebase回退到有问题的位置前**

```bash
 git rebase e186c75c5431a6eb683d4640ac30c4b8900ba0c1^ --interactive
warning: stopped at e186c75... 第二次提交
You can amend the commit now, with

  git commit --amend

Once you are satisfied with your changes, run

  git rebase --continue
```

**4、在出来的编辑界面，将有问题的提交前的`pick`改为`edit`，然后保存退出**

![](https://img2018.cnblogs.com/blog/693958/201809/693958-20180929150557120-774661110.jpg)

```
#  p, pick = use commit  使用提交(即保留它，不做修改)
#  r, reword = use commit, but edit the commit message 使用提交，但编辑提交的日志消息
#  e, edit = use commit, but stop for amending 使用提交，但停下来修改（就是要修改提交的内容）
#  s, squash = use commit, but meld into previous commit 使用提交，但融入此前的提交（就是与在此之前一个提交合并）
#  f, fixup = like "squash", but discard this commit's log message 类似于squash，但是丢弃此提交的日志消息
#  x, exec = run command (the rest of the line) using shell 运行shell命令
# d,drop = remove commit 删除提交
```

**5、修改有问题的文件，解决后重新提交。注意提交使用的参数是`--amend`**

```bash
$ vim A.cpp
$ git add A.cpp
$ git commit --amend
[detached HEAD 8b4daa5] 第二次提交
 Date: Sat Sep 29 14:34:11 2018 +0800
 2 files changed, 8 insertions(+), 1 deletion(-)
 create mode 100644 B.cpp
```

**6、使用`git rebase --continue`逐步前进到最新的提交位置。**

```bash
$ git rebase --continue
error: could not apply aa3f6b7... 第三次提交

When you have resolved this problem, run "git rebase --continue".
If you prefer to skip this patch, run "git rebase --skip" instead.
To check out the original branch and stop rebasing, run "git rebase --abort".

Could not apply aa3f6b7... 第三次提交
Auto-merging A.cpp
CONFLICT (content): Merge conflict in A.cpp
```
上面执行后因为有两处都有修改，需要解决冲突。
```bash
$ git status
interactive rebase in progress; onto ebcd312
Last commands done (2 commands done):
   edit e186c75 第二次提交
   pick aa3f6b7 第三次提交
No commands remaining.
You are currently rebasing branch 'master' on 'ebcd312'.
  (fix conflicts and then run "git rebase --continue")
  (use "git rebase --skip" to skip this patch)
  (use "git rebase --abort" to check out the original branch)

Unmerged paths:
  (use "git reset HEAD <file>..." to unstage)
  (use "git add <file>..." to mark resolution)

        both modified:   A.cpp

no changes added to commit (use "git add" and/or "git commit -a")
```
修改后再次提交即可
```bash
$ vim A.cpp
$ git add A.cpp
$ git commit -a
[detached HEAD 8070ac2] 第三次提交
 1 file changed, 6 insertions(+), 1 deletion(-)
```
然后重新执行
```
$ git rebase --continue
Successfully rebased and updated refs/heads/master.
```
如果还有冲突，则重复执行上面两步骤。

**7、最后将stash存储的内容释放出来，继续工作**

```bash
$ git stash pop
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   B.cpp

no changes added to commit (use "git add" and/or "git commit -a")
Dropped refs/stash@{0} (fc32de3118386c30047df86670371c8ab049e0e0)

```