---
layout:  post
title:  "libmongoc关于\$pullAll和\$addToSet的一个使用问题记录"
date:  2018-09-03
categories:  其它
tags:  其它
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2018/09/03/9578888.html](http://www.cnblogs.com/oloroso/archive/2018/09/03/9578888.html)
## 问题描述及测试结果
在使用mongodb时，对一个document中的数组成员进行更新的时候，可以使用`$pull` `$push` `$pop` `$addToSet` `$pullAll`和`$each` `$position` `$slice` `$sort`等操作符。

**以下问题出现在`$addToSet`和`pullAll`操作中，`$set`操作没有这个问题，其他的操作符没有测试，不知道有没有问题。**

之前在libmongoc中更新一个对象，用到了这些操作的时候，对于添加进`update`这个bson对象中的数组成员，其`key`是没有要求的，大概如下：
```cpp
// 更新对象的选择
bson_t* selector = BCON_NEW("_id",BCON_OID(_id));
// 更新的内容
bson_t* update = bson_new();
bson_t* each = bson_new();
bson_t  array;
bson_append_array_begin(each, "$each", 5, &array);
// 向数组中逐个添加元素,itemArray是要添加的数据保存的数组
for(auto& item:itemArray){
    // 以前这里添加的时候，key都使用'0'是没有问题的
    bson_append_utf8(&array,"0",1,item.data(),item.size());
}
bson_append_array_end(each, &array);
bson_append_document(update,"$addToSet",9,each);
// 执行更新操作
 mongoc_collection_update(
		coll, MONGOC_UPDATE_NONE, selector, update, NULL, &error);
```
这是去年我写代码的时候的做法，这样的操作是一点问题都没有的。当时使用的`MongoDB`是`3.4.0`版本，使用的`libmongoc`是`1.3.1`版本。最近一个新项目中再次使用到了`MongoDB`，这次使用的是`4.0.2`版本，`libmongoc`使用的是`1.9.3`版本。
**经过测试(lobmongoc1.9.3)，这样的代码在`MongoDB 4.0.2`版本中，`$addToSet`和`pullAll`两个有问题，无法实现多个成员的操作。在`MongoDB3.4.0`中，`$addToSet`是没有问题的，但是`$pullAll`也是只能移除一个，不能移除多个。我这里没有测试更多操作，因为只用到了这两个，也没有测1.3.1版本的lobmongoc，因为不能回退到这个版本了。**

做如下修改可以完成正常想要的操作
```cpp
// 更新对象的选择
bson_t* selector = BCON_NEW("_id",BCON_OID(_id));
// 更新的内容
bson_t* update = bson_new();
bson_t* each = bson_new();
bson_t  array;
bson_append_array_begin(each, "$each", 5, &array);
// 向数组中逐个添加元素,itemArray是要添加的数据保存的数组
uint32_t i = 0;
for(auto& item:itemArray){
    // 以前这里添加的时候，key都使用'0'是没有问题的
    // bson_append_utf8(&array,"0",1,item.data(),item.size());
   // 不能再使用像上面一样的同一个key，使用下面的形式
    char keybuf[16];
    const char *key = keybuf;
    int keylen = bson_uint32_to_string(i++, &key, keybuf, sizeof(keybuf));
    bson_append_utf8(&array,key,keylen,item.data(),item.size());
}
bson_append_array_end(each, &array);
bson_append_document(update,"$addToSet",9,each);
// 执行更新操作
 mongoc_collection_update(
		coll, MONGOC_UPDATE_NONE, selector, update, NULL, &error);
```