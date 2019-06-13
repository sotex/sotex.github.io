---
layout:  post
title:  "MSVC下使用Boost的自动链接"
date:  2018-03-15
categories:  编程
tags:  编程
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2018/03/15/8574936.html](http://www.cnblogs.com/oloroso/archive/2018/03/15/8574936.html)
## 简述
好久没有用过boost库了，以前用也是在linux下，需要哪个部分就添加哪个部分到Makefile中。
最近要在Windows下使用，主要是mongocxx库依赖它，不想自己去编译它了，就直接在[https://dl.bintray.com/boostorg/release/1.66.0/binaries/](https://dl.bintray.com/boostorg/release/1.66.0/binaries/)上下载已经编译好的版本。
下载安装后发现一个问题，就是它的`lib`目录下存在多个不同编译参数编译的版本，在VC中它会自己根据当前环境选择对应的库进行链接（通过`#pragma comment(lib,"库路径"`指令实现）。而我需要使用指定的版本。

关于boost在windows下编译出的库文件的命令方式可以在这里查到[http://www.boost.org/doc/libs/1_66_0/more/getting_started/windows.html](http://www.boost.org/doc/libs/1_66_0/more/getting_started/windows.html)。
我就不做翻译了，网上找到了两篇介绍的文章
 [Boost库的命名规则](http://www.xuebuyuan.com/543678.html)
 [Boost库编译后命名方式](https://www.cnblogs.com/dementia/archive/2009/04/10/1433217.html)

## 指定使用的boost编译版本说明
这里主要是要说一下如何指定使用特定编译版本的boost库。
在工程中可以通过定义下面几个宏变量来设置

|变量名|含义|
|:---|:---|
|BOOST_LIB_NAME|必需：包含库的基本名称的字符串，例如boost_regex|
|BOOST_LIB_TOOLSET|可选：工具集的基本名称，例如VS2015就是vc140|
|BOOST_LIB_THREAD_OPT|多线程版本选项，-mt用于多线程构建，否则为空|
|BOOST_LIB_RT_OPT|指示使用的运行时库的后缀在连字符后包含以下一个或多个字母：<br>s    使用静态运行时库的版本(对应VC的MT)，留空则为动态运行时库版本<br>g    Debug版本运行时库版本(对应VC的MTd或MDd)，为空则为release运行时版本<br>y    python Debug版本<br>d    Debug版本库<br>p    使用STLport编译版本<br>n    没有使用iostream的STLport构建版本|
|BOOST_LIB_ARCH_AND_MODEL_OPT|架构和地址模型(-x32表示x86/32版本-x64表示x86/64版本)|
|BOOST_LIB_VERSION|Boost版本，形式为x_y，用于Boost版本x.y.|
|以下是用于编译boost时候的|
|BOOST_DYN_LINK|可选：要设置链接dll而不是静态库时|
|BOOST_LIB_DIAGNOSTIC|可选：要设置头文件打印出选定的库名（用于调试）|
|BOOST_AUTO_LINK_NOMANGLE|指定应该连接到BOOST_LIB_NAME.lib，而不是带这些版本信息名称(就是-mt -s -gb等，名称错位)的版本|
|BOOST_AUTO_LINK_TAGGED|指定链接到使用--layout = tagged选项构建的库。这在本质上是一样的默认名称错位版本，但没有编译器的名称和版本，或boost版本。 仅用于构建选项|

这些信息可以在`boost/config/auto_link.hpp`文件中看到。

比如说我要使用的是Boost的多线程版本静态库，链接release版动态运行时库的版本，使用的是64位架构版本，那我使用的参数如下
```
BOOST_LIB_THREAD_OPT=-mt
BOOST_LIB_RT_OPT
BOOST_LIB_ARCH_AND_MODEL_OPT=-x64
BOOST_LIB_VERSION=1_66
```
对于`BOOST_LIB_NAME`和`BOOST_LIB_TOOLSET`等无需指定，`BOOST_LIB_TOOLSET`在编译时候会自己确定，`BOOST_LIB_NAME`会根据你引用的头文件进行确定。

如果不想使用自动链接，自己添加指定的库到项目中，可以指定`BOOST_ALL_NO_LIB`或者`BOOST_模块名_NO_LIB`来取消自动链接库名。