---
layout:  post
title:  "mingw 构建 Geos"
date:  2017-04-25
categories:  其它
tags:  其它
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2017/04/25/6762624.html](http://www.cnblogs.com/oloroso/archive/2017/04/25/6762624.html)
## 简述

在做某个小程序时候用到了QT，而用的Qt是mingw版本的，所以使用mingw构建了一下`geos`库。

## 1、准备工作
首先需要先安装好`mingw`,这里直接使用[http://www.mingw-w64.org](http://www.mingw-w64.org)里面下载的安装器。
下载之后进行安装，根据你的需求，可以选择64位版本或者32位版本。

![](http://images2015.cnblogs.com/blog/693958/201704/693958-20170425150614115-1400318769.png)

如果是安装的mingw32版本的Qt，使用其自带的mingw编译套件也是可以的。

安装了之后还需要安装`cmake`这里就不介绍了。
安装之后进入`mingw`的安装目录下的`bin`目录，将其中的`mingw32-make.exe`拷贝一份，并改名为`make.exe`。

然后就是下载`geos`的源码了，直接点击下载[http://download.osgeo.org/geos/geos-3.6.1.tar.bz2]( http://download.osgeo.org/geos/geos-3.6.1.tar.bz2)

下载之后解压。

## 2、生成Makefile

双击打开`mingw`安装目录下的`mingw-w64.bat`，然后进入`geos`源码目录，新建并进入目录`build_mingw`。
执行下面语句生成`Makefile`文件
```bash
cmake -G "MinGW Makefiles" -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=Z:/geos_mingw ..
```
上面使用了`-DCMAKE_BUILD_TYPE=Release`指定构建Release版本，如果不指定，则构建debug版本。


> 实际上我是在安装的Git自带的MINGW64命令行工具(Git Bash实际上是msys，你也可以自己下载msys安装)下使用的，使用前先使用下面命令将`mingw`安装路径添加到`PATH`环境变量中。
```bash
export PATH=$PATH:/C/Program\ Files/mingw-w64/x86_64-5.4.0-win32-seh-rt_v5-rev0/mingw64/bin/
```
> 生成Makefile的命令是(只是路径风格不一样)
```bash
cmake -G "MinGW Makefiles" -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/Z/geos_mingw ..
```
> 其余步骤是一致的。


生成Makefile后还需要做点工作，就是将`build_mingw\include\geos\`目录下的`platform.h`和`version.h`文件拷贝到源码目录下的`include\geos`目录。

**注意，上面应该是正常的做法，但是会有错误，就是`error: 'isnan' was not declared in this scope`**
对于这个错误，只需要将源码目录下的`include\geos`中`platform.h.in`重命名为`platform.h`即可（不使用cmake生成的）。

## 3、编译
生成`Makefile`之后，使用下面命令进行编译
```
# 编译
mingw32-make -f Makefile
# 安装
mingw32-make -f Makefile install
```

我编译的64位版本下载地址在这里[https://www.justbeamit.com/zup5i](https://www.justbeamit.com/zup5i)