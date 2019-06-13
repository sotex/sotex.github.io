---
layout:  post
title:  "libvlc_new 调用产生段错误"
date:  2017-01-17
categories:  其它
tags:  其它
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2017/01/18/6295205.html](http://www.cnblogs.com/oloroso/archive/2017/01/18/6295205.html)
在调试程序的时候，碰到一个奇怪的段错误问题。只要链接的时候使用`-Wl,-rpath=./vlc/lib`就会产生段错误，如果链接的时候使用的是`-Wl,-rpath=../../tool/vlc/lib`则不会出现。

机器是老旧的`NeoKylin 4.0`版本，没有安装gdb（实际上也不可能去安装）。无法调试，所以在程序里多添加了一些打印输出，终于定位到产生段错误的位置。

代码
![](http://images2015.cnblogs.com/blog/693958/201701/693958-20170117232957640-2060322573.png)
输出
![](http://images2015.cnblogs.com/blog/693958/201701/693958-20170117233143406-1763775341.png)


查了一些资料
[http://www.videolan.org/developers/vlc/doc/doxygen/html/group__libvlc__core.html](http://www.videolan.org/developers/vlc/doc/doxygen/html/group__libvlc__core.html)

这里说 **在POSIX系统上，`SIGCHLD`信号不能被忽略，即信号处理程序必须设置为`SIG_DFL`（默认处理）或函数指针，而不能是`SIG_IGN`（忽略）。**
还有 **LibVLC可以创建线程。 因此，任何`线程不安全`的进程初始化必须在调用`libvlc_new()`之前执行。** 

但是这里没有说到重点，这些都不是产生段错误的原因。
真正的原因是因为vlc存在插件缓存，需要刷新插件缓存才行。
在`vlc/lib/vlc/plugins`下存在一个`plugins.bat`文件，cat这个文件可以发现其中大部分都是普通文本，少数是非文本内容。

通过网络搜索，找到这篇文章，真正的说明了问题 [http://blog.jianchihu.net/libvlc_new-return-null.html](http://blog.jianchihu.net/libvlc_new-return-null.html)

自己编译的vlc，`vlc-cache-gen`程序实际上在`vlc/lib/vlc/vlc-cache-gen`。执行下面命令即可
```shell
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:./vlc/lib
./vlc/lib/vlc/vlc-cache-gen ./vlc/lib/vlc/plugins/
```
然后就没有问题了