title: 程序后台运行
date: 2014-10-11 12:25
tags: Linux
categories: Linux
---

平时经常需要让程序在后台运行，一般都是在命令结尾加”&“。如果需要在关闭终端也能运行，就在带上nohup，如：nohup ping xxx.com > /dev/null &。

其实还有很多命令可以做到，而且更加灵活，像setsid、disown。特别是disown和jobs配合。这里有篇[文章](http://www.ibm.com/developerworks/cn/linux/l-cn-nohup/)说的很详细。

再补充下 jobs相关的命令：
>	jobs        //查看任务，返回任务编号n和进程号

>	bg  %n   //将编号为n的任务转后台运行

>	fg  %n   //将编号为n的任务转前台运行

>	ctrl+z    //挂起当前任务

>	ctrl+c    //结束当前任务


最后再介绍一个很强大的工具，tmux。之前我一直都是把它当分屏工具时候，其实它也可以做到。
>	tmux ls	//列出挂起的窗口

>	tmux a -t %n	//进入 %n 编号的窗口

>	(tmux)ctrl+b d	//将现在所处的窗口挂起
