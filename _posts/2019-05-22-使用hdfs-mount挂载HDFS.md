---
layout: post
title:  "使用hdfs-mount挂载HDFS"
date:   2019-05-22
categories: 分布式
tags: HDFS FUSE hdfs-mount
comments: 1
---
# 使用hdfs-mount挂载HDFS

[TOC]

[博客园文章地址 https://www.cnblogs.com/oloroso/p/10906275.html](https://www.cnblogs.com/oloroso/p/10906275.html)

`hdfs-mount`是一个将HDFS挂载为**本地`Linux`文件系统**的工具，使用go语言开发，不依赖libdfs和java虚拟机。它允许将远程HDFS作为本地Linux文件系统挂载，并允许任意应用程序或shell脚本以高效和安全的方式访问HDFS作为普通文件和目录。

## 1、特性(计划)简介

以下翻译自 [hdfs-mount/README.md](https://github.com/microsoft/hdfs-mount/blob/master/README.md)

-   高性能
    -   使用protocol buffers协议直接连接HDFS和Linux内核FUSE接口（无需Java虚拟机）
    -   针对吞吐量密集型工作负载进行设计和优化（尽可能以延迟交换吞吐量）
    -   完全流式传输和自动预读支持
    -   并发操作
    -   在内存中缓存元数据 (速度非常快 l!)
-   高稳定性和强大的故障处理行为
    -   自动重试和故障转移，全部可配置
    -   可选的延迟挂载, 在 HDFS 可用之前
-   读写操作都支持
    -   支持随机写入[慢，但功能正确]
    -   支持文件截断
-   （可选）扩展ZIP存档，并根据需要提取内容
    -   这为”数百万个小文件在HDFS上“(millions of small files on HDFS)问题提供了有效的解决方案
-   对CoreOS和Docker友好
    -   可选择打包为静态链接的独立可执行文件



## 2、构建程序

我的系统环境是`CentOS 7.0 x86_64`，以下所有操作都是基于此。

先安装编译所需的必要工具软件：

```bash
yum install make golang
```

然后下载`hdfs-mount`的源码回来，源码仓库地址:[https://github.com/microsoft/hdfs-mount.git](https://github.com/microsoft/hdfs-mount.git)

```bash
git clone https://github.com/microsoft/hdfs-mount.git
```

然后就可以编译了，先进入源码目录

```bash
# 进入源码根目录
cd hdfs-mount
# 执行构建
make
```

**编译的过程中因为要下载依赖（[bazil/fuse](github.com/bazil/fuse)、[x/net/context](golang.org/x/net/context)、[protobuf/proto](github.com/golang/protobuf/proto)），所以需要保持网络通畅。**



## 3、使用hdfs-mount挂载HDFS

因为`hdfs-mount`基于FUSE(用户空间文件系统)框架设计，所以需要先安装`fuse`内核模块，并加载（如果没有加载`fuse`内核模块，则无法hdfs-mount操作时候将失败，并提示加载）。

```bash
# 安装fuse模块
sudo yum install fuse
# 加载fuse模块
sudo modprobe fuse
```

然后就可以使用`hdfs-mount`进行HDFS的挂载了，挂载后可以使用`dd`命令进行一下读写速度测试。

```bash
# 挂载，因为hdfs-mount并不会后台运行，所以使用&放入后台
sudo ./hdfs-mount 192.168.0.32:9000 /mnt/hdfs &
# 测试一下读取速度
dd bs=64k if=/mnt/hdfs/test.img of=/dev/null
记录了24421+1 的读入
记录了24421+1 的写出
1600485903字节(1.6 GB)已复制，9.56565 秒，167 MB/秒
```

`hdfs-mount`的一些命令行参数如下，更多细节可执行`hdfs-mount --help`查看。

```bash
 # 查看使用帮助信息
./hdfs-mount --help
# 挂载必要选项:  HDFS名字节点 端口  挂载位置
./hdfs-mount      NameNode:Port MountPoint 

# 下面是一些可选参数
-allowedPrefixes string
   # 允许的前缀字符串，以逗号分隔的远程HDFS上允许访问的路径前缀列表，如果指定了，挂载点将仅公开对这些前缀路径的访问(默认*)
-expandZips
   # 实现zip文件的自动展开（访问zip包内部文件）
-fuse.debug
   # 日志输出FUSE处理的线性信息
-lazy
   # 延迟加载，允许在HDFS就绪前挂载HDFS文件系统
-logLevel int
   # 日志输出级别控制。0：仅fatal/err日志；1：+warning日志；2：+info日志
-readOnly
   # 启用只读挂载
-retryMaxAttempts int
   # 失败操作的最大重试次数（默认99999999）
-retryMaxDelay    duration
   # 两次重试之间的最大延时（默认1分0秒）
-retryMinDelay    duration
   # 两次重试之间的最小延时（注意，第一次重试总是立即进行）（默认为1秒）
-retryTimeLimit   duration
   # 失败操作的所有重试尝试的时间限制 (默认为 5分0秒)
```