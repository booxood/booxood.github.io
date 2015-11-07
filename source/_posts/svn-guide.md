title: Subversion（SVN）上手
date: 2015-10-15 09:46:05
tags: SVN
categories: Tools
---

很久没用 SVN 了，最基本的命令都忘了，重新熟悉下并记录下来。

- 克隆代码

`svn checkout http://xxx/svn/git-svn-practice/`
一般会生成类似下面这样的目录结构，trunk 类似 git 里的 master 分支， branches、tags 这些目录下的 feature1、v1.0 目录也是分支，不过 svn 里的分支是整个仓库的完整拷贝。

    git-svn-practice
    ├── branches
    │   └── feature1
    │       ├── README.md
    │       └── hello.js
    ├── tags
    │   └── v1.0
    │       ├── README.md
    │       └── hello.js
    └── trunk
        ├── README.md
        └── hello.js


- 添加

`svn add xxx`
添加需要提交到仓库里的文件以及文件夹，已经在仓库的不需要添加，否则会报错。

- 删除

`svn remove xxx`

- 回滚

`svn revert xxx`

- 提交

`svn commit -m " xxx "`
提交就直接提交到 SVN 服务器上了，不像 git 还需要 push。

- 复制

`svn copy xxx xxx`
所谓的分支就是通过 copy 创建的。

- 更新

`svn update`
更新本地代码，会显示更新的结果，结果里第一个字母的含义是

    A  Added
    D  Deleted
    U  Updated
    C  Conflict
    G  Merged
    E  Existed
    R  Replaced

- 比对

`svn diff`
比对当前本地代码和 SVN 记录上最新版本的差异

`svn diff -rXXX`
比对当前本地代码和 SVN 记录上 XXX 版本的差异

`svn diff -rXXX:YYY`
比对 SVN 上 XXX 版本和 YYY 版本的差异



**一些其他的**

1. 默认 `git diff` 是没有颜色的，可以通过修改 `~/.subversion/config` 文件里的配置项 `diff-cmd = colordiff `，并安装工具 `brew install colordiff` 来增加颜色。

2. 如果还是想使用 git ，可以考虑使用 `git svn`， 具体看[这里](https://git-scm.com/book/zh/v1/Git-%E4%B8%8E%E5%85%B6%E4%BB%96%E7%B3%BB%E7%BB%9F-Git-%E4%B8%8E-Subversion)。
