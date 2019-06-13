---
layout:  post
title:  "nmake构建Geos库"
date:  2017-04-18
categories:  其它
tags:  其它
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2017/04/18/6728506.html](http://www.cnblogs.com/oloroso/archive/2017/04/18/6728506.html)
## 1、下载源码包
下载地址 [http://download.osgeo.org/geos/geos-3.6.1.tar.bz2](http://download.osgeo.org/geos/geos-3.6.1.tar.bz2)
下载之后解压即可。

## 2、编译

geos源码包中自带了`makefile.vc`，所以可以直接使用`nmake`进行构建。
打开VS的命令行工具（我的是VS2015 x64 Native Build Tools Command Prompt）
进入源码目录，使用下面命令进行构建（构建前请先运行一下autogen.bat，也可手动将include目录下的version.h.vc和platform.h.vc，去掉.vc后缀名）
```cmd
nmake -f makefile.vc BUILD_DEBUG=YES WIN64=YES ENABLE_INLINE=YES

# 以下参数也可以在nmake.opt中修改
# BUILD_DEBUG=YES 构建Debug版本，构建Release版本改为NO（默认就是）
# WIN64=YES       构建Win64版本
# ENABLE_INLINE=YES 开启内联（默认为NO）
```
geos的makefile.vc中是没有带`install`目标的，所以构建完成之后需要手动去拷贝相关的文件。
编译出的`lib`和`dll`文件在源码包的`src`目录下，头文件在`include`目录下。

编译出的`geos.lib`是静态库，`geos_i.lib`和`geos_c_i.lib`则是动态库接口导出文件。