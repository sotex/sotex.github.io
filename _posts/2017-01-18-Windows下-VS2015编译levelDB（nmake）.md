---
layout:  post
title:  "Windows下 VS2015编译levelDB（nmake）"
date:  2017-01-18
categories:  编程
tags:  编程
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2017/01/18/6297517.html](http://www.cnblogs.com/oloroso/archive/2017/01/18/6297517.html)
# VS2015编译levelDB

Leveldb是一个google实现的非常高效的kv数据库，非常适合嵌入到程序中。如果有简单的key-value数据库需求，而又想使用一个数据库服务的话，levelDB是非常合适的。(BerkeleyDB和forestdb也不错)。

本文不适用于VS2013及更低版本。



## 1、下载level源码
直接使用git克隆一个即可
```shell
git clone https://github.com/google/leveldb.git
```

## 2、切换到windows分支
进入leveldb目录，执行下面操作

```shell
git checkout origin/windows
```
现在的`leveldb`版本是`1.19`但是Windows版本为`1.17`。


## 3、源码修改
切换到windows分支后，还需要修改两处地方。

1、修改`db/c.cc`文件
打开`db/c.cc`文件，将第八行位置修改如下
```c
#ifndef WIN32
#include <unistd.h>
#endif
```

2、修改`port/port.h`文件
在如下代码(17、18行)
```c
#elif defined(LEVELDB_PLATFORM_ANDROID)
#  include "port/port_android.h"
```
后面添加
```c
#elif defined(LEVELDB_PLATFORM_WINDOWS)  
#  include "port/port_win.h"  
```

3、修改`port/port_win.h`文件
将第四行的宏定义给注释掉
```c
#define snprintf _snprintf	// 注释掉此句
```
因为VS2015中已经实现了`snprintf`的定义，所以不需要这个了。
如果不去掉，编译的时候将出现以下错误。
```bat
        cl -c -nologo -Zc:wchar_t -FS -Zc:strictStrings -Zc:throwingNew  -Zi -MDd -GR -W3 -w34100 -w34189 -w44996 -w44456 -w44457 -w44458 -wd4577  -EHsc  -DLEVELDB_PLATFORM_WINDOWS -DOS_WIN -DWIN32 -DWIN64 -DNDEBUG -D_CRT_SECURE_NO_WARNINGS -I. -I.\include -IC:\Boost\include\boost-1_62 -Fobuild\obj\ @C:\Users\o\AppData\Local\Temp\nm35AF.tmp
port_win.cc
C:\Program Files (x86)\Windows Kits\10\include\10.0.14393.0\ucrt\stdio.h(1925): warning C4005: 'snprintf': macro redefinition
D:\work_code\DataServices\3rd\LevelDB\port/port_win.h(34): note: see previous definition of 'snprintf'
C:\Program Files (x86)\Windows Kits\10\include\10.0.14393.0\ucrt\stdio.h(1927): fatal error C1189: #error:  Macro definition of snprintf conflicts with Standard Library function declaration
```

## 3、添加Makefile.vc文件
直接拷贝下面的内容，在`leveldb`目录下创建`Makefile.vc`文件，粘贴过去。

注意，需要修改下面`INCPATH`和`LIBS`两处，需要修改为正确的`boost`库路径。

下面的`SOURCES`中的`.\db\db_bench.cc`、`.\util\testutil.cc`、`.\util\win_logger.cc`
以及`OBJECTS`中的`$(OBJECTS_DIR)\db_bench.obj`、`$(OBJECTS_DIR)\testutil.obj`、`$(OBJECTS_DIR)\win_logger.obj`实际上是不需要的，可以移除。但是放在这里也没有什么关系，可以留着。

如果需要启用`Snappy`压缩，那么还需要在`DEFINES`处添加`-DSNAPPY`，同时还需要在`INCPATH`和`LIBS`处指定`libsnappy`的`include`和库文件路径。

```Makefile
#############################################################################

MAKEFILE      = Makefile

####### 编译器，工具和选项设置

CC            = cl
CXX           = cl
DEFINES       = -DLEVELDB_PLATFORM_WINDOWS -DOS_WIN -DWIN32 -DWIN64 -DNDEBUG -D_CRT_SECURE_NO_WARNINGS
				# 如果需要编译debug版本，只需要将上面的 -DNDEBUG 去掉即可
CXXFLAGS      = -nologo -Zc:wchar_t -FS -Zc:strictStrings -Zc:throwingNew \
				-Zi -MT -O2 -GR -W3 -w34100 -w34189 -w44996 -w44456 -w44457 -w44458 -wd4577 \
				-EHsc  $(DEFINES)
				# 如果需要编译debug版本，可以将上面的 -O2 去掉，-W3改为-W1
				# -MT表示使用运行时库的多线程静态版本，可以根据需要改为-MTd/-MD/-MDd
INCPATH       = -I. -I.\include -IC:\Boost\include\boost-1_62
LINKER        = lib
				# 如果要生成dll，上面的改为 link
LFLAGS        =  /NOLOGO /MACHINE:X64
				# 如果要生成32位版本，上的X64改为X86
				# 如果是生成dll，上面还应该添加 /DYNAMICBASE /NXCOMPAT /INCREMENTAL:NO /DLL /SUBSYSTEM:WINDOWS /MANIFEST:embed

LIBS          = /LIBPATH:C:\Boost\lib64-msvc-14.0
IDC           = idc
IDL           = midl
ZIP           = zip -r -9
COPY          = copy /y
COPY_FILE     = copy /y
COPY_DIR      = xcopy /s /q /y /i
DEL_FILE      = del
DEL_DIR       = rmdir
MOVE          = move
CHK_DIR_EXISTS= if not exist
MKDIR         = mkdir
INSTALL_FILE    = copy /y
INSTALL_PROGRAM = copy /y
INSTALL_DIR     = xcopy /s /q /y /i

####### 输出目录设置

OBJECTS_DIR   = build\obj

####### 文件列表
SOURCES		= 	.\db\builder.cc \
				.\db\c.cc \
				.\db\dbformat.cc \
				.\db\db_bench.cc \
				.\db\db_impl.cc \
				.\db\db_iter.cc \
				.\db\filename.cc \
				.\db\log_reader.cc \
				.\db\log_writer.cc \
				.\db\memtable.cc \
				.\db\repair.cc \
				.\db\table_cache.cc \
				.\db\version_edit.cc \
				.\db\version_set.cc \
				.\db\write_batch.cc \
				.\port\port_win.cc \
				.\table\block.cc \
				.\table\block_builder.cc \
				.\table\format.cc \
				.\table\iterator.cc \
				.\table\merger.cc \
				.\table\table.cc \
				.\table\table_builder.cc \
				.\table\two_level_iterator.cc \
				.\util\arena.cc \
				.\util\cache.cc \
				.\util\coding.cc \
				.\util\comparator.cc \
				.\util\crc32c.cc \
				.\util\env.cc \
				.\util\env_boost.cc \
				.\util\hash.cc \
				.\util\histogram.cc \
				.\util\logging.cc \
				.\util\options.cc \
				.\util\status.cc \
				.\util\testutil.cc \
				.\util\win_logger.cc \

OBJECTS		=	$(OBJECTS_DIR)\builder.obj \
				$(OBJECTS_DIR)\c.obj \
				$(OBJECTS_DIR)\dbformat.obj \
				$(OBJECTS_DIR)\db_bench.obj \
				$(OBJECTS_DIR)\db_impl.obj \
				$(OBJECTS_DIR)\db_iter.obj \
				$(OBJECTS_DIR)\filename.obj \
				$(OBJECTS_DIR)\log_reader.obj \
				$(OBJECTS_DIR)\log_writer.obj \
				$(OBJECTS_DIR)\memtable.obj \
				$(OBJECTS_DIR)\repair.obj \
				$(OBJECTS_DIR)\table_cache.obj \
				$(OBJECTS_DIR)\version_edit.obj \
				$(OBJECTS_DIR)\version_set.obj \
				$(OBJECTS_DIR)\write_batch.obj \
				$(OBJECTS_DIR)\port_win.obj \
				$(OBJECTS_DIR)\block.obj \
				$(OBJECTS_DIR)\block_builder.obj \
				$(OBJECTS_DIR)\format.obj \
				$(OBJECTS_DIR)\iterator.obj \
				$(OBJECTS_DIR)\merger.obj \
				$(OBJECTS_DIR)\table.obj \
				$(OBJECTS_DIR)\table_builder.obj \
				$(OBJECTS_DIR)\two_level_iterator.obj \
				$(OBJECTS_DIR)\arena.obj \
				$(OBJECTS_DIR)\cache.obj \
				$(OBJECTS_DIR)\coding.obj \
				$(OBJECTS_DIR)\comparator.obj \
				$(OBJECTS_DIR)\crc32c.obj \
				$(OBJECTS_DIR)\env.obj \
				$(OBJECTS_DIR)\env_boost.obj \
				$(OBJECTS_DIR)\hash.obj \
				$(OBJECTS_DIR)\histogram.obj \
				$(OBJECTS_DIR)\logging.obj \
				$(OBJECTS_DIR)\options.obj \
				$(OBJECTS_DIR)\status.obj \
				$(OBJECTS_DIR)\testutil.obj \
				$(OBJECTS_DIR)\win_logger.obj \

DESTDIR        = build
TARGET         = leveldb.lib
DESTDIR_TARGET = build\leveldb.lib


####### 隐式规则

.SUFFIXES: .c .cpp .cc .cxx

## util目录
{.\util}.cc{build\obj\}.obj::
	$(CXX) -c $(CXXFLAGS) $(INCPATH) -Fo$(OBJECTS_DIR)\ @<<
	$<
<<

## table目录
{.\table}.cc{build\obj\}.obj::
	$(CXX) -c $(CXXFLAGS) $(INCPATH) -Fo$(OBJECTS_DIR)\ @<<
	$<
<<

## port目录
{.\port}.cc{build\obj\}.obj::
	$(CXX) -c $(CXXFLAGS) $(INCPATH) -Fo$(OBJECTS_DIR)\ @<<
	$<
<<

## doc目录
{.\doc}.cc{build\obj\}.obj::
	$(CXX) -c $(CXXFLAGS) $(INCPATH) -Fo$(OBJECTS_DIR)\ @<<
	$<
<<

## helpers目录
{.\helpers}.cc{build\obj\}.obj::
	$(CXX) -c $(CXXFLAGS) $(INCPATH) -Fo$(OBJECTS_DIR)\ @<<
	$<
<<

## db目录
{.\db}.cc{build\obj\}.obj::
	$(CXX) -c $(CXXFLAGS) $(INCPATH) -Fo$(OBJECTS_DIR)\ @<<
	$<
<<

####### 构建规则

first: all
all: $(DESTDIR_TARGET)
$(DESTDIR_TARGET):  $(OBJECTS) FORCE
	$(LINKER) $(LFLAGS) /OUT:$(DESTDIR_TARGET) @<<
$(OBJECTS) $(LIBS)
<<

clean:
	$(DEL_FILE) /q $(OBJECTS)
	$(DEL_FILE) /q $(DESTDIR_TARGET)
	$(DEL_DIR) /s /q $(DESTDIR)

install: 
uninstall: 
	
check: first

# 因为windows下的mkdir没有-p参数，所以必须先判断目录是否存在
FORCE:
	if not exist $(DESTDIR) $(MKDIR) $(DESTDIR)
	if not exist $(OBJECTS_DIR) $(MKDIR) $(OBJECTS_DIR)
```

## 4、编译
打开`VS 2015 开发人员命令提示`工具，切换到`leveldb`目录。
执行下面操作
```shell
nmake -f Makefile.vc
```
如果没包错误的话，编译完成后会在`build`目录下生成`leveldb.lib`文件。

因为`leveldb`没有使用`__declspec(dllexport)`导出接口，所以这里如果改为生成动态库，不会生成`.lib`文件。