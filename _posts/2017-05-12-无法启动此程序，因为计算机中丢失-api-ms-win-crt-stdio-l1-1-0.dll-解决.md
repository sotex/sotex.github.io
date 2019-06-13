---
layout:  post
title:  "无法启动此程序，因为计算机中丢失 api-ms-win-crt-stdio-l1-1-0.dll 解决"
date:  2017-05-12
categories:  编程
tags:  编程
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2017/05/12/6846039.html](http://www.cnblogs.com/oloroso/archive/2017/05/12/6846039.html)
## 问题描述

最近用一台Windows Server 2012 R2系统的机器的时候碰到了这个问题。
![](http://images2015.cnblogs.com/blog/693958/201705/693958-20170512153452519-844850695.png)

因为在网上看了很多解决方案，都没有很好的解决。所以记录一下这个问题的解决。

之前使用`VS2013`编译出的程序，是没有这个问题的。这个问题仅仅出现在`VS2015`编译的程序上。

重新安装了一个 `Windows server 2008 R2`的虚拟机，然后安装了`vc_redist.exe`(VC2015x64版本)，运行程序是没有问题的。这个winserver2008的系统镜像是下载的微软原版的，所以这里猜测安装win server 2012的服务器安装的系统可能并不是完整的。

## 解决过程

通过在服务器上的`C:\Windows\System32`（64位系统System32下是64位dll，SysWOW64目录下是32位dll）下搜索也没有找到相关的`dll`文件。
根据网上的一些资料，解决的办法就是安装`VC运行时库`和`KB2999226`补丁。这个方法我尝试过了，但是没有效果。微软提供了[`WindowsUCRT.zip(Windows 10 通用 C 运行时 )`](https://download.microsoft.com/download/C/5/D/C5D68AA1-F62E-422A-9084-4AD85CEB8D4D/WindowsUCRT.zip)下载，里面包含多个操作系统下的补丁。

既然上面的方法可能无法解决，那就先看看具体的依赖情况
**使用VS2015自带dumpbin查看依赖**
```cmd
dumpbin /dependents uds_services.exe
Microsoft (R) COFF/PE Dumper Version 14.00.24218.2
Copyright (C) Microsoft Corporation.  All rights reserved.


Dump of file uds_services.exe

File Type: EXECUTABLE IMAGE

  Image has the following dependencies:

    uds_module_foundation.dll
    KERNEL32.dll
    MSVCP140.dll
    WS2_32.dll
    MSWSOCK.dll
    VCRUNTIME140.dll
    api-ms-win-crt-stdio-l1-1-0.dll
    api-ms-win-crt-heap-l1-1-0.dll
    api-ms-win-crt-convert-l1-1-0.dll
    api-ms-win-crt-runtime-l1-1-0.dll
    api-ms-win-crt-string-l1-1-0.dll
    api-ms-win-crt-environment-l1-1-0.dll
    api-ms-win-crt-math-l1-1-0.dll
    api-ms-win-crt-locale-l1-1-0.dll

  Summary

        5000 .data
        1000 .gfids
        5000 .pdata
       1E000 .rdata
        1000 .reloc
        1000 .rsrc
       49000 .text
        1000 .tls
```
在我本机上搜索`api-ms-win-crt`相关的文件，发现在4处地方都有找到
```
C:\Program Files (x86)\Windows Kits\10\Redist\ucrt\DLLs\x86\
C:\Program Files (x86)\Windows Kits\10\Redist\ucrt\DLLs\x64\
C:\Program Files (x86)\Microsoft Visual Studio\Installer\resources\app\ServiceHub\Services\Microsoft.VisualStudio.Setup.Service\
C:\Program Files (x86)\Mozilla Firefox\
```
使用PEtools工具，可以看到`Mozilla Firefox`目录下的是`VS2013`编译的版本，而我的程序是`VS2015`编译的64位版本，所以使用的是`C:\Program Files (x86)\Windows Kits\10\Redist\ucrt\DLLs\x64\`目录下的文件。

将这几处中的相关文件拷贝到程序目录之后，重新运行，还是有dll找不到的错误。
把所有`api-ms-win-crt-*...dll`文件都拷贝之后，报如下错误：
![](http://images2015.cnblogs.com/blog/693958/201705/693958-20170512155327160-1294783801.png)

因为已经没有dll找不到的问题了，所以对于这个问题就比较费解了。因为`dumpbin`并没有找出所有依赖的`dll`(比如上面没有找到api-ms-win-crt-utility-l1-1-0.dll，但这个是被依赖的)。
使用`Dependecy Walker`工具可以看出来，`__stdio_common_vfprintf`这个函数在`api-ms-win-crt-stdio-l1-1-0.dll`里面。
![](http://images2015.cnblogs.com/blog/693958/201705/693958-20170512160827832-467315025.png)

但是无法看出`api-ms-win-crt-stdio-l1-1-0.dll`依赖了那些项目。
所以考虑是不是还有`dll`没有拷贝进去。发现目录下有`ucrtbase.dll`这个文件，感觉这应该是所有这些dll的基础依赖。把它拷贝进去之后，便可以正常运行了。
经过试验，这个问题的原因在于没有成功安装`KB2999226`补丁，有些系统这个补丁是安装不上的。只要找到`ucrtbase.dll`这个文件，拷贝到程序目录下就可以了。


因为一台机器上同时按照`VS2013`和`VS2015`编译出的版本可能会有冲突，所以不适合把它拷贝到`System32`目录（火狐就是自带了）。
可以通过设置`Path`环境变量来设置加载的`dll`查找位置。因为`Windows`下依赖的dll查找顺序([Dynamic-Link Library Search Order](https://msdn.microsoft.com/en-us/library/windows/desktop/ms682586(v=vs.85).aspx))是最后一个从`Path`环境变量中查找的，从而可能导致找到的并不是你想要的。

```
# 这里将dll放置在当前路径下的api-ms-win-crt目录下
set Path=%Path%;%cd%\api-ms-win-crt
# 启动程序
start program
```

**补充**
[https://msdn.microsoft.com/en-us/library/windows/desktop/ms686203(v=vs.85).aspx](https://msdn.microsoft.com/en-us/library/windows/desktop/ms686203(v=vs.85).aspx)
Windows下dll默认加载路径顺序如下：
1. 应用程序所在的目录
2. SetDllDirectory所设置的路径，如果没有设置，就是当前工作路径(GetCurrentDirectory)
3. system目录，可通过 GetSystemDirectory获取。（通常是C:\Windows\system32）
4. 16位系统的目录。(16位程序使用的，通常是C:\Windows\system)
5. Windows目录，可通过GetWindowsDirectory获取。（C:\Windows）
6. PATH环境变量中指定的路径。（PATH环境变量中路径的搜索顺序是在前面的优先，且系统环境变量优先于用户环境变量）

可以在程序中使用`SetDllDirectory`来指定DLL加载的目录，但`SetDllDirectory`的每一次调用都会替换掉之前调用的结果。可以使用`AddDllDirectory`来添加多个DLL加载路径。
可参考：http://www.cnblogs.com/tocy/p/windows_dll_searth_path.html