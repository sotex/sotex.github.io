---
layout:  post
title:  "Mingw编译DLib"
date:  2017-03-21
categories:  编程
tags:  编程
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2017/03/21/6593758.html](http://www.cnblogs.com/oloroso/archive/2017/03/21/6593758.html)
# Mingw编译DLib

因为机器上安装了`qt-opensource-windows-x86-mingw530-5.8.0`，所以准备使用其自带的`mingw530`来编译`DLib`使用。
因为`DLib`使用`CMake`的构建脚本，所以还请先安装好`CMake`。
cmake的下载地址如下[https://cmake.org/files/v3.7/cmake-3.7.2-win64-x64.msi
](https://cmake.org/files/v3.7/cmake-3.7.2-win64-x64.msi
)

编译出的文件在这里[https://share.weiyun.com/1e6d7dc98bc8a9fa2922f99ee11dcdac](https://share.weiyun.com/1e6d7dc98bc8a9fa2922f99ee11dcdac)

## 下载Dlib源码
可以直接去`DLib`的官网[http://dlib.net/](http://dlib.net/)找到你想要版本进行下载。
这里给出`DLib-19.4`的下载链接[http://dlib.net/files/dlib-19.4.zip
](http://dlib.net/files/dlib-19.4.zip
)

或者直接从github克隆一个
```bash
git clone https://github.com/davisking/dlib.git
```

## 生成Makefile

使用`Cmake`来生成`Makefile`文件。
打开开始菜单里面的`Qt 5.8`下面的`Qt 5.8 for Desktop (MinGW 5.3.0 32 bit)`命令行工具，然后输入下面的命令生成`Makefile`文件。

如果你不是使用的`Qt`自带的`mingw`，那么也可以使用你自己下载安装的`mingw`。前提是需要先把`mingw`的`bin`目录路径添加到环境变量`Path`中。可以使用`set Path=%Path%;mingw的bin路径`来临时使用。

```bash
D:\dlib-19.4\build>cmake -DCMAKE_INSTALL_PREFIX=D:/libs/dlib -DCMAKE_C_COMPILER=gcc -DCMAKE_CXX_COMPILER=g++ -DCMAKE_BUILD_TYPE=Release -DCMAKE_MAKE=mingw32-make ..
```

实际上上面的命令还有些不对，请看后面的说明。

使用上面命令之后出现了如下错误输出
```bash
-- The C compiler identification is GNU 5.3.0
-- The CXX compiler identification is GNU 5.3.0
-- Check for working C compiler: C:/Qt/Qt5.8.0/Tools/mingw530_32/bin/gcc.exe
CMake Error: Generator: execution of make failed. Make command was: "nmake" "/NOLOGO" "cmTC_faf78\fast"
-- Check for working C compiler: C:/Qt/Qt5.8.0/Tools/mingw530_32/bin/gcc.exe -- broken
CMake Error at C:/Program Files/CMake/share/cmake-3.7/Modules/CMakeTestCCompiler.cmake:51 (message):
  The C compiler "C:/Qt/Qt5.8.0/Tools/mingw530_32/bin/gcc.exe" is not able to
  compile a simple test program.
```
看输出是其去创建了`nmake`的构建脚本，这不是预期的。

于是添加了`-G "MinGW Makefiles"`选项之后重新生成，命令如下
```bash
cmake -DCMAKE_INSTALL_PREFIX=D:/libs/dlib -DCMAKE_C_COMPILER=gcc -DCMAKE_CXX_COMPILER=g++ -DCMAKE_BUILD_TYPE=Release  -G"MinGW Makefiles"  ..
```
又出现下面的错误了
```bash
CMake Error: Error: generator : MinGW Makefiles
Does not match the generator used previously: NMake Makefiles
Either remove the CMakeCache.txt file and CMakeFiles directory or choose a different binary directory.
```
这里说不能匹配以前生成使用的`NMake Makefiles`。这是因为`cmake`命令执行的时候具有缓存`cache`的原因，解决的办法可以是删除它。

删除掉`build`目录下的所有文件后重新运行下面命令
```
cmake -DCMAKE_INSTALL_PREFIX=D:/libs/dlib -DCMAKE_C_COMPILER=gcc -DCMAKE_CXX_COMPILER=g++ -DCMAKE_BUILD_TYPE=Release  -G"MinGW Makefiles" ..
```
生成`Makefile`成功后输出
```bash
-- Configuring done
-- Generating done
-- Build files have been written to: D:/development_library/dlib/dlib-19.4/build
```

----
在生成的时候还提示了没有发现`CUDA`和`cuDNN`。因为我这里不需要就不做了，有需要的人可以自己下载安装(需要显卡支持)后，重新运行`cmake`命令。

## 编译
生成`Makefile`后，使用下面命令进行编译

``` bash
mingw32-make -f Makefile	# 编译
mingw32-make -f Makefile install # 安装
```
安装之后可以到`-DCMAKE_INSTALL_PREFIX=D:/libs/dlib`指定的目录中找到对应的头文件和库文件。