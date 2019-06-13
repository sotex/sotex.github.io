---
layout:  post
title:  "使用Amalgamate将C/C++项目合并成一个.h/.c[pp]文件"
date:  2018-06-01
categories:  编程
tags:  编程
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2018/06/01/9121728.html](http://www.cnblogs.com/oloroso/archive/2018/06/01/9121728.html)
# 简述
C/C++开源库一般是一堆的头文件和源文件，做到声明和实现分离，减小单个模块大小，这在设计上是很好的，但是用起来稍显麻烦。在网上看到有好心人推荐了一个开源工具[Amalgamate](https://github.com/vinniefalco/Amalgamate.git)，专门用来对C/C++的头文件和源文件进行合并用的，于是尝试了一下。
编译过[sqlite源码](http://www.sqlite.org/amalgamation.html)的应该知道，sqlite3源码包有一个是指包含`sqlite3.h`、`sqlite3_ext.h`、`sqlite.c`等为数不多几个代码文件的（也有分开的），嵌入到项目中非常方便。这就是用Amalgamate进行合并的。
# 下载并编译Amalgamate
下载很简单，这里就不细述了
```
git clone https://github.com/vinniefalco/Amalgamate.git
```
编译也很简单，直接使用VS打开`Amalgamate\Builds\VisualStudio2010\Amalgamate.vcxproj`，然后编译生成即可。
最后的示例下载中有我编译的程序。

```bash
# gcc 编译
g++ Amalgamate.cpp juce_core_amalgam.cpp -o Amalgamate -lpthread -ldl
# clang编译
clang++ Amalgamate.cpp juce_core_amalgam.cpp -o Amalgamate -lpthread -ldl
```
具体的使用可以参考程序的帮助信息。

# 将libuv合并为单一头文件和源文件版本(Windows下)

用于合并的模板文件编写可以参考[https://github.com/vinniefalco/Amalgams.git](https://github.com/vinniefalco/Amalgams.git)中的几个。

以`libuv`为例进行简单的说明。
先下载`libuv`的源码，目录结构如下：
![](https://images2018.cnblogs.com/blog/693958/201806/693958-20180601142948616-473242562.jpg)

## 首先先合并头文件
先编写一个头文件`uv_all.h`，里面把`libuv-v1.9.1\include`下的文件都`include`进来。
源码如下：
```c
#include "android-ifaddrs.h"
#include "pthread-barrier.h"
#include "stdint-msvc2008.h"
#include "tree.h"
#include "uv.h"
#include "uv_all.h"
#include "uv-errno.h"
#include "uv-threadpool.h"
#include "uv-version.h"
#include "uv-win.h"
```
一个简单的做法就是cygwin或msys下使用命令`ls *.h |xargs -I{} echo '#include "{}"'`直接输出。
因为我这里只做windows平台的，所以把多余的都给删除了。
实际上因为`uv.h`已经把需要的都包含上了，所以这里直接使用`uv.h`也就够了。
运行下面命令生成合并后的头文件
```bash
Amalgamate.exe -i C:\Users\o\Documents\code\libuv-v1.9.1\include  -w "*.h;*.c"  C:\Users\o\Documents\code\libuv-v1.9.1\include\uv.h  uv.h
```
执行完上面命令后会在当前目录生成一个新的`uv.h`文件，也就是合并后的文件。上面参数中`-i`后面的是附加包含目录，也就是和gcc中使用的`-I`是一样的。最后的`uv.h`是输出文件名，前面的是输入的配置模板文件。

## 合并源码文件
合并源码文件的做法和合并头文件的做法是一致的，先写一个配置文件`uv_win_all.h`（把src和src/win目录下所有文件都包含进来），内容如下：
```c
#include "win/atomicops-inl.h"
#include "win/handle-inl.h"
#include "win/internal.h"
#include "win/req-inl.h"
#include "win/stream-inl.h"
#include "win/winapi.h"
#include "win/winsock.h"

#include "heap-inl.h"
#include "queue.h"
#include "uv-common.h"

#include "win/async.c"
#include "win/core.c"
#include "win/dl.c"
#include "win/error.c"
#include "win/fs.c"
#include "win/fs-event.c"
#include "win/getaddrinfo.c"
#include "win/getnameinfo.c"
#include "win/handle.c"
#include "win/loop-watcher.c"
#include "win/pipe.c"
#include "win/poll.c"
#include "win/process.c"
#include "win/process-stdio.c"
#include "win/req.c"
#include "win/signal.c"
#include "win/snprintf.c"
#include "win/stream.c"
#include "win/tcp.c"
#include "win/thread.c"
#include "win/timer.c"
#include "win/tty.c"
#include "win/udp.c"
#include "win/util.c"
#include "win/winapi.c"
#include "win/winsock.c"

#include "fs-poll.c"
#include "inet.c"
#include "threadpool.c"
#include "uv-common.c"
#include "version.c"
```

然后执行下面命令进行合并
```
Amalgamate.exe -i C:\Users\o\Documents\code\libuv-v1.9.1\include -i C:\Users\o\Documents\code\libuv-v1.9.1\src  -w "*.h;*.c"   C:\Users\o\Documents\code\libuv-v1.9.1\src\uv_win_all.c uv_win.c
```
合并后的文件中会遇到一些问题，需要手动修改一下。比如多出遇到`uv_zero_`重定义的问题，这个需要把第一次定义之后出现的都全部注释掉。
还有会遇到`error LNK2019: 无法解析的外部符号 _InterlockedOr，该符号在函数 _uv_tty_line_read_thread@4 中被引用`的问题，这个只需要使用VS2012之后的版本编译就没问题了。

## 合并后的源码及项目文件
这里不多说，直接放出下载链接 [https://files.cnblogs.com/files/oloroso/libuv_webtest.7z](https://files.cnblogs.com/files/oloroso/libuv_webtest.7z)

测试的代码部分来自于[https://github.com/liigo/tinyweb.git](https://github.com/liigo/tinyweb.git)

# Amalgamate参数简单说明
|参数|解释|
|:---|:---|
|-s|处理`#include <xxx>`的行，即处理包含在系统目录中的头文件（通常我们只需要处理双引号括起来的）|
|-w{wildcards}|指定要处理的文件类型(后缀名)，如果不是列表中指定的，那么即便使用#include包含也不会处理，默认设置是"*.cpp;*.c;*.h;*.mm;*.m"|
|-f  `{file|macro}`|在inlcude出现的所有行中强制重新指定文件或宏|
|-p `{file|macro}`|避免在#include行中的后续出现中重新包含指定的文件或宏|
|-d {name}={file}|如果宏{name}出现在include包含行中，使用{file}替代|
|-i {dir}|在处理include包含时，可以在指定的{dir}目录中搜索文件|
|-v|输出详细信息|