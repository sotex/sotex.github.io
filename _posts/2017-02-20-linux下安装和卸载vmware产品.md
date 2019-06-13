---
layout:  post
title:  "linux下安装和卸载vmware产品"
date:  2017-02-20
categories:  linux
tags:  linux
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2017/02/20/6419899.html](http://www.cnblogs.com/oloroso/archive/2017/02/20/6419899.html)
## 1、安装
一般的发行版都不会带有vmware，所以通常是下载安装包来安装。
具体的可以见 http://www.cnblogs.com/oloroso/p/5845227.html

## 2、卸载
这里主要说的就是卸载，因为它不是通过包管理工具安装的，所以不能在包管理工具里面卸载。

### 2.1、先查看安装了vmware的哪些Product(产品)。
通过以下命令来查询
```shell
┌─(~)───────────────────────────────────----──────────(o@o-pc:pts/1)─┐
└─(15:38:48)──> vmware-installer -l                 1 ↵ ──(一,2月20)─┘

Product Name         Product Version     
==================== ====================
vmware-player        12.5.2.4638234
```
我这里只安装了`vmware-player`，所以这里只有这一个。
### 2.2、卸载
查询到安装的产品之后，就可以卸载了。
```shell
sudo vmware-installer --uninstall-product vmware-player
```
![](http://images2015.cnblogs.com/blog/693958/201702/693958-20170220154840288-463248616.png)
![](http://images2015.cnblogs.com/blog/693958/201702/693958-20170220154849445-160413496.png)

因为我这里只安装了这一个产品，所以卸载之后`vmware-installer`工具也一起卸载了。