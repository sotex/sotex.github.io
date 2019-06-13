---
layout:  post
title:  "Windows下libjpeg-trubo-1.5.3编译（VS2015）"
date:  2018-06-04
categories:  编程
tags:  编程
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2018/06/04/9132665.html](http://www.cnblogs.com/oloroso/archive/2018/06/04/9132665.html)
# 简述
[https://libjpeg-turbo.org/](https://libjpeg-turbo.org/)的网站上是有已经编译好的版本下载的，但是VC下是使用的VC10.0编译的。虽然在VC14.0下也能用，但是我还是需要编译一个VC14.0版本的。

# 准备工作
先去下载源码包[https://jaist.dl.sourceforge.net/project/libjpeg-turbo/1.5.3/libjpeg-turbo-1.5.3.tar.gz](https://jaist.dl.sourceforge.net/project/libjpeg-turbo/1.5.3/libjpeg-turbo-1.5.3.tar.gz)
然后需要安装一下NASM汇编工具，这个可以在[https://www.nasm.us/](https://www.nasm.us/)网站找到。
[nasm-2.13.03-installer-x86.exe](https://www.nasm.us/pub/nasm/releasebuilds/2.13.03/win32/nasm-2.13.03-installer-x86.exe)
[nasm-2.13.03-installer-x64.exe](https://www.nasm.us/pub/nasm/releasebuilds/2.13.03/win64/nasm-2.13.03-installer-x64.exe)

# 使用cmake生成VS工程
我使用的CMake命令如下：
**32位**
```
cmake -DCMAKE_CONFIGURATION_TYPES:STRING="Release" -DCMAKE_LINKER:FILEPATH="C:/Program Files (x86)/Microsoft Visual Studio 14.0/VC/bin/link.exe" -DNASM:FILEPATH="C:/Program Files (x86)/NASM/nasm.exe" -DCMAKE_C_FLAGS:STRING="/DWIN32 /D_WINDOWS /W3" -DCMAKE_INSTALL_PREFIX:PATH="Z:/compiler/out/libjpeg-turbo"  .
```
**64位**
```
cmake -DCMAKE_CONFIGURATION_TYPES:STRING="Release" -DCMAKE_LINKER:FILEPATH="C:/Program Files (x86)/Microsoft Visual Studio 14.0/VC/bin/x86_amd64/link.exe" -DNASM:FILEPATH="C:/Program Files/NASM/nasm.exe" -DCMAKE_C_FLAGS:STRING="/DWIN32 /D_WINDOWS /W3" -DCMAKE_INSTALL_PREFIX:PATH="Z:/compiler/out/libjpeg-turbo64" -DCMAKE_MODULE_LINKER_FLAGS_DEBUG:STRING="/debug /INCREMENTAL" .
```
生成后直接VS打开编译或者进入工程目录使用下面命令进行编译
```
msbuild libjpeg-trurbo.sln
```

# 编译好的文件
64位版本 [libjpeg-turbo_v140x86.7z](https://files.cnblogs.com/files/oloroso/libjpeg-turbo_v140x86.7z)
32位版本 [libjpeg-turbo_v140x64.7z](https://files.cnblogs.com/files/oloroso/libjpeg-turbo_v140x64.7z)