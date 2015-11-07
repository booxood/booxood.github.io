title: \r 和 \n 的差异
date: 2013-12-09 16:01:44
categories: Other
---

前段时间在写一个处理文件小程序的时候碰到的一个坑，记录下。

#### 问题

表现如下：


	> console.log('hello\n world');
	hello
	 world
	undefined
	> console.log('hello\r world');
	 world
	undefined



#### 原因

' \r'是回车,使光标到行首

'\n'是换行,光标下移一格

在Unix下，每行结尾只有一个 \n
而 window 下，则是 \n\r

P.S. 一般编程语言会提供一些特殊的变量适应不同操作系统，类似目录分割符那样的，换行符在 Node.js 中是：require('os').EOL 。
