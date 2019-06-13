---
layout:  post
title:  "mingw 构建 mysql-connector-c-6.1.9记录"
date:  2017-05-17
categories:  其它
tags:  其它
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2017/05/17/6867162.html](http://www.cnblogs.com/oloroso/archive/2017/05/17/6867162.html)
## 1、准备工作
首先需要下载[mysql-connector-c-6.1.9](https://dev.mysql.com/downloads/connector/c/6.1.html)的源码，然后解压。
然后需要准备编译环境，这里我使用的是`msys2`(下载地址[http://repo.msys2.org/distrib/x86_64/msys2-x86_64-20161025.exe](http://repo.msys2.org/distrib/x86_64/msys2-x86_64-20161025.exe))。
下载安装后执行下面命令：

```bash
# 先更新一下（这一步后面可能会报错，只需要关闭终端，再打开重新更新一下即可）
pacman -Syu
# 安装编译需要工具
pacman -S gcc make cmake
# 下面这句也可以不要
# pacman -S mingw-w64-x86_64-extra-cmake-modules
# 再安装一个vim，这个不是必须的
pacman -S vim
```
全部安装完成之后，即可进入下一步

## 2、生成makefile
进入解压后的源码目录
执行下面的命令生成`Makefile文件
```bash
# 先创建并进入一个构建目录
mkdir build && cd build
# 生成makefile
cmake -G"Unix Makefiles" ..
```
执行过程中报下面的错误
```bash
$ cmake -G"Unix Makefiles"  ..
-- Running cmake version 3.6.2
System is unknown to cmake, create:
Platform/MINGW64_NT-6.2 to use this system, please send your config file to cmake@www.cmake.org so it can be added to cmake
Your CMakeCache.txt file was copied to CopyOfCMakeCache.txt. Please send that file to cmake@www.cmake.org.
-- SIZEOF_VOIDP 8
-- LibMySQL 6.1.9
-- Built from MySQL 5.7.16 sources
-- Packaging as: mysql-connector-c-6.1.9-MINGW64_NT-6.2-unknown
-- Installing to: /usr/local/mysql
CMake Error at configure.cmake:573 (MESSAGE):
  No mysys timer support detected!
Call Stack (most recent call first):
  CMakeLists.txt:451 (INCLUDE)


-- Configuring incomplete, errors occurred!
See also "/z/mysql-connector-c-6.1.9-src/build/CMakeFiles/CMakeOutput.log".
See also "/z/mysql-connector-c-6.1.9-src/build/CMakeFiles/CMakeError.log".
```
这里提示是`Platform/MINGW64_NT-6.2`这个系统环境，cmake不知道怎么去创建构建脚本。这里提示让你把生成的`CopyOfCmakeCache.txt`文件发送到[cmake@www.cmake.org](cmake@www.cmake.org)，这里就不发了。
根据提示，打开`configure.cmake`文件，定位到`573`行，内容如下
```cmake
IF(NOT HAVE_POSIX_TIMERS AND NOT HAVE_KQUEUE_TIMERS AND NOT WIN32)
  MESSAGE(FATAL_ERROR "No mysys timer support detected!")
ENDIF()
```
既然这里判断出错，那就直接修改这里好了，在文件最前面前面加上下面语句
```cmake
SET(WIN32 1)
# 后面不是必须的
SET(WIN64 1)
SET(WINDOWS 1)
```
重新运行`cmake -G"Unix Makefiles" ..`即可生成`Makefile`文件。
**这样还不够，还需添加下面一句。具体的请看后面编译时候出现的问题描述**
```cmake
add_definitions(-DWIN32=1 -D_WIN32=1 -DWIN64=1 -DWINDOWS=1 -D__USE_W32_SOCKETS=1)
```

## 3、编译
编译过程并不是很顺利，中间遇到很多问题需要解决，这里简单的记录一下

### 问题1

直接执行`make`命令来生成，结果报如下错误：

```bash
[ 74%] Building C object mysys/CMakeFiles/mysys.dir/my_winerr.c.obj
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:34:6: 错误：‘ERROR_INVALID_FUNCTION’未声明(不在函数内)
   {  ERROR_INVALID_FUNCTION,       EINVAL    },  /* 1 */
      ^~~~~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:35:6: 错误：‘ERROR_FILE_NOT_FOUND’未声明(不在函数内)
   {  ERROR_FILE_NOT_FOUND,         ENOENT    },  /* 2 */
      ^~~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:36:6: 错误：‘ERROR_PATH_NOT_FOUND’未声明(不在函数内)
   {  ERROR_PATH_NOT_FOUND,         ENOENT    },  /* 3 */
      ^~~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:37:6: 错误：‘ERROR_TOO_MANY_OPEN_FILES’未声明(不在函数内)
   {  ERROR_TOO_MANY_OPEN_FILES,    EMFILE    },  /* 4 */
      ^~~~~~~~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:38:6: 错误：‘ERROR_ACCESS_DENIED’未声明(不在函数内)
   {  ERROR_ACCESS_DENIED,          EACCES    },  /* 5 */
      ^~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:39:6: 错误：‘ERROR_INVALID_HANDLE’未声明(不在函数内)
   {  ERROR_INVALID_HANDLE,         EBADF     },  /* 6 */
      ^~~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:40:6: 错误：‘ERROR_ARENA_TRASHED’未声明(不在函数内)
   {  ERROR_ARENA_TRASHED,          ENOMEM    },  /* 7 */
      ^~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:41:6: 错误：‘ERROR_NOT_ENOUGH_MEMORY’未声明(不在函数内)
   {  ERROR_NOT_ENOUGH_MEMORY,      ENOMEM    },  /* 8 */
      ^~~~~~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:42:6: 错误：‘ERROR_INVALID_BLOCK’未声明(不在函数内)
   {  ERROR_INVALID_BLOCK,          ENOMEM    },  /* 9 */
      ^~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:43:6: 错误：‘ERROR_BAD_ENVIRONMENT’未声明(不在函数内)
   {  ERROR_BAD_ENVIRONMENT,        E2BIG     },  /* 10 */
      ^~~~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:44:6: 错误：‘ERROR_BAD_FORMAT’未声明(不在函数内)
   {  ERROR_BAD_FORMAT,             ENOEXEC   },  /* 11 */
      ^~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:45:6: 错误：‘ERROR_INVALID_ACCESS’未声明(不在函数内)
   {  ERROR_INVALID_ACCESS,         EINVAL    },  /* 12 */
      ^~~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:46:6: 错误：‘ERROR_INVALID_DATA’未声明(不在函数内)
   {  ERROR_INVALID_DATA,           EINVAL    },  /* 13 */
      ^~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:47:6: 错误：‘ERROR_INVALID_DRIVE’未声明(不在函数内)
   {  ERROR_INVALID_DRIVE,          ENOENT    },  /* 15 */
      ^~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:48:6: 错误：‘ERROR_CURRENT_DIRECTORY’未声明(不在函数内)
   {  ERROR_CURRENT_DIRECTORY,      EACCES    },  /* 16 */
      ^~~~~~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:49:6: 错误：‘ERROR_NOT_SAME_DEVICE’未声明(不在函数内)
   {  ERROR_NOT_SAME_DEVICE,        EXDEV     },  /* 17 */
      ^~~~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:50:6: 错误：‘ERROR_NO_MORE_FILES’未声明(不在函数内)
   {  ERROR_NO_MORE_FILES,          ENOENT    },  /* 18 */
      ^~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:51:6: 错误：‘ERROR_LOCK_VIOLATION’未声明(不在函数内)
   {  ERROR_LOCK_VIOLATION,         EACCES    },  /* 33 */
      ^~~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:52:6: 错误：‘ERROR_BAD_NETPATH’未声明(不在函数内)
   {  ERROR_BAD_NETPATH,            ENOENT    },  /* 53 */
      ^~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:53:6: 错误：‘ERROR_NETWORK_ACCESS_DENIED’未声明(不在函数内)
   {  ERROR_NETWORK_ACCESS_DENIED,  EACCES    },  /* 65 */
      ^~~~~~~~~~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:54:6: 错误：‘ERROR_BAD_NET_NAME’未声明(不在函数内)
   {  ERROR_BAD_NET_NAME,           ENOENT    },  /* 67 */
      ^~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:55:6: 错误：‘ERROR_FILE_EXISTS’未声明(不在函数内)
   {  ERROR_FILE_EXISTS,            EEXIST    },  /* 80 */
      ^~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:56:6: 错误：‘ERROR_CANNOT_MAKE’未声明(不在函数内)
   {  ERROR_CANNOT_MAKE,            EACCES    },  /* 82 */
      ^~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:57:6: 错误：‘ERROR_FAIL_I24’未声明(不在函数内)
   {  ERROR_FAIL_I24,               EACCES    },  /* 83 */
      ^~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:58:6: 错误：‘ERROR_INVALID_PARAMETER’未声明(不在函数内)
   {  ERROR_INVALID_PARAMETER,      EINVAL    },  /* 87 */
      ^~~~~~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:59:6: 错误：‘ERROR_NO_PROC_SLOTS’未声明(不在函数内)
   {  ERROR_NO_PROC_SLOTS,          EAGAIN    },  /* 89 */
      ^~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:60:6: 错误：‘ERROR_DRIVE_LOCKED’未声明(不在函数内)
   {  ERROR_DRIVE_LOCKED,           EACCES    },  /* 108 */
      ^~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:61:6: 错误：‘ERROR_BROKEN_PIPE’未声明(不在函数内)
   {  ERROR_BROKEN_PIPE,            EPIPE     },  /* 109 */
      ^~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:62:6: 错误：‘ERROR_DISK_FULL’未声明(不在函数内)
   {  ERROR_DISK_FULL,              ENOSPC    },  /* 112 */
      ^~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:63:6: 错误：‘ERROR_INVALID_TARGET_HANDLE’未声明(不在函数内)
   {  ERROR_INVALID_TARGET_HANDLE,  EBADF     },  /* 114 */
      ^~~~~~~~~~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:64:6: 错误：‘ERROR_INVALID_NAME’未声明(不在函数内)
   {  ERROR_INVALID_NAME,           ENOENT    },  /* 123 */
      ^~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:66:6: 错误：‘ERROR_WAIT_NO_CHILDREN’未声明(不在函数内)
   {  ERROR_WAIT_NO_CHILDREN,       ECHILD    },  /* 128 */
      ^~~~~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:67:6: 错误：‘ERROR_CHILD_NOT_COMPLETE’未声明(不在函数内)
   {  ERROR_CHILD_NOT_COMPLETE,     ECHILD    },  /* 129 */
      ^~~~~~~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:68:6: 错误：‘ERROR_DIRECT_ACCESS_HANDLE’未声明(不在函数内)
   {  ERROR_DIRECT_ACCESS_HANDLE,   EBADF     },  /* 130 */
      ^~~~~~~~~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:69:6: 错误：‘ERROR_NEGATIVE_SEEK’未声明(不在函数内)
   {  ERROR_NEGATIVE_SEEK,          EINVAL    },  /* 131 */
      ^~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:70:6: 错误：‘ERROR_SEEK_ON_DEVICE’未声明(不在函数内)
   {  ERROR_SEEK_ON_DEVICE,         EACCES    },  /* 132 */
      ^~~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:71:6: 错误：‘ERROR_DIR_NOT_EMPTY’未声明(不在函数内)
   {  ERROR_DIR_NOT_EMPTY,          ENOTEMPTY },  /* 145 */
      ^~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:72:6: 错误：‘ERROR_NOT_LOCKED’未声明(不在函数内)
   {  ERROR_NOT_LOCKED,             EACCES    },  /* 158 */
      ^~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:73:6: 错误：‘ERROR_BAD_PATHNAME’未声明(不在函数内)
   {  ERROR_BAD_PATHNAME,           ENOENT    },  /* 161 */
      ^~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:74:6: 错误：‘ERROR_MAX_THRDS_REACHED’未声明(不在函数内)
   {  ERROR_MAX_THRDS_REACHED,      EAGAIN    },  /* 164 */
      ^~~~~~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:75:6: 错误：‘ERROR_LOCK_FAILED’未声明(不在函数内)
   {  ERROR_LOCK_FAILED,            EACCES    },  /* 167 */
      ^~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:76:6: 错误：‘ERROR_ALREADY_EXISTS’未声明(不在函数内)
   {  ERROR_ALREADY_EXISTS,         EEXIST    },  /* 183 */
      ^~~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:77:6: 错误：‘ERROR_FILENAME_EXCED_RANGE’未声明(不在函数内)
   {  ERROR_FILENAME_EXCED_RANGE,   ENOENT    },  /* 206 */
      ^~~~~~~~~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:78:6: 错误：‘ERROR_NESTING_NOT_ALLOWED’未声明(不在函数内)
   {  ERROR_NESTING_NOT_ALLOWED,    EAGAIN    },  /* 215 */
      ^~~~~~~~~~~~~~~~~~~~~~~~~
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:79:6: 错误：‘ERROR_NOT_ENOUGH_QUOTA’未声明(不在函数内)
   {  ERROR_NOT_ENOUGH_QUOTA,       ENOMEM    }    /* 1816 */
      ^~~~~~~~~~~~~~~~~~~~~~
```
**解决办法：**
直接打开`my_winerr.c`这个文件，可以看到这里是定义了`windows`系统返回错误码与`System V`错误码的一个对照表。
```c
struct errentry
{
  unsigned long oscode;   /* OS return value */
  int sysv_errno;  /* System V error code */
};

static struct errentry errtable[]= {
  {  ERROR_INVALID_FUNCTION,       EINVAL    },  /* 1 */
  {  ERROR_FILE_NOT_FOUND,         ENOENT    },  /* 2 */
  {  ERROR_PATH_NOT_FOUND,         ENOENT    },  /* 3 */
  {  ERROR_TOO_MANY_OPEN_FILES,    EMFILE    },  /* 4 */ 
   .... 后面省略 ...
```
**Windows的错误码定义都在`WinError.h`文件中，所以只需要修改`my_winerr.c`文件，在前面添加
```c
#include <WinError.h>
```
即可。

### 问题2

继续`make`编译，报如下错误
```c
[ 39%] Building C object mysys/CMakeFiles/mysys.dir/my_winerr.c.obj
In file included from /z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:26:0:
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:37:6: 错误：初始值设定元素不是常量
   {  ERROR_INVALID_FUNCTION,       EINVAL    },  /* 1 */
      ^
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:37:6: 附注：(在‘errtable[0].oscode’的初始化附近)
/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c:38:6: 错误：初始值设定元素不是常量
   {  ERROR_FILE_NOT_FOUND,         ENOENT    },  /* 2 */
```
这个错误的原因是因为c编译器不支持函数外动态声明变量和分配空间，但是这里都是宏，应该替换成了字面常量才对。
通过`gcc -E`预处理以下(预处理需要指定包含路径，这个可以去libmysql/CMakeFiles目录下找flags.cmake文件查看)，可以得到下面代码（为了方便，我把行数输出了）
```
  7309  struct errentry
  7310  {
  7311    unsigned long oscode;
  7312    int sysv_errno;
  7313  };
  7314
  7315  static struct errentry errtable[]= {
  7316    {
  7317  # 37 "/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c" 3 4
  7318      __MSABI_LONG(1)
  7319  # 37 "/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c"
  7320                            ,
  7321  # 37 "/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c" 3 4
  7322                                    22
  7323  # 37 "/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c"
  7324                                              },
  7325    {
  7326  # 38 "/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c" 3 4
  7327      __MSABI_LONG(2)
  7328  # 38 "/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c"
  7329                          ,
  7330  # 38 "/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c" 3 4
  7331                                    2
  7332  # 38 "/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c"
  7333                                              },
  7334    {
  7335  # 39 "/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c" 3 4
  7336      __MSABI_LONG(3)
  7337  # 39 "/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c"
  7338                          ,
  7339  # 39 "/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c" 3 4
  7340                                    2
  7341  # 39 "/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c"
  7342                                              },
  7343    {
  7344  # 40 "/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c" 3 4
  7345      __MSABI_LONG(4)
  7346  # 40 "/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c"
  7347                               ,
  7348  # 40 "/z/mysql-connector-c-6.1.9-src/mysys/my_winerr.c" 3 4
```
**解决办法：**
这里看到常数被`__MSABI_LONG`这个宏处理了一次，所以这里把这个宏给去掉即可。
在结构体数组初始化前后分别加上下面语句即可：
```c
// 初始化前加上下面语句
#pragma push_macro("__MSABI_LONG")
#define __MSABI_LONG(x) x

// 初始化赋值
static struct errentry errtable[]= {
  {  ERROR_INVALID_FUNCTION,       EINVAL    },  /* 1 */
  {  ERROR_FILE_NOT_FOUND,         ENOENT    },  /* 2 */
   .... 省略 ...
  {  ERROR_NESTING_NOT_ALLOWED,    EAGAIN    },  /* 215 */
  {  ERROR_NOT_ENOUGH_QUOTA,       ENOMEM    }    /* 1816 */
};

// 初始化后加上下面语句
#undef __MSABI_LONG
#pragma pop_macro("__MSABI_LONG")
```
再次执行`make`即可。

### 问题3

继续编译，报如下错误：
```bash
[ 75%] Building C object vio/CMakeFiles/vio.dir/viopipe.c.obj
/z/mysql-connector-c-6.1.9-src/vio/viopipe.c: 在函数‘wait_overlapped_result’中:
/z/mysql-connector-c-6.1.9-src/vio/viopipe.c:21:3: 错误：未知的类型名‘DWORD’
   DWORD transferred, wait_status, timeout_ms;
   ^~~~~
.... 省略更多 ...
```
这里只需要添加头文件`Windows.h`即可。

### 问题4

继续编译，报如下错误
```c
[ 77%] Building C object vio/CMakeFiles/vio.dir/viopipe.c.obj
In file included from /z/mysql-connector-c-6.1.9-src/vio/vio_priv.h:24:0,
                 from /z/mysql-connector-c-6.1.9-src/vio/viopipe.c:18:
/z/mysql-connector-c-6.1.9-src/include/m_string.h: 在函数‘native_strcasecmp’中:
/z/mysql-connector-c-6.1.9-src/include/my_global.h:779:22: 警告：隐式声明函数‘_stricmp’ [-Wimplicit-function-declaration]
   #define strcasecmp _stricmp
                      ^
/z/mysql-connector-c-6.1.9-src/include/my_global.h:779:22: 附注：in definition of macro ‘strcasecmp’
   #define strcasecmp _stricmp
                      ^~~~~~~~
/z/mysql-connector-c-6.1.9-src/vio/viopipe.c: 在函数‘wait_overlapped_result’中:
/z/mysql-connector-c-6.1.9-src/vio/viopipe.c:25:38: 警告：signed and unsigned type in conditional expression [-Wsign-compare]
   timeout_ms= timeout >= 0 ? timeout : INFINITE;
                                      ^
/z/mysql-connector-c-6.1.9-src/vio/viopipe.c:28:39: 错误：‘Vio {或称 struct st_vio}’没有名为‘overlapped’的成员
   wait_status= WaitForSingleObject(vio->overlapped.hEvent, timeout_ms);
                                       ^~
/z/mysql-connector-c-6.1.9-src/vio/viopipe.c:34:32: 错误：‘Vio {或称 struct st_vio}’没有名为‘hPipe’的成员
     if (GetOverlappedResult(vio->hPipe, &vio->overlapped, &transferred, FALSE))
                                ^~
/z/mysql-connector-c-6.1.9-src/vio/viopipe.c:34:45: 错误：‘Vio {或称 struct st_vio}’没有名为‘overlapped’的成员
     if (GetOverlappedResult(vio->hPipe, &vio->overlapped, &transferred, FALSE))
                                             ^~
/z/mysql-connector-c-6.1.9-src/vio/viopipe.c:40:17: 错误：‘Vio {或称 struct st_vio}’没有名为‘hPipe’的成员
     CancelIo(vio->hPipe);
```
两个警告就先不管了（找具体头文件）。这里后面的报错，应该是`Vio`这个结构体中的定义问题，需要找到其定义的位置。
可以找到相关的定义在`mysql-connector-c-6.1.9-src/include/violite.h`文件中的`267`到`337`之间（就是最后一部分）。
可以看到`overlapped`成员是否被定义，与`_WIN32`宏是否被定义有关。
看来是之前生成`Makefile`的时候没有把widnows下相关的宏定义给加上。

再次修改`configure.cmake`文件，添加下面语句
```cmake
add_definitions(-DWIN32=1 -D_WIN32=1 -DWIN64=1 -DWINDOWS=1)
```
再次运行`cmake -G"Unix Makefiles" ..`生成编译脚本，然后运行`make`进行编译

**以下的大部分错误，都是由于_WIN32这个宏导致的！！！**


### 问题5

再次遇到如下错误：
```bash
[  7%] Building CXX object extra/yassl/CMakeFiles/yassl.dir/src/cert_wrapper.cpp.o
In file included from /usr/include/w32api/winsock2.h:56:0,
                 from Z:/mysql-connector-c-6.1.9-src/extra/yassl/include/socket_wrapper.hpp:31,
                 from Z:/mysql-connector-c-6.1.9-src/extra/yassl/include/log.hpp:27,
                 from Z:/mysql-connector-c-6.1.9-src/extra/yassl/include/yassl_int.hpp:32,
                 from Z:/mysql-connector-c-6.1.9-src/extra/yassl/src/cert_wrapper.cpp:26:
/usr/include/w32api/psdk_inc/_fd_types.h:100:2: 警告：#warning "fd_set and associated macros have been defined in sys/types.      This can cause runtime problems with W32 sockets" [-Wcpp]
 #warning "fd_set and associated macros have been defined in sys/types.  \
  ^~~~~~~
In file included from Z:/mysql-connector-c-6.1.9-src/extra/yassl/include/socket_wrapper.hpp:31:0,
                 from Z:/mysql-connector-c-6.1.9-src/extra/yassl/include/log.hpp:27,
                 from Z:/mysql-connector-c-6.1.9-src/extra/yassl/include/yassl_int.hpp:32,
                 from Z:/mysql-connector-c-6.1.9-src/extra/yassl/src/cert_wrapper.cpp:26:
/usr/include/w32api/winsock2.h:995:34: 错误：conflicting declaration of C function ‘int select(int, _types_fd_set*, _types_fd_set*, _types_fd_set*, PTIMEVAL)’
   WINSOCK_API_LINKAGE int WSAAPI select(int nfds,fd_set *readfds,fd_set *writefds,fd_set *exceptfds,const PTIMEVAL timeout);
                                  ^~~~~~
In file included from /usr/include/sys/types.h:68:0,
                 from /usr/include/time.h:28,
                 from Z:/mysql-connector-c-6.1.9-src/extra/yassl/taocrypt/include/asn.hpp:27,
                 from Z:/mysql-connector-c-6.1.9-src/extra/yassl/include/cert_wrapper.hpp:36,
                 from Z:/mysql-connector-c-6.1.9-src/extra/yassl/src/cert_wrapper.cpp:25:
/usr/include/sys/select.h:73:5: 附注：previous declaration ‘int select(int, _types_fd_set*, _types_fd_set*, _types_fd_set*, timeval*)’
 int select __P ((int __n, fd_set *__readfds, fd_set *__writefds,
     ^~~~~~
make[2]: *** [extra/yassl/CMakeFiles/yassl.dir/build.make:89：extra/yassl/CMakeFiles/yassl.dir/src/cert_wrapper.cpp.o] 错误 1
make[1]: *** [CMakeFiles/Makefile2:279：extra/yassl/CMakeFiles/yassl.dir/all] 错误 2
make: *** [Makefile:152：all] 错误 2
```
这次的问题提示很明确，就是`fd_set`和`select`函数多处声明冲突的问题。
这个是由于在mingw下有相关的socket定义的问题，这个可以看`/usr/include/w32api/psdk_inc/_fd_types.h`文件得知。
```c
#ifndef ___WSA_FD_TYPES_H
#define ___WSA_FD_TYPES_H

#include <psdk_inc/_socket_types.h>

#ifndef FD_SETSIZE
#define FD_SETSIZE      64
#endif

#ifndef _SYS_TYPES_FD_SET
/* fd_set may have been defined by the newlib <sys/types.h>
 * if  __USE_W32_SOCKETS not defined.
 */

typedef struct fd_set
{
        u_int   fd_count;
        SOCKET  fd_array[FD_SETSIZE];
} fd_set;
```
可以看到注释中有`if  __USE_W32_SOCKETS not defined.`一句，这里只需要将其加入`Makefile`中即可。
我直接修改的`configure.cmake`文件中之前添加的语句，如下：
```cmake
add_definitions(-DWIN32=1 -D_WIN32=1 -DWIN64=1 -DWINDOWS=1 -D__USE_W32_SOCKETS=1)
```
再次生成编译脚本并执行`make`即可。

### 问题6
继续编译，又遇到如下问题
```bash
[  9%] Building CXX object extra/yassl/CMakeFiles/yassl.dir/src/socket_wrapper.cpp.obj
Z:/mysql-connector-c-6.1.9-src/extra/yassl/src/socket_wrapper.cpp: 在成员函数‘yaSSL::uint yaSSL::Socket::get_ready() const’中:
Z:/mysql-connector-c-6.1.9-src/extra/yassl/src/socket_wrapper.cpp:119:42: 错误：不能将‘int ioctlsocket(SOCKET, int, __ms_u_long*)’的实参‘3’从‘long unsigned int*’转换到‘__ms_u_long* {aka unsigned int*}’
     ioctlsocket(socket_, FIONREAD, &ready);
                                          ^
make[2]: *** [extra/yassl/CMakeFiles/yassl.dir/build.make:214：extra/yassl/CMakeFiles/yassl.dir/src/socket_wrapper.cpp.obj] 错误 1
make[1]: *** [CMakeFiles/Makefile2:215：extra/yassl/CMakeFiles/yassl.dir/all] 错误 2
make: *** [Makefile:152：all] 错误 2
```
这里是因为函数`ioctlsocket`的第三个参数`&ready`类型为`long unsigned int*`而不是需要的`__ms_u_long* {aka unsigned int*}`，直接强转即可。
这个很好解决，直接修改`mysql-connector-c-6.1.9-src/extra/yassl/src/socket_wrapper.cpp`文件中的118行 为如下:
```c
// unsigned long ready = 0;
__ms_u_long ready = 0;
```

### 问题7
继续编译，遇到如下问题
```bash
[  7%] Building CXX object extra/yassl/CMakeFiles/yassl.dir/src/ssl.cpp.obj
Z:/mysql-connector-c-6.1.9-src/extra/yassl/src/ssl.cpp: 在函数‘char* yaSSL::ya_SSL_ASN1_TIME_to_string(yaSSL::ASN1_STRING*, char*, size_t)’中:
Z:/mysql-connector-c-6.1.9-src/extra/yassl/src/ssl.cpp:1860:42: 错误：‘_snprintf’在此作用域中尚未声明
                t.tm_sec, t.tm_year + 1900);
                                          ^
make[2]: *** [extra/yassl/CMakeFiles/yassl.dir/build.make:239：extra/yassl/CMakeFiles/yassl.dir/src/ssl.cpp.obj] 错误 1
make[1]: *** [CMakeFiles/Makefile2:215：extra/yassl/CMakeFiles/yassl.dir/all] 错误 2
make: *** [Makefile:152：all] 错误 2
```
这里可以直接将错误处(mysql-connector-c-6.1.9-src/extra/yassl/src/ssl.cpp的1860行)的`_snprintf`直接改为`snprintf`即可。

### 问题8
继续编译，将遇到一大堆重定义的问题，基本都来自于`my_global.h`文件中。
这是由于`mingw`的自带的头文件中与`Widnows sSDK`中的头文件中的一些定义有冲突（mingw把一些linux上的系统函数进行了实现），这里只需要判断后进行注释掉即可。

下面把有冲突的地方列出来。
```bash
[  8%] Building C object extra/yassl/CMakeFiles/yassl.dir/__/__/client/get_password.c.obj

Z:/mysql-connector-c-6.1.9-src/include/my_global.h:134:0: 警告：“O_NONBLOCK”重定义
 #define O_NONBLOCK 1    /* For emulation of fcntl() */

/usr/include/sys/_default_fcntl.h:44:0: 附注：这是先前定义的位置
 #define O_NONBLOCK _FNONBLOCK

In file included from Z:/mysql-connector-c-6.1.9-src/client/get_password.c:20:0:
Z:/mysql-connector-c-6.1.9-src/include/my_global.h:219:19: 错误：与‘sigset_t’类型冲突
 typedef int       sigset_t;
                   ^~~~~~~~
/usr/include/sys/signal.h:18:20: 附注：‘sigset_t’的上一个声明在此
 typedef __sigset_t sigset_t;
                    ^~~~~~~~
In file included from Z:/mysql-connector-c-6.1.9-src/client/get_password.c:20:0:
Z:/mysql-connector-c-6.1.9-src/include/my_global.h:220:19: 错误：与‘mode_t’类型冲突
 typedef int       mode_t;
                   ^~~~~~
/usr/include/sys/types.h:205:18: 附注：‘mode_t’的上一个声明在此
 typedef __mode_t mode_t;  /* permissions */
                  ^~~~~~
Z:/mysql-connector-c-6.1.9-src/include/my_global.h:221:19: 错误：与‘ssize_t’类型冲突
 typedef SSIZE_T   ssize_t;
                   ^~~~~~~
/usr/include/sys/types.h:200:18: 附注：‘ssize_t’的上一个声明在此
 typedef _ssize_t ssize_t;
                  ^~~~~~~
In file included from Z:/mysql-connector-c-6.1.9-src/client/get_password.c:20:0:
Z:/mysql-connector-c-6.1.9-src/include/my_global.h:653:1: 错误：对‘localtime_r’的静态声明出现在非静态声明之后
 {
 ^
/usr/include/time.h:81:12: 附注：‘localtime_r’的上一个声明在此
 struct tm *_EXFUN(localtime_r, (const time_t *__restrict,
            ^
In file included from Z:/mysql-connector-c-6.1.9-src/client/get_password.c:20:0:
Z:/mysql-connector-c-6.1.9-src/include/my_global.h: 在函数‘localtime_r’中:
Z:/mysql-connector-c-6.1.9-src/include/my_global.h:654:3: 警告：隐式声明函数‘localtime_s’ [-Wimplicit-function-declaration]
   localtime_s(tmp, timep);
   ^~~~~~~~~~~
Z:/mysql-connector-c-6.1.9-src/include/my_global.h: 在文件作用域：
Z:/mysql-connector-c-6.1.9-src/include/my_global.h:659:1: 错误：对‘gmtime_r’的静态声明出现在非静态声明之后
 {
/usr/include/time.h:79:12: 附注：‘gmtime_r’的上一个声明在此
 struct tm *_EXFUN(gmtime_r, (const time_t *__restrict,
            ^
```

### 问题9
继续编译，遇到如下问题
```bash
[  6%] Building C object extra/yassl/CMakeFiles/yassl.dir/__/__/client/get_password.c.obj
Z:/mysql-connector-c-6.1.9-src/client/get_password.c: 在文件作用域：
Z:/mysql-connector-c-6.1.9-src/client/get_password.c:49:19: 致命错误：conio.h：No such file or directory
 #include <conio.h>
                   ^
编译中断。
make[2]: *** [extra/yassl/CMakeFiles/yassl.dir/build.make:364：extra/yassl/CMakeFiles/yassl.dir/__/__/client/get_password.c.obj] 错误 1
make[1]: *** [CMakeFiles/Makefile2:215：extra/yassl/CMakeFiles/yassl.dir/all] 错误 2
make: *** [Makefile:152：all] 错误 2
```
`conio.h`这个文件没有，直接下载一个即可[http://files.cnblogs.com/files/oloroso/conio.zip](http://files.cnblogs.com/files/oloroso/conio.zip)
下载完成之后直接放在源码目录的`include`目录下即可。

### 问题10
```bash
[ 18%] Building CXX object extra/yassl/taocrypt/CMakeFiles/taocrypt.dir/src/random.cpp.obj
Z:/mysql-connector-c-6.1.9-src/extra/yassl/taocrypt/src/random.cpp: 在构造函数‘TaoCrypt::OS_Seed::OS_Seed()’中:
Z:/mysql-connector-c-6.1.9-src/extra/yassl/taocrypt/src/random.cpp:76:29: 错误：从类型‘TaoCrypt::OS_Seed::ProviderHandle* {aka long unsigned int*}’到类型‘HCRYPTPROV* {aka long long unsigned int*}’的转换无效 [-fpermissive]
     if(!CryptAcquireContext(&handle_, 0, 0, PROV_RSA_FULL,
                             ^~~~~~~~
```
直接强转即可。

### 问题11
```bash
[ 52%] Building C object mysys/CMakeFiles/mysys.dir/my_create.c.obj
......
Z:/mysql-connector-c-6.1.9-src/mysys/my_create.c:23:19: 致命错误：share.h：No such file or directory
 #include <share.h>
                   ^
编译中断。
make[2]: *** [mysys/CMakeFiles/mysys.dir/build.make:964：mysys/CMakeFiles/mysys.dir/my_create.c.obj] 错误 1
make[1]: *** [CMakeFiles/Makefile2:597：mysys/CMakeFiles/mysys.dir/all] 错误 2
make: *** [Makefile:152：all] 错误 2
```
直接打开`my_create.c`定位到`#include <share.h>`将其注释掉即可。

### 问题12

```bash
[ 38%] Building C object mysys/CMakeFiles/mysys.dir/my_fopen.c.obj
....
Z:/mysql-connector-c-6.1.9-src/mysys/my_fopen.c:130:35: 错误：‘_O_APPEND’未声明(在此函数内第一次使用)
                                   _O_APPEND | _O_TEXT)) == -1)
                                   ^~~~~~~~~
Z:/mysql-connector-c-6.1.9-src/mysys/my_fopen.c:130:35: 附注：每个未声明的标识符在其出现的函数内只报告一次
Z:/mysql-connector-c-6.1.9-src/mysys/my_fopen.c:130:47: 错误：‘_O_TEXT’未声明(在此函数内第一次使用)
                                   _O_APPEND | _O_TEXT)) == -1)
                                               ^~~~~~~
Z:/mysql-connector-c-6.1.9-src/mysys/my_fopen.c:136:7: 警告：隐式声明函数‘_dup2’ [-Wimplicit-function-declaration]
   if (_dup2(handle_fd, fd) < 0)
       ^~~~~
make[2]: *** [mysys/CMakeFiles/mysys.dir/build.make:1089：mysys/CMakeFiles/mysys.dir/my_fopen.c.obj] 错误 1
make[1]: *** [CMakeFiles/Makefile2:597：mysys/CMakeFiles/mysys.dir/all] 错误 2
make: *** [Makefile:152：all] 错误 2
```
将` _O_APPEND`和`_O_TEXT`改为`O_APPEND`和`O_TEXT`即可。

### 问题13

```bash
[ 39%] Building C object mysys/CMakeFiles/mysys.dir/my_gethwaddr.c.obj
.......
In file included from /usr/include/w32api/iprtrmib.h:12:0,
                 from /usr/include/w32api/iphlpapi.h:15,

                 from Z:/mysql-connector-c-6.1.9-src/mysys/my_gethwaddr.c:141:
/usr/include/w32api/mprapi.h: 在文件作用域：
/usr/include/w32api/mprapi.h:869:3: 错误：未知的类型名‘CERT_NAME_BLOB’
   CERT_NAME_BLOB *certificateNames;
   ^~~~~~~~~~~~~~
/usr/include/w32api/mprapi.h:891:3: 错误：未知的类型名‘CRYPT_HASH_BLOB’
   CRYPT_HASH_BLOB certBlob;
   ^~~~~~~~~~~~~~~
make[2]: *** [mysys/CMakeFiles/mysys.dir/build.make:1139：mysys/CMakeFiles/mysys.dir/my_gethwaddr.c.obj] 错误 1
make[1]: *** [CMakeFiles/Makefile2:597：mysys/CMakeFiles/mysys.dir/all] 错误 2
make: *** [Makefile:152：all] 错误 2
```
直接打开文件`/usr/include/w32api/iprtrmib.h`将第12行的`#include <mprapi.h>`给注释掉。

### 问题14
```bash
In file included from /usr/include/w32api/iphlpapi.h:15:0,
                 from Z:/mysql-connector-c-6.1.9-src/mysys/my_gethwaddr.c:141:
/usr/include/w32api/iprtrmib.h: 在文件作用域：
/usr/include/w32api/iprtrmib.h:81:17: 错误：‘MAX_INTERFACE_NAME_LEN’未声明(不在函数内)
   WCHAR wszName[MAX_INTERFACE_NAME_LEN];
                 ^~~~~~~~~~~~~~~~~~~~~~
make[2]: *** [mysys/CMakeFiles/mysys.dir/build.make:1139：mysys/CMakeFiles/mysys.dir/my_gethwaddr.c.obj] 错误 1
```
直接打开文件`/usr/include/w32api/iprtrmib.h`,在最前面添加宏定义`宏#define MAX_INTERFACE_NAME_LEN 512`


### 问题15
```bash
Z:/mysql-connector-c-6.1.9-src/mysys/my_getwd.c: 在文件作用域：
Z:/mysql-connector-c-6.1.9-src/mysys/my_getwd.c:25:17: 致命错误：dos.h：No such file or directory
 #include <dos.h>
                 ^
编译中断。
make[2]: *** [mysys/CMakeFiles/mysys.dir/build.make:1189：mysys/CMakeFiles/mysys.dir/my_getwd.c.obj] 错误 1
```
这里直接将`my_getwd.c`文件中的25、26行都注释掉（`include <dos.h>`和`include <direct.h>`）

### 问题16

```bash
Z:/mysql-connector-c-6.1.9-src/mysys/my_init.c:30:20: 致命错误：crtdbg.h：No such file or directory
 #include <crtdbg.h>
                    ^
编译中断。
make[2]: *** [mysys/CMakeFiles/mysys.dir/build.make:1239：mysys/CMakeFiles/mysys.dir/my_init.c.obj] 错误 1
```
直接注释掉这个头文件的包含

### 问题17
```bash
Z:/mysql-connector-c-6.1.9-src/mysys/my_init.c: 在函数‘my_end’中:
Z:/mysql-connector-c-6.1.9-src/mysys/my_init.c:189:4: 警告：隐式声明函数‘_CrtSetReportMode’ [-Wimplicit-function-declaration]
    _CrtSetReportMode( _CRT_WARN, _CRTDBG_MODE_FILE );
    ^~~~~~~~~~~~~~~~~
Z:/mysql-connector-c-6.1.9-src/mysys/my_init.c:189:23: 错误：‘_CRT_WARN’未声明(在此函数内第一次使用)
    _CrtSetReportMode( _CRT_WARN, _CRTDBG_MODE_FILE );
                       ^~~~~~~~~
Z:/mysql-connector-c-6.1.9-src/mysys/my_init.c:189:23: 附注：每个未声明的标识符在其出现的函数内只报告一次
Z:/mysql-connector-c-6.1.9-src/mysys/my_init.c:189:34: 错误：‘_CRTDBG_MODE_FILE’未声明(在此函数内第一次使用)
    _CrtSetReportMode( _CRT_WARN, _CRTDBG_MODE_FILE );
                                  ^~~~~~~~~~~~~~~~~
Z:/mysql-connector-c-6.1.9-src/mysys/my_init.c:190:4: 警告：隐式声明函数‘_CrtSetReportFile’ [-Wimplicit-function-declaration]
    _CrtSetReportFile( _CRT_WARN, _CRTDBG_FILE_STDERR );
    ^~~~~~~~~~~~~~~~~
Z:/mysql-connector-c-6.1.9-src/mysys/my_init.c:190:34: 错误：‘_CRTDBG_FILE_STDERR’未声明(在此函数内第一次使用)
    _CrtSetReportFile( _CRT_WARN, _CRTDBG_FILE_STDERR );
                                  ^~~~~~~~~~~~~~~~~~~
Z:/mysql-connector-c-6.1.9-src/mysys/my_init.c:191:23: 错误：‘_CRT_ERROR’未声明(在此函数内第一次使用)
    _CrtSetReportMode( _CRT_ERROR, _CRTDBG_MODE_FILE );
                       ^~~~~~~~~~
Z:/mysql-connector-c-6.1.9-src/mysys/my_init.c:193:23: 错误：‘_CRT_ASSERT’未声明(在此函数内第一次使用)
    _CrtSetReportMode( _CRT_ASSERT, _CRTDBG_MODE_FILE );
```
直接定位到`my_init.c`文件的188行，做如下修改
```c
// #if defined(_WIN32) 修改为下面语句
#if defined(_WIN32) && defined(_MSC_VER)
```

### 问题18
```bash
Z:/mysql-connector-c-6.1.9-src/mysys/my_lib.c: 在函数‘my_dir’中:
Z:/mysql-connector-c-6.1.9-src/mysys/my_lib.c:220:22: 错误：‘find’的存储大小未知
   struct _finddata_t find;
                      ^~~~
Z:/mysql-connector-c-6.1.9-src/mysys/my_lib.c:269:15: 警告：隐式声明函数‘_findfirst’ [-Wimplicit-function-declaration]
   if ((handle=_findfirst(tmp_path,&find)) == -1L)
               ^~~~~~~~~~
Z:/mysql-connector-c-6.1.9-src/mysys/my_lib.c:291:21: 错误：‘_A_HIDDEN’未声明(在此函数内第一次使用)
       if (attrib & (_A_HIDDEN | _A_SYSTEM))
                     ^~~~~~~~~
Z:/mysql-connector-c-6.1.9-src/mysys/my_lib.c:291:21: 附注：每个未声明的标识符在其出现的函数内只报告一次
Z:/mysql-connector-c-6.1.9-src/mysys/my_lib.c:291:33: 错误：‘_A_SYSTEM’未声明(在此函数内第一次使用)
       if (attrib & (_A_HIDDEN | _A_SYSTEM))
                                 ^~~~~~~~~
Z:/mysql-connector-c-6.1.9-src/mysys/my_lib.c:304:24: 错误：‘_A_RDONLY’未声明(在此函数内第一次使用)
         if (!(attrib & _A_RDONLY))
                        ^~~~~~~~~
Z:/mysql-connector-c-6.1.9-src/mysys/my_lib.c:306:22: 错误：‘_A_SUBDIR’未声明(在此函数内第一次使用)
         if (attrib & _A_SUBDIR)
                      ^~~~~~~~~
```
这个问题不是很好解决，在`/usr/include/`和`C:\Program Files (x86)\Microsoft SDKs\Windows\v7.1A\Include`下的头文件中都找不到相关定义。
既然是mingw下，那就可以使用非Windows的实现。
打开`my_lib.c`文件做如下修改：
```c
// 在26行左右位置的 #if defined(HAVE_DIRENT_H)前添加
#define HAVE_DIR_ENT_H 1
#include <sys/types.h>

// 将77行左右位置的 #if !defined(_WIN32)修改为
#if 1
```

### 问题19
```bash
Z:/mysql-connector-c-6.1.9-src/mysys/my_mkdir.c:23:20: 致命错误：direct.h：No such file or directory
 #include <direct.h>
                    ^
编译中断。
make[2]: *** [mysys/CMakeFiles/mysys.dir/build.make:1364：mysys/CMakeFiles/mysys.dir/my_mkdir.c.obj] 错误 1
```
将` #include <direct.h>`给注释掉即可


### 问题20
```bash
Z:/mysql-connector-c-6.1.9-src/mysys/my_mkdir.c: 在函数‘my_mkdir’中:
Z:/mysql-connector-c-6.1.9-src/mysys/my_mkdir.c:32:7: 错误：提供给函数‘mkdir’的实参太少
   if (mkdir((char*) dir))
       ^~~~~
```
这里直接将32行前的`#if defined(_WIN32)`修改为`#if defined(_WIN32) && defined(_MSC_VER)`即可

### 问题21
```bash
Z:/mysql-connector-c-6.1.9-src/mysys/my_thr_init.c:455:25: 错误：‘_CALL_REPORTFAULT’未声明(在此函数内第一次使用)
   _set_abort_behavior(0,_CALL_REPORTFAULT);
                         ^~~~~~~~~~~~~~~~~
```
`_CALL_REPORTFAULT`的定义在`stdlib.h`中，但是这个是在VS下才有，具体定义如下
```c
// Argument values for _set_abort_behavior().
#define _WRITE_ABORT_MSG  0x1 // debug only, has no effect in release
#define _CALL_REPORTFAULT 0x2
```
这里直接将其修改为`0x2`了。

### 问题22
```bash
Z:/mysql-connector-c-6.1.9-src/mysys/stacktrace.c: 在函数‘my_safe_puts_stderr’中:
Z:/mysql-connector-c-6.1.9-src/mysys/stacktrace.c:552:3: 错误：‘__try’未声明(在此函数内第一次使用)
   __try
   ^~~~~
Z:/mysql-connector-c-6.1.9-src/mysys/stacktrace.c:552:3: 附注：每个未声明的标识符在其出现的函数内只报告一次
Z:/mysql-connector-c-6.1.9-src/mysys/stacktrace.c:553:3: 错误：expected ‘;’ before ‘{’ token
   {
   ^
```
这是windows的seh异常捕捉不被支持，直接注释掉即可（修改my_safe_puts_stderr函数如下）
```c
void my_safe_puts_stderr(const char *val, size_t len)
{
  //__try
  {
    my_write_stderr(val, len);
    my_safe_printf_stderr("%s", "\n");
  }
  //__except(EXCEPTION_EXECUTE_HANDLER)
  {
  //  my_safe_printf_stderr("%s", "is an invalid string pointer\n");
  }
}
```

### 问题23
```bash
[ 43%] Building C object mysys/CMakeFiles/mysys.dir/lf_alloc-pin.c.obj
In file included from Z:/mysql-connector-c-6.1.9-src/include/lf.h:20:0,
                 from Z:/mysql-connector-c-6.1.9-src/mysys/lf_alloc-pin.c:98:
Z:/mysql-connector-c-6.1.9-src/include/my_atomic.h:64:4: 错误：#error Native atomics support not found!
 #  error Native atomics support not found!
    ^~~~~
```
原生原子变量支持没有找到，这里直接注释掉` #  error Native atomics support not found!`这句

### 问题24
```bash
Z:/mysql-connector-c-6.1.9-src/mysys/my_winfile.c:51:19: 致命错误：share.h：No such file or directory
 #include <share.h>
```
这里直接将文件开头出的`#ifdef _WIN32`改为`#if 0`算了，不然还会有一堆错误。


### 问题24
```bash
[ 81%] Building CXX object mysys_ssl/CMakeFiles/mysys_ssl.dir/crypt_genhash_impl.cc.obj
In file included from Z:/mysql-connector-c-6.1.9-src/mysys_ssl/crypt_genhash_impl.cc:32:0:
Z:/mysql-connector-c-6.1.9-src/include/m_string.h: 在函数‘longlong my_strtoll(const char*, char**, int)’中:
Z:/mysql-connector-c-6.1.9-src/include/m_string.h:115:38: 错误：‘_strtoi64’在此作用域中尚未声明
   return _strtoi64(nptr, endptr, base);
                                      ^
Z:/mysql-connector-c-6.1.9-src/include/m_string.h: 在函数‘ulonglong my_strtoull(const char*, char**, int)’中:
Z:/mysql-connector-c-6.1.9-src/include/m_string.h:124:39: 错误：‘_strtoui64’在此作用域中尚未声明
   return _strtoui64(nptr, endptr, base);
                                       ^
Z:/mysql-connector-c-6.1.9-src/include/m_string.h: 在函数‘char* my_strtok_r(char*, const char*, char**)’中:
Z:/mysql-connector-c-6.1.9-src/include/m_string.h:133:38: 错误：‘strtok_s’在此作用域中尚未声明
   return strtok_s(str, delim, saveptr);
                                      ^
Z:/mysql-connector-c-6.1.9-src/include/m_string.h: 在函数‘int native_strcasecmp(const char*, const char*)’中:
Z:/mysql-connector-c-6.1.9-src/include/m_string.h:143:25: 错误：‘_stricmp’在此作用域中尚未声明
   return _stricmp(s1, s2);
                         ^
Z:/mysql-connector-c-6.1.9-src/include/m_string.h: 在函数‘int native_strncasecmp(const char*, const char*, size_t)’中:
Z:/mysql-connector-c-6.1.9-src/include/m_string.h:153:29: 错误：‘_strnicmp’在此作用域中尚未声明
   return _strnicmp(s1, s2, n);
                             ^
make[2]: *** [mysys_ssl/CMakeFiles/mysys_ssl.dir/build.make:64：mysys_ssl/CMakeFiles/mysys_ssl.dir/crypt_genhash_impl.cc.obj] 错误 1
```

这里直接打开`m_string.h`文件，将其中的`#if defined _WIN32`后面都加上`&& defined(_MSC_VER)`即可。

### 问题25

```bash
[ 81%] Building CXX object mysys_ssl/CMakeFiles/mysys_ssl.dir/crypt_genhash_impl.cc.obj
In file included from Z:/mysql-connector-c-6.1.9-src/mysys_ssl/crypt_genhash_impl.cc:32:0:
Z:/mysql-connector-c-6.1.9-src/include/m_string.h: 在函数‘int native_strcasecmp(const char*, const char*)’中:
Z:/mysql-connector-c-6.1.9-src/include/m_string.h:145:27: 错误：‘_stricmp’在此作用域中尚未声明
   return strcasecmp(s1, s2);
                           ^
make[2]: *** [mysys_ssl/CMakeFiles/mysys_ssl.dir/build.make:64：mysys_ssl/CMakeFiles/mysys_ssl.dir/crypt_genhash_impl.cc.obj] 错误 1
```
这是上面修改没有完善的问题，这里直接自己实现。我这里直接抄写的linux下的实现(为避免重定义，可以在crypt_genhash_impl.cc中去实现，这里只引用一下即可)
```c
typedef unsigned char u_char;
static const u_char charmap[] = {  
		000, 001, 002, 003, 004, 005, 006, 007,  
		010, 011, 012, 013, 014, 015, 016, 017,  
		020, 021, 022, 023, 024, 025, 026, 027,  
		030, 031, 032, 033, 034, 035, 036, 037,  
		040, 041, 042, 043, 044, 045, 046, 047,  
		050, 051, 052, 053, 054, 055, 056, 057,  
		060, 061, 062, 063, 064, 065, 066, 067,  
		070, 071, 072, 073, 074, 075, 076, 077,  
		0100, 0141, 0142, 0143, 0144, 0145, 0146, 0147,  
		0150, 0151, 0152, 0153, 0154, 0155, 0156, 0157,  
		0160, 0161, 0162, 0163, 0164, 0165, 0166, 0167,  
		0170, 0171, 0172, 0133, 0134, 0135, 0136, 0137,  
		0140, 0141, 0142, 0143, 0144, 0145, 0146, 0147,  
		0150, 0151, 0152, 0153, 0154, 0155, 0156, 0157,  
		0160, 0161, 0162, 0163, 0164, 0165, 0166, 0167,  
		0170, 0171, 0172, 0173, 0174, 0175, 0176, 0177,  
		0200, 0201, 0202, 0203, 0204, 0205, 0206, 0207,  
		0210, 0211, 0212, 0213, 0214, 0215, 0216, 0217,  
		0220, 0221, 0222, 0223, 0224, 0225, 0226, 0227,  
		0230, 0231, 0232, 0233, 0234, 0235, 0236, 0237,  
		0240, 0241, 0242, 0243, 0244, 0245, 0246, 0247,  
		0250, 0251, 0252, 0253, 0254, 0255, 0256, 0257,  
		0260, 0261, 0262, 0263, 0264, 0265, 0266, 0267,  
		0270, 0271, 0272, 0273, 0274, 0275, 0276, 0277,  
		0300, 0301, 0302, 0303, 0304, 0305, 0306, 0307,  
		0310, 0311, 0312, 0313, 0314, 0315, 0316, 0317,  
		0320, 0321, 0322, 0323, 0324, 0325, 0326, 0327,  
		0330, 0331, 0332, 0333, 0334, 0335, 0336, 0337,  
		0340, 0341, 0342, 0343, 0344, 0345, 0346, 0347,  
		0350, 0351, 0352, 0353, 0354, 0355, 0356, 0357,  
		0360, 0361, 0362, 0363, 0364, 0365, 0366, 0367,  
		0370, 0371, 0372, 0373, 0374, 0375, 0376, 0377
	};  

int  
xxxxx_strcasecmp(const char *s1, const char *s2)
{  
    const u_char *cm = charmap,  
            *us1 = (const u_char *)s1,  
            *us2 = (const u_char *)s2;  
  
    while (cm[*us1] == cm[*us2++])  
        if (*us1++ == '\0')  
            return (0);  
    return (cm[*us1] - cm[*--us2]);  
}  
  
int  
xxxx_strncasecmp(const char *s1, const char *s2, size_t n)
{  
    if (n != 0) {  
        const u_char *cm = charmap,  
                *us1 = (const u_char *)s1,  
                *us2 = (const u_char *)s2;  
  
        do {  
            if (cm[*us1] != cm[*us2++])  
                return (cm[*us1] - cm[*--us2]);  
            if (*us1++ == '\0')  
                break;  
        } while (--n != 0);  
    }  
    return (0);  
}  
```

### 问题26
```bash
Z:/mysql-connector-c-6.1.9-src/mysys_ssl/my_default.cc: 在函数‘size_t my_get_system_windows_directory(char*, size_t)’中:
Z:/mysql-connector-c-6.1.9-src/mysys_ssl/my_default.cc:1344:58: 错误：‘stricmp’在此作用域中尚未声明
   if (count > 8 && stricmp(buffer+(count-8), "\\System32") == 0)
                                                          ^
make[2]: *** [mysys_ssl/CMakeFiles/mysys_ssl.dir/build.make:114：mysys_ssl/CMakeFiles/mysys_ssl.dir/my_default.cc.obj] 错误 1
```
这里将`stricmp`改为`strcasecmp`即可。