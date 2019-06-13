---
layout:  post
title:  "Linux下RocksDB、LevelDB、ForestDB性能测试对比"
date:  2017-01-20
categories:  linux
tags:  linux
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2017/01/20/6323400.html](http://www.cnblogs.com/oloroso/archive/2017/01/20/6323400.html)
## 简要说明

本次环境与[http://www.cnblogs.com/oloroso/p/6306352.html](http://www.cnblogs.com/oloroso/p/6306352.html)中的一致。
依然是增删查改各测试10000次，每个测试重复5次取平均值。

## 1、不使用jemalloc和tbb测试
三个数据库除了`rocksdb`之外，默认都不使用`jemalloc`和`tbb`。
设置`rocksdb`的编译参数为`make static_lib -e DISABLE_JEMALLOC=1 -j8`，以便不启用`jemalloc`。

三个测试代码的编译命令如下:
```shell
g++ rocksdb_test.cpp  -o rocksdb_test  -I./include  -L. -lrocksdb -lpthread -lrt -lsnappy -lgflags -lz -lbz2 -llz4 -O2
g++ leveldb_test.cpp  -o leveldb_test  -I../include -L. -lleveldb -O2 -Wl,-rpath=. 
g++ forestdb_test.cpp -o forestdb_test -I../include -L. -lforestdb -O2 -Wl,-rpath=. 
```
测试结果对比直方图如下
![no-jt](http://images2015.cnblogs.com/blog/693958/201701/693958-20170120173835468-1593238098.png)

## 2、使用jemalloc和tbb测试
因为`leveldb`内没有设置使用`jemalloc`的代码，所以只在链接的时候添加。
`forestdb`使用`cmake`生成`Makefile`的时候设置变量`COUCHBASE_SERVER_BUILD`和`_JEMALLOC`的值为`1`来使用`jemalloc`。

三个测试代码的编译命令如下:
```shell
g++ rocksdb_test.cpp  -o rocksdb_test  -I./include  -L. -lrocksdb -lpthread -lrt -lsnappy -lgflags -lz -lbz2 -llz4 -ljemalloc -ltbb -O2
g++ leveldb_test.cpp  -o leveldb_test  -I../include -L. -lleveldb  -ljemalloc -ltbb -O2 -Wl,-rpath=. 
g++ forestdb_test.cpp -o forestdb_test -I../include -L. -lforestdb  -ljemalloc -ltbb -O2 -Wl,-rpath=. 
```
测试结果对比直方图如下
![jt](http://images2015.cnblogs.com/blog/693958/201701/693958-20170120174435203-1415575180.png)

## 测试数据和代码

`leveldb`和`forestdb`的代码见[http://www.cnblogs.com/oloroso/p/6306352.html](http://www.cnblogs.com/oloroso/p/6306352.html)最后部分。
`rocksdb`的测试代码和`leveldb`的测试代码基本一致，只是将其中的`leveldb`全部改为`rocksdb`即可(namespace和头文件路径)。

实际测试的时候发现一个问题，`rocksdb`测试程序在`Open`数据库的时候耗时比较长，貌似花了很多时间在做一些处理，可能是和它的压缩和校验策略有关吧（RocksDB的ReadOptions默认构造时，`verify_checksun`为`true`，而LevelDB默认为`false`）。

![](http://images2015.cnblogs.com/blog/693958/201701/693958-20170120174755437-1458662175.png)

5次测试平均值如下
![](http://images2015.cnblogs.com/blog/693958/201701/693958-20170120174821359-24477442.png)