---
layout:  post
title:  "Windows下 VS2015编译ForestDB"
date:  2017-01-19
categories:  编程
tags:  编程
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2017/01/19/6305615.html](http://www.cnblogs.com/oloroso/archive/2017/01/19/6305615.html)
# VS2015编译ForestDB

ForestDB 是一个快速的 Key-Value 存储引擎，基于层次B +树单词查找树。由 Couchbase 缓存和存储团队开发。

## 1、下载forestdb源码

```shell
git clone https://github.com/couchbase/forestdb.git
```

## 2、使用CMAKE生成VS工程

打开cmd窗口(最好使用`VS2015开发人员命令提示`)，进入源码目录，执行下面命令
```shell
mkdir msvc14	# 创建构建目录
cd msvc14		# 进入构建目录
# 生成VS项目(D:\Libs\forsetdb是编译后安装路径)
cmake -DCMAKE_INSTALL_PREFIX=D:\Libs\forsetdb -G "Visual Studio 14 Win64" ..
```

## 3、编译ForestDB
打开`VS2015 x64 本机工具命令提示`(或VS2015开发人员命令提示)工具，执行下面命令进行编译64位release版本的ForestDB库
```shell
msbuild ALL_BUILD.vcproject /p:configuration=release
```
编译完成后使用下面命令进行安装
```shell
msbuild INSTALL.vcproject /p:configuration=release
```
安装完成后可以到前面设置的安装路径(D:\Libs\forsetdb)去查看生成的库。
```shell
D:\Libs\forsetdb>tree /f
Folder PATH listing for volume work
Volume serial number is 00000001 000D:5E54
D:.
├───bin
│       forestdb.dll
│       forestdb_dump.exe
│
├───include
│   └───libforestdb
│           fdb_errors.h
│           fdb_types.h
│           forestdb.h
│
└───lib
        forestdb.lib
```