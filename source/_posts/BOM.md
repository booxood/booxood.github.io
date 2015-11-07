title: UTF-8 中的 'efbb'
date: 2013-12-18 17:53
tags: Vim
categories: Other
---
在处理一个 utf-8 格式文件的时候发现，第一行的文件头有 'efbb' 这么些字符，一查才发现原来又是跨平台的问题。[百度 BOM](http://baike.baidu.com/subview/126558/5073180.htm)

要去掉其实也很简单，用 Vi 打开文件，然后输入：

	:set nobomb

要添加或判断是否有BOM：

	#添加BOM
	:set bomb
	#判断是否有BOM
	:set bomb?
