---
layout:  post
title:  "MongoDB下Map-Reduce使用简单翻译及示例"
date:  2018-12-10
categories:  其它
tags:  其它
comments: 1
---
[TOC]
[博客园原文地址 http://www.cnblogs.com/oloroso/archive/2018/12/10/10097035.html](http://www.cnblogs.com/oloroso/archive/2018/12/10/10097035.html)



> * [原文地址https://docs.mongodb.com/manual/core/map-reduce/](https://docs.mongodb.com/manual/core/map-reduce/)
> * [Map-Reduce 示例](https://docs.mongodb.com/manual/tutorial/map-reduce-examples/)

Map-reduce是一种数据处理范例，用于将大量数据压缩为有用的聚合结果。 对于map-reduce操作，MongoDB提供了[mapReduce](https://docs.mongodb.com/manual/reference/command/mapReduce/#dbcmd.mapReduce)数据库命令。
一个简单的`map-reduce`示例如下：

![](https://img2018.cnblogs.com/blog/693958/201812/693958-20181210155915402-1054962392.png)

在此`map-reduce`操作中，MongoDB将`映射(map)`操作应用于每个输入文档（即集合中与**查询条件匹配**的文档）。map函数提交(emit)一个键值对(key-value)。对于具有多个值的key钥，MongoDB应用`reduce`操作，该操作用于聚合数据。然后MongoDB将结果存储在一个`集合`中。reduce函数的输出还可以选择通过`finalize`函数以进一步压缩或处理聚合的结果。

MongoDB中的所有map-reduce函数都是JavaScript，并在[mongod](https://docs.mongodb.com/manual/reference/program/mongod/#bin.mongod)进程中运行。 Map-reduce操作将[**单个集合**](https://docs.mongodb.com/manual/reference/glossary/#term-collection)的文档作为输入，并可在开始映射阶段之前执行任意排序和限制。 mapReduce可以将map-reduce操作的结果作为文档返回，也可以将结果写入集合。 输入和输出集合可以分片。

对于大多数聚合操作，聚合管道( Aggregation Pipeline)[https://docs.mongodb.com/manual/core/aggregation-pipeline/]提供更好的性能和更一致的接口。 但是，map-reduce操作提供了一些目前在聚合管道中不可用的灵活性。

## Map-Reduce JavaScript 函数
在MongoDB中，map-reduce操作使用自定义JavaScript函数将值(value)映射或关联到键(key)。 如果某个键(key)有对应多个值(value)，则该操作应该将键的值`reduces`为**单个对象**。

使用自定义JavaScript函数可以灵活地进行map-reduce操作。 例如，在处理文档时，map函数可以创建多个键和值映射或不进行映射。 Map-reduce操作还可以使用自定义JavaScript函数对映射的结果进行最终修改，并在映射操作的最后阶段进行reduce操作，执行其他计算。


## Map-Reduce 行为
在MongoDB中，map-reduce操作可以将结果写入集合或返回结果内联。 如果将map-reduce输出写入集合，则可以在合并替换，合并或减少新结果与先前结果的同一输入集合上执行后续map-reduce操作。 有关详细信息和示例，请参阅[mapReduce](https://docs.mongodb.com/manual/reference/command/mapReduce/#dbcmd.mapReduce)和[Perform Incremental(执行增量) Map-Reduce](https://docs.mongodb.com/manual/tutorial/perform-incremental-map-reduce/)。

当返回map-reduce操作的**内联结果**时，结果文档必须在[BSON文档大小限制](https://docs.mongodb.com/manual/reference/limits/#BSON-Document-Size)内，该限制当前为16兆字节。 有关map-reduce操作的限制和限制的其他信息，请参阅[mapReduce参考](https://docs.mongodb.com/manual/reference/command/mapReduce/)页面。

MongoDB支持分片集合上的map-reduce操作。 Map-reduce操作还可以将结果输出到分片集合。 请参见[Map-Reduce and Sharded Collections](https://docs.mongodb.com/manual/core/map-reduce-sharded-collections/)。

[Views](https://docs.mongodb.com/manual/core/views/)(视图)不支持map-reduce操作。

## 一个简单的测试
[MongoDB地理空间数据存储及检索](https://www.cnblogs.com/oloroso/p/9777141.html)
上面链接是之前曾经做过一个全国县级行政边界矢量入库到MongoDB的记录，这里用它来测试一下。

**简单的测试一下全国每个省都有多少个县**
```js
db.getCollection('xzbj').mapReduce(
    function() { emit(this.properties.sheng,1);},
    function(key,values){return Array.sum(values);},
    {
        query:{},
        out:"xian_count"
    }
)
```
这里将结果输出到了`xian_count`这个新的集合中，可以打开这个集合查看结果。
上面的`query`也可以没有，就是默认集合内全部文档。
如果不想把结果输出到一个集合，直接显示结果，则可以使用`out: { inline: 1 }`。

**计算一下湖南省每个地级市有多少个县**
使用下面语句
```js
db.getCollection('xzbj').mapReduce(
    function() { emit(this.properties.di,1);},
    function(key,values){return Array.sum(values);},
    {
        query:{ 'properties.sheng':'湖南'},
        out: { inline: 1 }
    }
)
```
得到输出如下（这里如果是针对全国的数据是有问题的，因为之前没有正确处理港澳台数据）：
```json
{
    "results" : [ 
        {
            "_id" : "娄底市",
            "value" : 5.0
        }, 
        {
            "_id" : "岳阳市",
            "value" : 7.0
        }, 
        {
            "_id" : "常德市",
            "value" : 9.0
        }, 
        {
            "_id" : "张家界市",
            "value" : 3.0
        }, 
        {
            "_id" : "怀化市",
            "value" : 12.0
        }, 
        {
            "_id" : "株洲市",
            "value" : 6.0
        }, 
        {
            "_id" : "永州市",
            "value" : 10.0
        }, 
        {
            "_id" : "湘潭市",
            "value" : 4.0
        }, 
        {
            "_id" : "湘西土家族苗族自治州",
            "value" : 8.0
        }, 
        {
            "_id" : "益阳市",
            "value" : 6.0
        }, 
        {
            "_id" : "衡阳市",
            "value" : 8.0
        }, 
        {
            "_id" : "邵阳市",
            "value" : 11.0
        }, 
        {
            "_id" : "郴州市",
            "value" : 11.0
        }, 
        {
            "_id" : "长沙市",
            "value" : 5.0
        }
    ],
    "timeMillis" : 19.0,
    "counts" : {
        "input" : 105,
        "emit" : 105,
        "reduce" : 14,
        "output" : 14
    },
    "ok" : 1.0,
    "_o" : {
        "results" : [ 
            {
                "_id" : "娄底市",
                "value" : 5.0
            }, 
            {
                "_id" : "岳阳市",
                "value" : 7.0
            }, 
            {
                "_id" : "常德市",
                "value" : 9.0
            }, 
            {
                "_id" : "张家界市",
                "value" : 3.0
            }, 
            {
                "_id" : "怀化市",
                "value" : 12.0
            }, 
            {
                "_id" : "株洲市",
                "value" : 6.0
            }, 
            {
                "_id" : "永州市",
                "value" : 10.0
            }, 
            {
                "_id" : "湘潭市",
                "value" : 4.0
            }, 
            {
                "_id" : "湘西土家族苗族自治州",
                "value" : 8.0
            }, 
            {
                "_id" : "益阳市",
                "value" : 6.0
            }, 
            {
                "_id" : "衡阳市",
                "value" : 8.0
            }, 
            {
                "_id" : "邵阳市",
                "value" : 11.0
            }, 
            {
                "_id" : "郴州市",
                "value" : 11.0
            }, 
            {
                "_id" : "长沙市",
                "value" : 5.0
            }
        ],
        "timeMillis" : 19,
        "counts" : {
            "input" : 105,
            "emit" : 105,
            "reduce" : 14,
            "output" : 14
        },
        "ok" : 1.0
    },
    "_keys" : [ 
        "results", 
        "timeMillis", 
        "counts", 
        "ok"
    ],
    "_db" : {
        "_mongo" : {
            "slaveOk" : true,
            "host" : "127.0.0.1:27017",
            "defaultDB" : "test",
            "_readMode" : "commands"
        },
        "_name" : "us"
    }
}
```