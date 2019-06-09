---
layout: post
title:  "SQLite R*Tree 模块测试"
date:   2019-05-29
categories: 地理信息
tags: SQLite GIS RTree
comments: 1
---

# SQLite R*Tree 模块测试

[TOC]

[博客园文章地址 https://www.cnblogs.com/oloroso/p/10941099.html](https://www.cnblogs.com/oloroso/p/10941099.html)

相关参考：

>   [MySQL空间索引简单使用](https://www.cnblogs.com/oloroso/p/9579720.html)
>
>   [MongoDB地理空间数据存储及检索](https://www.cnblogs.com/oloroso/p/9777141.html)
>
>   [The SQLite R*Tree Module](https://www.sqlite.org/rtree.html)
>
>   [Memory-Mapped I/O](https://www.sqlite.org/mmap.html)
>
>   [In-Memory Databases](https://sqlite.org/inmemorydb.html)
>
>   [libspatialindex](https://libspatialindex.org/)
>
>   [R* tree - Wikipedia](https://en.wikipedia.org/wiki/R*_tree)

我另外做了GEOS STRtree/Quadtree 空间检索的性能，测试代码和数据可见[Spatial_Index_Test](https://github.com/sotex/Spatial_Index_Test)


## 1、SQLite R*Tree 模块特性简介

关于SQLite的空间索引相关介绍可以查看官方文档 [The SQLite R*Tree Module](https://www.sqlite.org/rtree.html) ，这里只做简单的介绍。

1、**SQLite R \*Tree**模块实现部分在其源代码内（[源码下载页面](https://www.sqlite.org/download.html)），无需另外合并。但是默认是没有启用的，启用需要定义`SQLITE_ENABLE_RTREE=1`宏再编译。

2、**SQLite R \*Tree**模块采用虚拟表实现，每个R \*Tree索引都是一个虚拟表。对于这个表，其第一列必须是64位有符号整数类型，作为主键。其它的列（2-12列）根据空间维度确定，每个维度包含一对（两列），分别是该维度的最小和最大值。例如：一维R \*Tree索引虚拟表包含3列，分别是**Int64主键| 最小值| 最大值**；二维R\*Tree索引虚拟表包含5列，分别是**Int64主键| 第一维最小值| 第一维最大值| 第二维最小值| 第二维最大值**；3、4、5维R\*Tree索引虚拟表列数情况的以此论推，**SQLite R \*Tree**实现不支持宽度超过5维的R *树。

3、对于各个维度的最大最小值列，SQLite中可以使用`int32`或者`float32`类型进行数据存储。与其它常规表中的列不同，这里存储就是二进制类型的值，而不是转换为字符串。如果在插入数据的时候，使用了这两者之外的类型，则会进行隐式转换。
```sqlite
    -- 创建整型坐标rtree索引虚拟表
    CREATE VIRTUAL TABLE intrtree USING rtree_i32(id,x0,x1,y0,y1,z0,z1);
    -- 创建浮点型坐标rtree索引虚拟表
    CREATE VIRTUAL TABLE floatrtree USING rtree(id,x0,x1,y0,y1,z0,z1);
```

4、**SQLite R \*Tree**中查询并不限制查询的维度一定要与所查询的表中的维度一致，可以仅查询其中的某几个维度（如3维空间仅查询2个维度）。一般来说，约束（维度）越多，查询的范围框越小，速度越快。

5、默认情况下使用`float32`存储坐标值，当无法精确表示传入值时，下限坐标向下舍入，上限坐标向上舍入，因此边界框可能略大于指定，但永远不会变小。这在查询某个**范围之外**的数据时，可能会有极小的误差。

6、对于**3.24.0**之前的版本，**SQLite R \*Tree**索引虚拟表仅能存储整数主键和坐标值列，其它的信息需要另存于其它表中（通过主键进行关联）。从3.24.0版本开始，**SQLite R \*Tree**索引虚拟表可以存储任意类型数据的辅助列，辅助列必须以`+`开头，最多可以存储100个辅助列。
```sqlite
    CREATE VIRTUAL TABLE demo_index2 USING rtree(
       id,              -- 64位整型主键
       minX, maxX,      -- X方向最小最大值
       minY, maxY,      -- Y方向最小最大值
       +objname TEXT,   -- 辅助列 文本类型
       +objtype TEXT,   -- 辅助列 文本类型
       +boundary BLOB   -- 辅助列 二进制数据
    );
```

7、可以自定义R-Tree查询，以便实现非矩形框碰撞。这需要通过`sqlite3_rtree_query_callback`（新，3.8.5开始提供）或`sqlite3_rtree_geometry_callback`（旧）注册查询SQL语句和匹配检测回调。相关信息在SQLite网站上有详细介绍。

8、一个**SQLite R \*Tree**会附带三个影子表，用于存储数据，分别是`虚拟表名_node`（存储节点） `虚拟表名_parent`（存储父节点） `虚拟表名_rowid`（存储节点的rowid）。

9、可以使用`SELECT rtreecheck('虚拟表名')`来对R-Tree索引进行完整性和正确性检查。

## 2、SQLite R*Tree 模块简单测试代码

写了一个简单的测试程序来测试一下**R *Tree**树的速度，结果还是可以的。（可用，并不是最佳）

我的机器环境是：

>   Windows 10 1903 x64专业版
>
>   AMD 锐龙 2600X
>
>   DDR4 2400 8G
>
>   编译器：VS2017 Native x64

使用本地文件的时候，十万条数据插入时间大概在2秒以内，查询一个5x5度大小的范围，时间基本在0.07秒以内；使用内存模式时，插入时间大概在1.8秒以内，查询一个5x5度大小的范围，时间基本在0.04秒以内。

**注意：编译SQLite的时候要定义`SQLITE_ENABLE_RTREE`宏，开启RTree索引支持。**

```c
#include "sqlite/sqlite3.h"
#include<time.h>
#include <stdlib.h>
#include <stdio.h>

// 因为仅仅是进行一下试用测试，所以有些地方就没有处理，包括close

int main() 
{
    sqlite3* db = NULL;
    int rc = sqlite3_open(":memory:", &db);
    // int rc = sqlite3_open("D:/sqlite_rtree/test.db", &db);
    if (rc != SQLITE_OK) 
    {
        return -1;
    }
    
    char* errmsg;
    // 创建RTree索引虚拟表
    rc = sqlite3_exec(db,
                      "CREATE VIRTUAL TABLE demo_index USING rtree(id,minX, maxX,minY, maxY,+axucol INTEGER NOT NULL)",
                      NULL, NULL, &errmsg);
    if (rc != SQLITE_OK) 
    {
        printf("%4d Error:%sn", __LINE__, errmsg);
        return -2;
    }
    
    // 开始计时
    clock_t start = clock();
    
    // 开启事物
    if (sqlite3_exec(db, "begin", NULL, NULL, &errmsg) != SQLITE_OK) {
        printf("%4d Error:%sn", __LINE__, errmsg);
        return -2;
    }
    
    // 生成十万个大小在 边长在[0.002,0.202]度大小以内的数据(0.2~22.5公里左右)
    srand(time(NULL));  // 初始化随机数种子
    sqlite3_stmt *pStmt = NULL;
    
    // 预处理SQL语句
    if(sqlite3_prepare_v2(db,
                          "INSERT INTO demo_index VALUES(?,?,?,?,?,?)",
                          -1, &pStmt, NULL) != SQLITE_OK) {
        printf("%4d Error:%sn", __LINE__, errmsg);
        return -3;
    }
    // 逐个插入
    for (int i = 0; i < 100000; ++i) {
        // 生成在经纬度范围内的x,y
        double x0 = ((double)rand() / (double)RAND_MAX) * 360 - 180;
        double y0 = ((double)rand() / (double)RAND_MAX) * 180 - 90;
        double x1 = x0 + 0.002 + ((double)rand() / (double)RAND_MAX)*0.2;
        double y1 = y0 + 0.002 + ((double)rand() / (double)RAND_MAX)*0.2;
        // 绑定数据
        sqlite3_bind_int64(pStmt, 1, i);
        sqlite3_bind_double(pStmt, 2, x0);
        sqlite3_bind_double(pStmt, 3, x1);
        sqlite3_bind_double(pStmt, 4, y0);
        sqlite3_bind_double(pStmt, 5, y1);
        sqlite3_bind_int(pStmt, 6, rand()%3);
        // 执行
        sqlite3_step(pStmt);
        // 重置
        sqlite3_reset(pStmt);
    }
    sqlite3_finalize(pStmt); //结束语句，释放语句句柄
    
    // 结束事物
    if (sqlite3_exec(db, "commit", NULL, NULL, &errmsg) != SQLITE_OK){
        printf("%4d Error:%sn", __LINE__, errmsg);
        return -2;
    }
    
    
    // 结束计时
    clock_t end = clock();
    double hs = (double)(end - start) * 1000 / CLOCKS_PER_SEC;
    printf("插入总耗时: %lf msn", hs);
    // 查询
    // select * from test where NOT(maxX<74.254915 OR minX>79.765758 OR maxY< 24.214285 OR minY>29.725129) AND auxcol==2 ORDER BY id;
    // 预处理SQL语句
    pStmt = NULL;
    if (sqlite3_prepare_v2(db,
            "SELECT id,minX,minY,auxcol FROM demo_index WHERE NOT(maxX<? OR minX>? OR maxY<?  OR minY>?) AND auxcol==1;",
            -1, &pStmt, NULL) != SQLITE_OK) {
        printf("%4d Error:%sn", __LINE__, errmsg);
        return -4;
    }
    
    //-------------------------------------------------------------------------
    
    // 输入查询的范围框数据
    puts("Input x0,x1,y0,y1:");
    double x0, x1, y0, y1;
    scanf("%lf,%lf,%lf,%lf", &x0, &x1, &y0, &y1);
    printf("-----------[%lf,%lf,%lf,%lf]-------------n", x0, x1, y0, y1);
    
    // 开始计时
    start = clock();
    
    // 绑定查询范围数据
    sqlite3_bind_double(pStmt, 1, x0);
    sqlite3_bind_double(pStmt, 2, x1);
    sqlite3_bind_double(pStmt, 3, y0);
    sqlite3_bind_double(pStmt, 4, y1);
    while (sqlite3_step(pStmt) == SQLITE_ROW) {
        int id = sqlite3_column_int(pStmt, 0);
        double x = sqlite3_column_double(pStmt, 1);
        double y = sqlite3_column_double(pStmt, 2);
        int auxcol = sqlite3_column_int(pStmt, 3);
        // 可以把输出去掉，减少对时间统计的影响
        printf("%dt %lf,%lfn", id, x, y);
    }
    sqlite3_reset(pStmt); // 这里只查询一次可以没有，如果需要多次使用这个查询语句，则必须有，不然查出数据不对
    sqlite3_finalize(pStmt); //结束语句，释放语句句柄
    
    // 结束计时
    end = clock();
    hs = (double)(end - start) * 1000 / CLOCKS_PER_SEC;
    printf("本次查询总耗时: %lf msn", hs);
    
    
    sqlite3_close(db);
    system("pause");
    return 0;
}
```