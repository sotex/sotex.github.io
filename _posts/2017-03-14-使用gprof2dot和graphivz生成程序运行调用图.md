---
layout:  post
title:  "使用gprof2dot和graphivz生成程序运行调用图"
date:  2017-03-14
categories:  编程
tags:  编程
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2017/03/14/6548189.html](http://www.cnblogs.com/oloroso/archive/2017/03/14/6548189.html)
# 使用gprof2dot和graphivz生成程序运行调用图

`gprof2dot`是一个将`gprof`生成的输出转换为`dot`脚本的工具。通过给定一个`gprof`的输出文件，将其转换为生成程序调用图的`dot`脚本。`dot`脚本可以生成图像来进行查看。

## 1、下载gprof2dot工具
gprof2dot工具由JoséFronseca维护，并托管在Google代码（http://code.google.com/p/jrfonseca/w/list）,但是这个已经无法访问了。这里提供一个github的地址[https://github.com/jrfonseca/gprof2dot.git](https://github.com/jrfonseca/gprof2dot.git)。

```bash
git clone https://github.com/jrfonseca/gprof2dot.git
```
这个工具需要`python`环境的支持，请提前安装好`python3`。


## 2、使用gprof生成概要分析数据

### 2.1 gprof简介
Gprof 是GNU gnu binutils工具之一，默认情况下linux系统当中都带有这个工具。
- 1. 可以显示“flat profile”，包括每个函数的调用次数，每个函数消耗的处理器时间。
- 2. 可以显示“Call graph”，包括函数的调用关系，每个函数调用花费了多少时间。
- 3. 可以显示“注释的源代码”－－是程序源代码的一个副本，标记有程序中每行代码的执行次数。

### 2.2 使用gprof生成分析数据

1、在编译的时候使用`-pg`选项，可以写入makefile中。（链接时不需要）
```bash
# 普通程序编译
$ gcc -pg -g source.c -o binary
# cuda程序编译
$ nvcc -Xcompiler“-g -pg”-g -pg -arch = sm_20 source.cu -o binary
```

2、执行编译出的程序。
执行方式与没有加`pg`选项前是一样的，执行后将在程序运行目录下生成`gmon.out`文件。这是一个二进制文件，需要进一步分析生成文本文件。如果已经存在了`gmon.out`文件，将会被覆盖。
```bash
./binary	# 必须先运行一遍生成gmon.out文件，后面才能使用gprof工具继续操作
```

3、用`gprof`工具生成概要分析数据文件。
```bash
$ gprof ./binary> gprof_output.txt
```




## 3、生成调用图
配置好`gprof2dot`运行环境后，就可以使用其来生成`dot`脚本。
```bash
$ graph2dot.py gprof_output.txt> call_graph.dot
```
我这里是直接使用`git`克隆下来的，提前安装好了python3和graphivz`，所以直接运行即可。

## 4、生成可视化的图形
这里需要安装好`Graphivz`软件。
Windows下可以直接下载[安装包](http://www.graphviz.org/pub/graphviz/stable/windows/graphviz-2.38.msi)安装，或者下载[编写版本](http://www.graphviz.org/pub/graphviz/stable/windows/graphviz-2.38.zip)。
Linux下可以使用下面命令直接安装
```bash
# debian系列
sudo apt install graphivz
# Archlinux系列
sudo pacman -S graphivz
# fedora
sudo dnf install graphivz
```

然后使用`dot`命令将前面生成的`call_graph.dot`文件转换为图像文件。
**生成png文件**
```bash
$ dot -Tpng call_graph.dot -o call_graph.png
```
**生成PostScript文件**
```bash
$ dot -Tps call_graph.dot -o call_graph.ps
```
如果要生成其他格式的图像，可以使用`-T`选项指定，支持eps/gif/jpeg/ps/svg/png/ps2/svgz等格式/
如果需要对布局引擎进行选择，可以使用`-K`进行指定，支持circo/dot/fdp/neato/nop/nop1/nop2/osage/patchwork/sfdp/twopi等。
例如，生成`png`图像，使用`fdp`布局引擎的命令如下:

```bash
$ dot -Tsvg -Kfdp call_graph.dot -o call_graph.svg
```
注：布局引擎可以根据自己的喜好选择，个人倾向使用fdp。默认是上下方向布局的，可以在`call_graph.dot`中添加一行`rankdir=LR`改为左向右方向布局。

## 参考
[Linux性能評測工具之一：gprof篇](http://www.xuebuyuan.com/zh-tw/1849475.html)
[Linux下性能分析工具和内存泄露检测工具的简介(Valgrind和gprof)](http://blog.csdn.net/u014717036/article/details/50762252)
[gprof_call-graph_visualization](https://redmine.epfl.ch/projects/python_cookbook/wiki/gprof_call-graph_visualization)