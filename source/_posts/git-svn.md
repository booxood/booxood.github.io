title: Git 与 Subversion（SVN）
date: 2015-10-28 15:57:00
tags: [Git, SVN]
categories: Tools
---
Git 提供了一个 `git svn` 命令，可以使用 Git 来操作 SVN 仓库，具体介绍可以[看这里](https://git-scm.com/book/zh/v1/Git-%E4%B8%8E%E5%85%B6%E4%BB%96%E7%B3%BB%E7%BB%9F-Git-%E4%B8%8E-Subversion)。

### 基本用法

- 克隆仓库
    `git svn clone http://xxx -T trunk -b branches -t tags`

        -T trunk -b branches -t tags 告诉 Git 该 Subversion 仓库遵循了基本的分支和标签命名法则。
        如果你的主干(译注：trunk，相当于非分布式版本控制里的master分支，代表开发的主线），分支或者标签
        以不同的方式命名，则应做出相应改变。由于该法则的常见性，可以使用 -s 来代替整条命令，它意味着标准
        布局（s 是 Standard layout 的首字母），也就是前面选项的内容。

- 提交代码
    1. `git commit`
    直接使用 Git 命令将代码提交到 Git 本地仓库。
    2. `git svn dcommit`
    将 Git 本地仓库的新增记录提交到 SVN 服务器上。提交之后，之前 Git 上的提交记录

            commit 8907ffe03a2879091a35a869d9bc1ef480af967c
            Author: xxx <xxx@gmail.com>
            Date:   Wed Oct 28 16:26:08 2015 +0800

            Add xxx from git

    会被改成

            commit 3b707ba0daf0eee9791fc6694e6b4079960c5f3a
            Author: xxx <xxx@9fb60aa9-af34-4021-9c2d-4e8c7c99b066>
            Date:   Wed Oct 28 08:20:37 2015 +0000

                Add xxx from git

                git-svn-id: http://xxx/trunk@12 9fb60aa9-af34-4021-9c2d-4e8c7c99b066

    至于为什么会这样，引用官方的说明

            所有在原 Subversion 数据基础上提交的 commit 会一一提交到 Subversion，然后你本地 Git
            的 commit 将被重写，加入一个特别标识。这一步很重要，因为它意味着所有 commit 的 SHA-1
            指都会发生变化。这也是同时使用 Git 和 Subversion 两种服务作为远程服务不是个好主意的原因
            之一。检视以下最后一个 commit，你会找到新添加的 git-svn-id

    当你本地的代码比服务器上旧的话，如果可以 `Merge`，则直接 `Merge` 然后再提交，如果不能 `Merge`，有冲突，需要 `git rebase` 解决冲突后再 `git rebase --continue` 提交。

- 更新代码
    `git svn rebase`
    需要本地 Git 整洁才可以更新。如果本地有代码没有提交，会报错，可以先 `git stash` 保存当前的改动，然后再更新 `git svn rebase`，最后再还原之前的改动 `git stash pop`。

- 创建分支
    `git svn branch [xxx]`
    **在我本地尝试时，报错 `Authorization failed: Unable to connect to a repository at URL ...`**
    我都可以提交，为什么不能创建分支呢。。。如果有人知道，求告知！！！

- 使用 SVN 命令
    虽然已经完全是一个 Git 项目了，但仍可以使用一些 SVN 的命令。
    这些命令都是可以离线运行的，所以，看到的就是最后一次通讯所得到的信息。

    - `git svn log` 查看日志
    - `git svn blame` 查看文件的改动记录
    - `git svn info` 查看 SVN 仓库信息

### 注意事项

- 略 Subversion 之所略
    如果 SVN 项目包含 `svn:ignore` 属性，则需要创建 `.gitignore` 来保持一致，防止提交一些不应该提交的文件。
    使用 `git svn show-ignore` 可以看到 SVN 设置的忽略文件，把它们放到 `.gitignore` 或者 `.git/info/exclude`。

- 保持 Git 主干上的提交记录是线性的
    因为 SVN 只能保存一条线性的提交记录。
