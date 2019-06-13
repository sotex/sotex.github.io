---
layout:  post
title:  "windows下 jemalloc编译"
date:  2017-06-27
categories:  编程
tags:  编程
comments: 1
---
[TOC]
[博客园原文地址 http://www.cnblogs.com/oloroso/archive/2017/06/27/7085648.html](http://www.cnblogs.com/oloroso/archive/2017/06/27/7085648.html)



## 1、准备
Windows下使用VS2015进行编译，需要使用cmake构建版本。（如果有cygwin，在其中执行VS的vcvarsall.bat后使用`"CC=cl ./autogen.sh"`命令生成Makefile后编译也是可以的）
下载源码
```bash
git clone https://github.com/jemalloc/jemalloc-cmake.git
```
请确保已经安装好`cmake`工具。

还可以下载一个专门为win32修改的版本，支持VC6编译。
```bash
git clone https://github.com/BlzFans/jemalloc_win32.git
```

## 2、编译
分别使用VS2015和mingw编译。

### VS2015编译jemalloc
**方式一**
直接使用VS2015打开源码目录下的`msvc`目录下的`jemalloc_vc2015`，然后进行编译即可。
使用这种方式编译的时候有很多问题。首先是**VS2015不支持C11的atomic，没有stdatomic.h头文件**，这需要注释掉`JEMALLOC_C11ATOMICS`这个宏定义。然后是大量`__builtin_xxxxx`函数未定义，这个可以使用自己实现的。

**方式二**
打开`VS2015 x64本机工具命令提示符`(VS2015 x64 Native Tools Command Prompt)。
进入源码目录，创建一个目录`build`，进入`build`目录后执行下面命令。
```bat
Z:\jemalloc\jemalloc-cmake\build>cmake -G"Visual Studio 14 Win64" -DCMAKE_INSTALL_PREFIX=Z:\vs140-64 ..
```
因为jemalloc的`CMakeLists.txt`文件中实际上没有写`install`部分(被注释掉)，所以指定安装目录是无效的。
还可以直接执行`CMake_configure.cmd`来生成VS工程。

如果没有问题，将在`build`目录下生成VS工程文件。
执行下面命令进行编译(这里将只编译`Release`版本)
```bat
Z:\jemalloc\jemalloc-cmake\build>msbuild /p:configuration=Release /maxcpucount:8 ALL_BUILD.vcxproj
```
生成的目标文件在`build/Release`目录下。

也可以使用`CygWin`来生成`Makefile`文件，相关介绍在`ReadMe.txt`文件中有写。

### MinGW下编译jemalloc
打开mingw命令行工具(或者msys2/cygwin等)进入源码目录，新建目录`build-mingw`并进入。
运行下面命令生成Makefile文件
```bash
cmake -G"MSYS Makefiles" -DCMAKE_SYSTEM_NAME=Windows ..
```
生成过程中遇到以下错误
**错误1**
```bash
-- CMAKE_C_COMPILER_ID: GNU
CMake Error at Utilities.cmake:778 (CHECK_C_COMPILER_FLAG):
  Unknown CMake command "CHECK_C_COMPILER_FLAG".
Call Stack (most recent call first):
  CMakeLists.txt:149 (JeCflagsAppend)


-- Configuring incomplete, errors occurred!
See also "F:/compile/jemalloc-cmake/build/CMakeFiles/CMakeOutput.log".
```
这个直接注释掉`Utilities.cmake`的`778`行即可。

**错误2**
```bash
CMake Error at Utilities.cmake:755 (message):
  GetSystemPageSize failed compilation see
  F:/compile/jemalloc-cmake/build/GetPageSize/getpagesize.log
Call Stack (most recent call first):
  CMakeLists.txt:464 (GetSystemPageSize)
```
这里可以查看`CMakeLists.txt`的`464`行前后，发现是系统分页大小没有获取到的原因，这里可以直接给它设置为`4096`(这个可以使用下面的代码获取)
**GetPageSize.c**
```C
#include <windows.h>
#include <stdio.h>
int main(int argc, const char** argv) {
	int result;
#ifdef _WIN32
	SYSTEM_INFO si;
	GetSystemInfo(&si);
	result = si.dwPageSize;
#else
	result = sysconf(_SC_PAGESIZE);
#endif
	printf("%d", result);
	return 0;
}
```
使用下面命令重新生成
```
cmake -G"MSYS Makefiles" -DCMAKE_SYSTEM_NAME=Windows -DLG_PAGE=4096 ..
```
不知道是什么原因，`LG_PAGE`设置4096就会失败(Please wait while we configure class sizes)，设置为`4`没有问题，不知道是不是单位的原因(Windows下分页应该是4MB)

然后使用下面命令进行构建(必须指定`C_FLAGS`参数，因为生成的Makefile中使用的是VC编译器的参数)
```
 make C_FLAGS="-D_WIN32 -DWIN32 -DWIN64 -O2"
 ```
编译时出现如下错误
**错误1**
```bash
include/jemalloc/internal/spin.h:41:3: 错误：‘CPU_SPINWAIT’未声明(在此函数内第一次使用)
   CPU_SPINWAIT;
   ^~~~~~~~~~~~
```
这个只需要修改`include\jemalloc\internal\jemalloc_internal_defs.h`文件的第`23`行。修改为
```C
#define CPU_SPINWAIT _mm_pause()
```
更多类似的问题，可以通过拷贝cmake生成VS工程中的`jemalloc_internal_defs.h`覆盖原文件来解决。

**错误2**
```bash
[  4%] Linking C shared library libjemallocso.dll
gcc: 错误：/FC：No such file or directory
gcc: 错误：/d2Zi+：No such file or directory
gcc: 错误：/Zi：No such file or directory
gcc: 错误：/FS：No such file or directory
gcc: 错误：/nologo：No such file or directory
gcc: 错误：/W3：No such file or directory
gcc: 错误：/WX：No such file or directory
gcc: 错误：/GS：No such file or directory
gcc: 错误：/Zc:wchar_t：No such file or directory
gcc: 错误：/Zc:forScope：No such file or directory
gcc: 错误：/errorReport:queue：No such file or directory
gcc: 错误：/wd4267：No such file or directory
gcc: 错误：/wd4244：No such file or directory
gcc: 错误：/wd4146：No such file or directory
gcc: 错误：/wd4334：No such file or directory
gcc: 错误：/wd4090：No such file or directory
make[2]: *** [CMakeFiles/jemallocso.dir/build.make:798：libjemallocso.dll] 错误 1
make[1]: *** [CMakeFiles/Makefile2:1510：CMakeFiles/jemallocso.dir/all] 错误 2
make: *** [Makefile:95：all] 错误 2
```
直接打开`CMakeFiles/jemallocso.dir/build.make`文件，定位到798行，将其中的`/FC /d2Zi+`等删除即可。其他类似的问题都可以这样解决。（包括chunk、xallocx、thread_tcache_enabled、overflow、mallocx等等，如果不想一个个解决，可以使用-k参数跳过这些工具的编译。**只需要jemallocso编译出来了就可以用了**）