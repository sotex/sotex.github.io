---
layout:  post
title:  "linux下编译Zero C ICE"
date:  2017-02-27
categories:  linux
tags:  linux
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2017/02/27/6473334.html](http://www.cnblogs.com/oloroso/archive/2017/02/27/6473334.html)
## 0、简介
ZeroC ICE 是指ZeroC公司[(www.zeroc.com)](www.zeroc.com)的ICE（Internet Communications Engine）中间件平台。

目前ICE平台中包括`Ice`，`Ice-E`，`Ice Touch`。
**Ice**为主流平台设计，包括Windows和Linux，支持广泛的语言，包括C++，Java，C#（和其他.Net的语言，例如Visual Basic），Python，Ruby，PHP和ActionScript。也包括所有的ICE服务，例如Ice Grid，IceStorm等。
**Ice-E**是Ice在资源受限的平台上的一个实现，支持C++和嵌入式操作系统，例如Windows CE，Linux。Ice-E本身不包含任何服务，但是可以利用在Ice上提供的各种服务。因此，通过Ice-E，移动设备也能无缝的集成到分布式系统中。
**Ice Touch**是为iphone和ipod touch开发的版本，包括Object-C映射，支持Iphone OS，并为MAC OS X开发图形界面应用程序提供完整的Cocoa框架的访问。

本文由乌合之众瞎写，如有错漏之处,敬请指正。[http://www.cnblogs.com/oloroso/](http://www.cnblogs.com/oloroso/)

## 1、下载源码


```shell
git clone https://github.com/zeroc-ice/ice.git
```

## 2、编译和安装
先安装依赖项`libmcpp`、`openssl`、`lmdb`等。
```shell
sudo dnf install libmcpp-devel    openssl-devel lmdb-devel # defora下安装
sudo apt install libmcpp-dev libssl-dev # debian系下安装
```

**编译并安装**
```shell
make -j4    # 编译(时间比较久，可以把Makefile中对test的编译注释掉)
make prefix=/opt/zero-c-ice install
```
上面命令会编译所有支持语言的版本，如果不需要那么多，可以通过修改`config/Make.rules`来改变支持的语言。
打开`config/Make.rules`文件，跳到最后，修改`supported-languages`的值。
**例如：**
```shell
supported-languages ?= cpp python
```
或者在编译的时候直接指定语言。
```shell
make supported-languages='cpp python java js'
make supported-languages='cpp python java js' prefix=/opt/zero-c-ice install
```
----
如果需要支持`java`语言，在编译的时候会去下载`gradle`工具，所以需要确保能够正常访问[https://gradle.org/](https://gradle.org/)
如果需要支持`python`语言，需要安装`python开发包`(sudo dnf install python-devel)。
如果需要支持`javascript`语言，需要安装`npm`的(sudo dnf install npm)，此处会安装libuv/nodejs等。这里也需要能够正常连接到外网。
如果需要支持`ruby`语言，需要安装`ruby开发包`(sudo dnf install ruby-devel)。
如果需要支持`php`语言，需要安装`php开发包`(sudo dnf install ruby-php)。

## 3、测试
这里可以编写一个小的`slice`脚步测试一下。
**test.ice**
```ice
/*定义模块test*/
module test{
    /*定义自定义结构类型person*/
    struct person{
        /*字符串类型量 name*/
        string    name;    // 名字
        /*整型量 age*/
        int        age;    // 年龄
        /*浮点数 weight*/
        float    weight;    // 体重
    };
    /*定义一个接口，打印person信息*/
    interface print{
        /*定义打印person的方法*/
        void printperson(person p);
    };
};
```
使用`slice2cpp`来生成C++代码。
```shell
slice2cpp test.ice
```
`slice2cpp`工具怎么用，可以使用`slice2cpp --help`来查看。

## 4、补充 windows下编译ice
windows下编译比较简单。直接打开源码目录下的`cpp\msbuild`，找到对应版本的VS解决方案文件（例如我是使用的VS2015，则是`ice.v140.sln`），直接使用Visual Studio打开。然后直接编译即可。
编译的时候需要下载对应的nuget包，所以可能需要联网才行。
编译完成之后将在`cpp\lib`和`cpp\bin`目录下生成对应的库和可执行文件。