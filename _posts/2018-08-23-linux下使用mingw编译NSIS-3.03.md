---
layout:  post
title:  "linux下使用mingw编译NSIS-3.03"
date:  2018-08-23
categories:  linux
tags:  linux
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2018/08/23/9523351.html](http://www.cnblogs.com/oloroso/archive/2018/08/23/9523351.html)
# 简述

最近在研究使用`NSIS`做安装包，语法不算复杂，插件也很多，中文资料也不少，还挺好用的。先后用NSIS做出了安装和卸载需要输入密码，通过自定义页面实现安装时候选择多个目录、安装的时候输入配置文件信息，禁止在某些平台或环境下安装，检测是否已经安装或正在运行等，稍后将把这些都放出来，做个记录。

有一个问题就是`NSIS`打包的文件可以直接使用`7zip`解压，安装过程做的事情就被跳过了。为了解决这个问题，我想修改一下`NSIS`的源码，来使得打包的程序无法使用`7zip`等软件解压。这里记录一下编译过程。

修改过的文件及编译好的文件下载[nsis-3.03-src_修改.7z](https://files.cnblogs.com/files/oloroso/nsis-3.03-src_%E4%BF%AE%E6%94%B9.7z)

# 准备工作
我是在`linux`下使用`mingw32`进行编译的，所以先要安装`mingw32`。

然后是下载`zlib`库，我是在这里下载的[https://jaist.dl.sourceforge.net/project/mingw/MinGW/Extension/zlib/zlib-1.2.3-1-mingw32/libz-1.2.3-1-mingw32-dev.tar.gz](https://jaist.dl.sourceforge.net/project/mingw/MinGW/Extension/zlib/zlib-1.2.3-1-mingw32/libz-1.2.3-1-mingw32-dev.tar.gz)，这个说不定哪天就过期了，这是项目的页面[https://sourceforge.net/projects/mingw/files/MinGW/Extension/zlib/](https://sourceforge.net/projects/mingw/files/MinGW/Extension/zlib/)。

下载之后解压，将其中`lib`目录下的文件拷贝到`/usr/lib/gcc/i686-w64-mingw32/6.3-win32/lib`目录下，将`include`目录下的文件拷贝到`/usr/lib/gcc/i686-w64-mingw32/6.3-win32/include`目录下。

然后是下载`NSIS-3.03`的源码，地址在这里[https://jaist.dl.sourceforge.net/project/nsis/NSIS 3/3.03/nsis-3.03-src.tar.bz2](https://jaist.dl.sourceforge.net/project/nsis/NSIS 3/3.03/nsis-3.03-src.tar.bz2)，下载之后解压即可。

因为`NSIS`使用`scons`进行构建，所以还需要安装`python2.7`和`scons`工具。

# 编译过程
关于`NSIS`的编译，在这里[Appendix G: Building NSIS](http://nsis.sourceforge.net/Docs/AppendixG.html)有部分介绍。

准备工作做好后，使用下面命令进行构建。
```bash
scons SKIPSTUBS=all XGCC_W32_PREFIX=i686-w64-mingw32-    SKIPPLUGINS=all SKIPUTILS=all SKIPMISC=all NSIS_CONFIG_CONST_DATA_PATH=no PREFIX=./build
```
如果没有问题的话，正常会构建成功。但是会发现一个问题，`build/urelease/makensis`下面没有生成`makensis.exe`文件，只有一个`makensis`文件，而且使用`readelf`查看，这是一个`ELF`文件，而不是`PE`格式文件。
使用PETool查看结果如下：
![](https://images2018.cnblogs.com/blog/693958/201808/693958-20180823131434771-1419931166.jpg)

这个错误的原因在这里：
![](https://images2018.cnblogs.com/blog/693958/201808/693958-20180823131651828-1729400071.png)

根据这个报错，查看了`Sconstruct`文件后，找到错误的原因，是因为其中一个环境变量没有设置对。
打开`nsis-3.03-src/SCons/Config/gnu`文件，找到364行，在下面添加一行内容`makensis_env.Replace(CXX = stub_env['CXX'])`。

![](https://images2018.cnblogs.com/blog/693958/201808/693958-20180823132023556-327324227.jpg)

然后重新执行构建命令。

解决完`g++`的问题后，继续编译遇到下面的问题。

**下面有些修改其实是因为`PLATFROM`没有识别或者设置为`win32`的原因。**

## 错误 Source/scriptpp.cpp:1054:93: error: call to non-constexpr function 'size_t wcslen(const wchar_t*)'

![](https://images2018.cnblogs.com/blog/693958/201808/693958-20180823132331657-1189255522.jpg)
```bash
i686-w64-mingw32-g++ -o build/urelease/makensis/scriptpp.o -c -Wno-non-virtual-dtor -Wall -O2 -DNSISCALL= -D_UNICODE -DUNICODE -DMAKENSIS -D_WIN32_IE=0x0500 -Ibuild/urelease/config Source/scriptpp.cpp
In file included from /usr/share/mingw-w64/include/minwindef.h:163:0,
                 from /usr/share/mingw-w64/include/windef.h:8,
                 from /usr/share/mingw-w64/include/windows.h:69,
                 from Source/Platform.h:38,
                 from Source/scriptpp.cpp:17:
Source/scriptpp.cpp: In member function 'int CEXEBuild::pp_finalize(LineParser&)':
Source/scriptpp.cpp:1054:93: error: call to non-constexpr function 'size_t wcslen(const wchar_t*)'
   newcmd = (struct postbuild_cmd*) (new BYTE[FIELD_OFFSET(struct postbuild_cmd, cmd[_tcsclen(cmdstr)+1])]);
```
这里`FIELD_OFFSET`宏的参数应该是`结构体的名称`和`成员的名称`，但这里明显不是，结合代码上下文来看，我把这里给修改为了
```cpp
 newcmd = (struct postbuild_cmd*) (new BYTE[FIELD_OFFSET(struct postbuild_cmd, cmd)* (_tcsclen(cmdstr)+1)]);
```
![](https://images2018.cnblogs.com/blog/693958/201808/693958-20180823143548030-2079299031.jpg)

没有仔细研究它的代码，所以这里就多分配一点了。

继续编译。

## 错误 Source/util.cpp:1072:11: error: jump to label 'finalwrite' [-fpermissive]

![](https://images2018.cnblogs.com/blog/693958/201808/693958-20180823133541480-362085863.jpg)

```bash
i686-w64-mingw32-g++ -o build/urelease/makensis/util.o -c -Wno-non-virtual-dtor -Wall -O2 -DNSISCALL= -D_UNICODE -DUNICODE -DMAKENSIS -D_WIN32_IE=0x0500 -Ibuild/urelease/config Source/util.cpp
Source/util.cpp: In function 'int RunChildProcessRedirected(LPCWSTR, LPCWSTR, bool)':
Source/util.cpp:1072:11: error: jump to label 'finalwrite' [-fpermissive]
         { finalwrite:
           ^~~~~~~~~~
Source/util.cpp:1085:25: note:   from here
         if (cchwb) goto finalwrite; // End of stream without a ending newline, write out the remaining data.
                         ^~~~~~~~~~
Source/util.cpp:1033:17: note:   skips initialization of 'DWORD i'
       for(DWORD i = 0; i < cbRead;)
                 ^
Source/util.cpp: At global scope:
Source/util.cpp:1159:15: warning: 'void* NSISRT_GetConsoleScreenHandle()' defined but not used [-Wunused-function]
 static HANDLE NSISRT_GetConsoleScreenHandle()
               ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~
scons: *** [build/urelease/makensis/util.o] Error 1
scons: building terminated because of errors.
```
看来一下这里的代码，比较繁杂，就不改了，直接使用下面的命令先编译通过
```
i686-w64-mingw32-g++ -o build/urelease/makensis/util.o -c -fpermissive  -Wno-non-virtual-dtor -Wall -O2 -DNSISCALL= -D_UNICODE -DUNICODE -DMAKENSIS -D_WIN32_IE=0x0500 -Ibuild/urelease/config Source/util.cpp
```
![](https://images2018.cnblogs.com/blog/693958/201808/693958-20180823134315250-31373064.jpg)

**这里需要再次修改一下`nsis-3.03-src/SCons/Config/gnu`文件，在119行下添加一行`makensis_env.Append(CXXFLAGS = ['-fpermissive'])`**
![](https://images2018.cnblogs.com/blog/693958/201808/693958-20180823134450904-1838861984.jpg)

修改直接继续编译。

## 错误 build/urelease/makensis/crc32.o: file not recognized: File format not recognized

解决上面的错误之后，编译就没有问题了，一直到最后的链接这一步，又有点小问题了。
```bash
gcc -o build/urelease/makensis/crc32.o -c -Wall -O2 -DNSISCALL= -D_UNICODE -DUNICODE -DMAKENSIS -D_WIN32_IE=0x0500 -Ibuild/urelease/config Source/crc32.c
i686-w64-mingw32-g++ -o build/urelease/makensis/makensis -Wl,-Map,build/urelease/makensis/makensis.map -s -pthread build/urelease/makensis/build.o build/urelease/makensis/clzma.o build/urelease/makensis/crc32.o build/urelease/makensis/DialogTemplate.o build/urelease/makensis/dirreader.o build/urelease/makensis/fileform.o build/urelease/makensis/growbuf.o build/urelease/makensis/icon.o build/urelease/makensis/lang.o build/urelease/makensis/lineparse.o build/urelease/makensis/makenssi.o build/urelease/makensis/manifest.o build/urelease/makensis/mmap.o build/urelease/makensis/Plugins.o build/urelease/makensis/ResourceEditor.o build/urelease/makensis/ResourceVersionInfo.o build/urelease/makensis/BinInterop.o build/urelease/makensis/script.o build/urelease/makensis/scriptpp.o build/urelease/makensis/ShConstants.o build/urelease/makensis/strlist.o build/urelease/makensis/tokens.o build/urelease/makensis/tstring.o build/urelease/makensis/utf.o build/urelease/makensis/util.o build/urelease/makensis/winchar.o build/urelease/makensis/writer.o build/urelease/makensis/bzip2/blocksort.o build/urelease/makensis/bzip2/bzlib.o build/urelease/makensis/bzip2/compress.o build/urelease/makensis/bzip2/huffman.o build/urelease/makensis/7zip/7zGuids.o build/urelease/makensis/7zip/7zip/Common/OutBuffer.o build/urelease/makensis/7zip/7zip/Common/StreamUtils.o build/urelease/makensis/7zip/7zip/Compress/LZ/LZInWindow.o build/urelease/makensis/7zip/7zip/Compress/LZMA/LZMAEncoder.o build/urelease/makensis/7zip/7zip/Compress/RangeCoder/RangeCoderBit.o build/urelease/makensis/7zip/Common/Alloc.o build/urelease/makensis/7zip/Common/CRC.o -lpthread -lz
build/urelease/makensis/crc32.o: file not recognized: File format not recognized
collect2: error: ld returned 1 exit status
scons: *** [build/urelease/makensis/makensis] Error 1
scons: building terminated because of errors.
```
![](https://images2018.cnblogs.com/blog/693958/201808/693958-20180823134745137-296995545.jpg)

看我选中的部分，这里编译`crc32.c`的时候使用的是`gcc`而不是`mingw32-gcc`，所以这里的`crc32.o`文件是不能被正常识别的。
因为这是编译`makensis`部分出现的问题，所以这里还是环境没有设置对。
这里再次修改`nsis-3.03-src/SCons/Config/gnu`文件，在之前改`CXX`变量的前面吧`CC`也改了。这里在一开始就应该想到两个都应该重新设置的。
![](https://images2018.cnblogs.com/blog/693958/201808/693958-20180823135155988-1317011090.jpg)

修改之后继续编译。

## 错误 build/urelease/makensis/BinInterop.o:BinInterop.cpp:(.text+0xf5): undefined reference to 'GetFileVersionInfoSizeW@8' 等

这是最后链接的一点小问题
```
build/urelease/makensis/BinInterop.o:BinInterop.cpp:(.text+0xf5): undefined reference to `GetFileVersionInfoSizeW@8'
build/urelease/makensis/BinInterop.o:BinInterop.cpp:(.text+0x136): undefined reference to `GetFileVersionInfoW@16'
build/urelease/makensis/BinInterop.o:BinInterop.cpp:(.text+0x183): undefined reference to `VerQueryValueW@16'
```
这三个函数找不到，其实是没有链接`versions.lib`的原因，这里手动添加一下，执行链接即可。
![](https://images2018.cnblogs.com/blog/693958/201808/693958-20180823135706749-1482525367.jpg)

这里也可以修改`nsis-3.03-src/Source/SConscript`文件，在`libs`中添加`version`即可。
![](https://images2018.cnblogs.com/blog/693958/201808/693958-20180823143012714-1593711542.jpg)

这时候编译出来的`makensis`就是一个有效的`PE`文件了。

![](https://images2018.cnblogs.com/blog/693958/201808/693958-20180823135830720-1201061587.jpg)