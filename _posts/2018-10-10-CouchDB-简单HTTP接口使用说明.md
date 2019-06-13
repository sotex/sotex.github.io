---
layout:  post
title:  "CouchDB 简单HTTP接口使用说明"
date:  2018-10-10
categories:  其它
tags:  其它
comments: 1
---
[TOC]
[博客园原文地址 http://www.cnblogs.com/oloroso/archive/2018/10/10/9767546.html](http://www.cnblogs.com/oloroso/archive/2018/10/10/9767546.html)


# 1、简介
Apache CouchDB 是一个面向文档的数据库管理系统。它提供以 JSON 作为数据格式的 REST 接口来对其进行操作，并可以通过视图来操纵文档的组织和呈现。 CouchDB 是 Apache 基金会的顶级开源项目。
CouchDB里面有一个小问题，就是它的一些预定义的`key`有时候有下划线，有时候又不能有下划线开头，例如`id``rev`这样的。没有固定下来，不知道是为何。

参考资料：
- [CouchDB 视图简介及增量更新视图的方法](https://www.ibm.com/developerworks/cn/opensource/os-cn-couchdb-view-change/)
- [探索 CouchDB](https://www.ibm.com/developerworks/cn/opensource/os-couchdb/)
- [优化CouchDB的读性能](https://www.xuebuyuan.com/2222710.html)


# 2、安装
直接去[官网](http://couchdb.apache.org/)下载对应的安装包安装即可。
安装之后服务会自动启动，监听`5984`(HTTP)和`6984`(HTTPS)端口，可以直接在浏览器访问[http://127.0.0.1:5984/_utils](http://127.0.0.1:5984/_utils)地址，打开Web交互操作页面。
在`Your Account`页面创建服务管理员用户或修改用户密码。
![](https://img2018.cnblogs.com/blog/693958/201810/693958-20181010165329551-787171498.png)

# 2、HTTP接口简单使用
CounchDB相关文档可以查阅[http://docs.couchdb.org](http://docs.couchdb.org)
这里只简单记录一下我用到的几个`HTTP`接口。

**CouchDB HTTP接口返回状态码简述**
这里只列出常用的状态码及其说明，特定的请求类型不同的状态码信息在相应的API参考文档中有详细说明。

|状态码|状态|说明|
|:---|:---|:---|
|200|OK|请求已经成功完成|
|201|Created|文档创建成功|
|202|Accepted|请求已被接受，但相应的操作可能尚未完成。<br>这用于后台操作，例如数据库压缩|
|304|Not Modified|请求的其他内容尚未修改。这与ETag系统一起使用以识别返回的信息的版本|
|400|Bad Request|不良请求。该错误可能由于请求URL、Path或Header出错。<br>提供的MD5值和content不匹配也会触发此错误，因为这可能是消息内容被破坏|
|401|Unauthorized|使用提供的授权无法获得所请求的项目，或未提供授权|
|403|Forbidden|禁止请求的项目或操作|
|404|Not Found|找不到请求的内容。<br>返回的content将相关信息,返回的JSON对象包含键`error`和`reason`(原因)。例如：<br>`{"error":"not_found","reason":"no_db_file"}`|
|405|Method Not Allowed|对请求的URL使用无效的HTTP请求类型发出了请求。<br>例如，您需要POST时请求PUT。此类型的错误也可能由无效的URL字符串触发。|
|406|Not Acceptable|服务器不支持请求的内容类型|
|409|Conflict|请求导致更新冲突|
|412|Precondition Failed|来自客户端的请求标头和服务器的功能不匹配|
|413|Request Entity Too Large|请求实体太大|
|415|Unsupported Media Type|不支持的媒体类型|
|416|Requested Range Not Satisfiable|服务器无法满足请求头中指定的范围|
|417|Expectation Failed|批量发送文档时，批量加载操作失败|
|201|Internal Server Error|请求无效，原因是提供的JSON无效，或者作为请求的一部分提供了无效信息|

## 2.1、认证接口
认证的方式官网文档中介绍的有三种，我只对前面两种进行了操作。具体的可见[http://docs.couchdb.org/en/latest/api/server/authn.html](http://docs.couchdb.org/en/latest/api/server/authn.html)

要学习HTTP协议的认证相关内容，可以看Mozilla的文档，全中文的。[https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Authentication](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Authentication)

### 2.1.1 Basic Authentication
基本身份验证[Basic authentication (RFC 2617)](https://tools.ietf.org/html/rfc2617.html)是一个比较快速方便的认证方式，对于需要认证的接口，每次都要发送用户凭据。

这个认证方式具体的操作，就是在发送请求的时候，在`HTTP头`中添加`Authorization`消息头，其内容为`Bascic 使用Basic64编码的账号:密码`。
例如，账号为`user1`，密码为`abcd1234`的用户，那么他发送的一个请求的示例如下：
**curl命令请求**
```bash
curl --request GET \
  --url http://localhost:5984/db1/doc1 \
  --header 'authorization: Basic bzphc2QxMjM=' 
}'
```
请求报文如下：
```
GET / HTTP/1.1
Accept: application/json
Authorization: Basic dXNlcjE6YWJjZDEyMzQ=
Host: localhost:5984
```
其中`dXNlcjE6YWJjZDEyMzQ=`即为`user1:abcd1234`的Base64编码值。

### 2.1.2 Cookie Authentication
cookie身份验证[（RFC 2109）](https://tools.ietf.org/html/rfc2617.html)，CouchDB会生成一个`token`(令牌)，客户端可以使用该`token`(令牌)来处理CouchDB的下一个请求。 `token`(令牌)在超时前有效。 当CouchDB在后续请求中看到有效`token`(令牌)时，它将通过此`token`(令牌)对用户进行身份验证，而无需再次请求密码。 默认情况下，cookie有效期为10分钟，可以在配置页面进行修改，也可以使cookie持久化。

**具体操作过程如下：**
**1、获取token**
要获取一个token(实际就是获取响应消息头中的Set-Cookie值)，必须将用户名和密码发送到CouchDB服务器，这个通过`_session`接口来实现。
**_session接口说明**
`_session`是一个`POST`请求接口，只需要设置请求消息头`Content-Type`参数，其值可以是`application/x-www-form-urlencoded`或者`application/json`。请求的content实体内容格式根据`Content-Type`的值确定。
更多具体的参数请参考官方文档。
**curl命令如下(发送的是json格式的)：**
```bash
curl --request POST \
  --url http://localhost:5984/_session \
  --header 'content-type: application/json' \
  --data '{
	"name":"user1",
	"password":"abcd1234"
}'
```
报文示例如下：
```
POST /_session HTTP/1.1
Accept: application/json
Content-Length: 24
Content-Type: application/x-www-form-urlencoded
Host: localhost:5984

name=root&password=relax
```
使用JSON格式的请求报文示例：
```
POST /_session HTTP/1.1
Accept: application/json
Content-Length: 37
Content-Type: application/json
Host: localhost:5984

{
    "name": "root",
    "password": "relax"
}
```
成功返回的响应内容示例如下：
```
HTTP/1.1 200 OK
Cache-Control: must-revalidate
Content-Length: 43
Content-Type: application/json
Date: Mon, 03 Dec 2012 01:23:14 GMT
Server: CouchDB (Erlang/OTP)
Set-Cookie: AuthSession=cm9vdDo1MEJCRkYwMjq0LO0ylOIwShrgt8y-UkhI-c6BGw; Version=1; Path=/; HttpOnly

{"ok":true,"name":"root","roles":["_admin"]}
```
返回的json对象中`ok`的值表示操作状态，`name`的值是用户名，`relos`数组中是用户的角色（一个用户可以有多个角色）。
**响应报文中的Set-Cookie消息头的值，就是后面所有请求需要使用到的Cookie内容。**

返回的状态码可能是以下三个

|状态码|状态|说明|
|:---|:---|:---|
|200|OK|成功通过身份验证|
|302|Found|成功验证后重定向|
|401|Unathorized|无法识别用户名或密码|

**2、使用token**
通过上一步获取到`Cookie`值之后，就可以对一些需要认证的接口进行操作。简单来说，就是在发送请求的时候添加`Cookie`消息头。
**curl命令示例**
```bash
curl --request GET \
  --url http://localhost:5984/db1 \
  --header 'cookie: AuthSession=cm9vdDo1MEJCRkYwMjq0LO0ylOIwShrgt8y-UkhI-c6BGw
# 或将上面的--header不要，改为使用下面的参数
#  --cookie 'AuthSession=cm9vdDo1MEJCRkYwMjq0LO0ylOIwShrgt8y-UkhI-c6BGw'
```

**3、删除token**
删除一个`token`即表示这个`Cookie`不再使用了，也就是关闭这个会话。但因为CouchDB Cookie是无状态的，所以此操作实际并不起什么作用。
删除使用一个`DELETE`请求，curl命令示例如下:
```bash
curl --request DELETE \
  --url http://localhost:5984/_session \
  --header 'cookie: AuthSession=cm9vdDo1MEJCRkYwMjq0LO0ylOIwShrgt8y-UkhI-c6BGw;
```
请求报文示例如下：
```
DELETE /_session HTTP/1.1
Accept: application/json
Cookie: AuthSession=cm9vdDo1MjA1NEVGMDo1QXNQkqC_0Qmgrk8Fw61_AzDeXw
Host: localhost:5984
```
响应报文示例如下:
```
HTTP/1.1 200 OK
Cache-Control: must-revalidate
Content-Length: 12
Content-Type: application/json
Date: Fri, 09 Aug 2013 20:30:12 GMT
Server: CouchDB (Erlang/OTP)
Set-Cookie: AuthSession=; Version=1; Path=/; HttpOnly

{
    "ok": true
}
```

## 2.2 创建与删除数据库
创建数据库就是向服务器发送一个特定的`PUT`请求即可。
**curl命令示例如下:**
```bash
curl --request PUT \
  --url http://127.0.0.1:5984/db1 \
  --cookie 'AuthSession=cm9vdDo1MEJCRkYwMjq0LO0ylOIwShrgt8y-UkhI-c6BGw; Version=1; Path=/; HttpOnly'
```
这里的`URL`中的路径中的`db1`就是数据库名称，这个名称只能是小写字母和数组，不能是别的。
如果已经存在或别的错误，返回的JSON中会有相关信息
```
{
	"error": "file_exists",
	"reason": "The database could not be created, the file already exists."
}
或数据库名称不合规则时候(__user1不符合规则)
{
	"error": "illegal_database_name",
	"reason": "Name: '__user1'. Only lowercase characters (a-z), digits (0-9), and any of the characters _, $, (, ), +, -, and / are allowed. Must begin with a letter."
}
```
删除数据库则向服务器发送一个`DELETE`请求，参数与创建的一致。
**curl命令示例如下:**
```bash
curl --request DELETE \
  --url http://127.0.0.1:5984/db1 \
  --cookie 'AuthSession=cm9vdDo1MEJCRkYwMjq0LO0ylOIwShrgt8y-UkhI-c6BGw; Version=1; Path=/; HttpOnly'
```
不存在的时候也会返回相关的信息
```
{
	"error": "not_found",
	"reason": "Database does not exist."
}
```

## 2.3 插入、删除、更新文档(记录)

### 2.3.1 插入文档
插入文档也使用`PUT`请求，文档必须插入到某个数据库内。每个文档都有一个`_id`的主键，这个键的值**不能以下划线开头**。如果没有给定`_id`则会自动生成一个，类似于`f9ba72763355579163d36f31de002246`这样的。
文档的内容必须是一个合法的json格式。CouchDB中所有预定义的键值都是以下划线开头的，所以**所有的键都不能以下划线开头**。每个文档都有一个版本号，创建的时候会自动添加`_rev`字段表示。

**curl插入单个文档示例:**
```bash
curl --request PUT \
  --url http://localhost:5984/db1/%E4%B8%AD%E6%96%87 \
  --header 'content-type: application/json'
  --cookie 'AuthSession=bzo1QkJENkRCRToXLIOUaf3twhYIrs-eTTyTwJFB5w; Version=1; Path=/; HttpOnly' \
  --data '
{
    "字符串": "字符串值",
    "数组": [
        "值2",
        2,
        1.234
    ],
    "对象":{
			"key":"值"
		}
}'
```
上面的路径参数中的`db1`就是先前创建的数据库名，后面的`%E4%B8%AD%E6%96%87`是`键值`两个字的URL编码格式，也就是文档的`_id`值。
返回结果如下：
```json
{
	"ok": true,
	"id": "键值",
	"rev": "1-4fd4c8f7fe3d72bc579f6cfc45f6b0d4"
}
```

对于批量插入文档，则需要使用`_bulk_docs`命令来操作，这个命令需要使用`POST`方法来请求。
批量插入的时候，将需要插入的多个文档的内容，放入一个json数组中，每个文档的`_id`参数也必须明确指定。
示例如下：
**curl插入多个文档示例：**
```bash
curl --request POST \
  --url http://localhost:5984/db1/_bulk_docs \
  --header 'content-type: application/json' \
  --cookie AuthSession=bzo1QkJEOUM4QTr6P3NYSJprjQlgFYgeba6cwwBqJw \
  --data '
{
	"docs": [
		{"_id":"s1","desc":"bobo","age":15},
		{"_id":"s2","desc":"小红","age":19},
		{"_id":"s3","desc":"小明","age":22},
		{"_id":"s4","desc":"ab","age":14}
	]
}'
```
报文如下：
```
POST /db1/_bulk_docs HTTP/1.1
Host: localhost:5984
User-Agent: insomnia/6.0.2
Cookie: AuthSession=bzo1QkJENkRCRToXLIOUaf3twhYIrs-eTTyTwJFB5w
Content-Type: application/json
Accept: */*
Content-Length: 175
{
	"docs": [
		{"_id":"s1","desc":"bobo","age":15},
		{"_id":"s2","desc":"小红","age":19},
		{"_id":"s3","desc":"小明","age":22},
		{"_id":"s4","desc":"ab","age":14}
	]
}
```
返回结果如下(数组中的每个对象，都和单个插入是一样的)：
```
[
	{
		"ok": true,
		"id": "s1",
		"rev": "1-5efc8527e2a56dc31e71abe36d365e38"
	},
	{
		"ok": true,
		"id": "s2",
		"rev": "1-959ab718375f00a5b678ece00f68a1f7"
	},
	{
		"ok": true,
		"id": "s3",
		"rev": "1-08ded6f70086c004a2b8ef6bb2439b88"
	},
	{
		"ok": true,
		"id": "s4",
		"rev": "1-ccd6248a85ad565b2b12a3127c1b82d7"
	}
]
```

### 2.3.2 删除文档
删除文档和删除数据库类似，也是使用`DELETE`请求，使用`URL`中的`Path`部分指定数据库和文档`_id`。
删除的时候还必须指定文档的版本，如果没有指定版本，则会提示更新冲突。
**curl删除单个文档示例：**
```bash
curl --request DELETE \
  --url 'http://localhost:5984/dbtest/s3?rev=1-08ded6f70086c004a2b8ef6bb2439b88' \
  --cookie AuthSession=bzo1QkJEOUYxMjr2HOFipeXf2K3e3Qy8QmeKuHgTaw
```
返回结果示例:
```
{
	"ok": true,
	"id": "s3",
	"rev": "2-eba8354ebc6964892c780f2d1435dd7f"
}
```
可以看到，删除的结果中仅`rev`值变化了。这时候如果再向数据库中插入`_id`为`s3`的文档，则其`_rev`值将会变为`3-xxxxx...xxx`，也就是版本号更新了。

对于删除多个文档，也是使用`_bulk_docs`命令来操作。在POST提交的数据中，JSON对象中`docs`数组下的每个对象都表示一个要删除的文档，必须含有文档的`_id`和`_rev`参数，`_deleted`参数用来指示操作为删除操作。
**curl删除多个文档示例：**
```bash
curl --request POST \
  --url http://127.0.0.1:5984/dbtest/_bulk_docs \
  --header 'content-type: application/json' \
  --cookie AuthSession=bzo1QkJEQTEzOTpmznvulsDVLy38QY1AemMA5LZXNQ \
  --data '{
	"docs": [
		{
			"_id": "s1",
			"_rev": "5-64964a8e3910a7f17b8bd1b2942811bf",
			"_deleted": true
		},
		{
			"_id": "s2",
			"_rev": "5-64964a8e3910a7f17b8bd1b2942811bf",
			"_deleted": true
		},
		{
			"_id": "s4",
			"_rev": "3-e319f63912531859882a9495568fe7f4",
			"_deleted": true
		}
	]
}'
```
返回结果如下：
```
[
	{
		"ok": true,
		"id": "s1",
		"rev": "6-266b9d0fda1846ccf8c37facbb2f03ba"
	},
	{
		"id": "s2",
		"error": "conflict",
		"reason": "Document update conflict."
	},
	{
		"ok": true,
		"id": "s4",
		"rev": "4-836b5d321c9eed62ffdbe124f21df167"
	}
]
```
因为我把`s2`的`_rev`参数写错了，所以未能成功删除`s2`。

### 2.3.3 更新文档
更新文档和插入文件基本是一致的，只是更新文档需要指定文档的`_rev`值，如果没有指定则会返回更新冲突。

这里其实好理解可以这么认为CouchDB是没有插入(删除)操作的，当指定`_id`的文档不存在的时候，就会插入，否则就是更新。如果更新没有指定`_rev`参数或参数值不正确，则更新就不会成功，原因是`Document update conflict`。

**CouchDB不支持对JSON文档进行部分更新，必须是全更新。也就是说不能添加或者删除字段，也不能仅更新某些字段的值。更新文档的时候`_rev 参数必须是最新获取的，因为任何修改操作都会引起`_rev`值变化。**

**curl更新单个文档示例：**
```bash
curl --request PUT \
  --url http://localhost:5984/dbtest/doc4 \
  --header 'content-type: application/json' \
  --cookie 'AuthSession=bzo1QkJEN0ZFRToKbM2PpPtuAYMsfUoiml7T1NesyQ; Version=1; Path=/; HttpOnly' \
  --data '
{
	"_rev":"4-8cf3de6bbe77006418d2f74e570be270",
    "地区": "北京北边",
	"新的键":"新的 值"
}'
```
返回和插入的返回时一样的，这里就不写了。
**curl批量更新文档示例：**
```bash
curl --request POST \
  --url http://localhost:5984/dbtest/_bulk_docs \
  --header 'content-type: application/json' \
  --cookie AuthSession=bzo1QkJEQTU4NDr5_i2OSSoponSuDz4rLu9fh4NHbw \
  --data '
{
	"docs": [
		{"_id":"s1","_rev":"7-5efc8527e2a56dc31e71abe36d365e38","desc":"bobo2","age":56.32},
		{"_id":"s2","_rev": "6-959ab718375f00a5b678ece00f68a1f7","desc":"小红2","age":19.36}
	]
}'
```
返回结果如下：
```
[
	{
		"ok": true,
		"id": "s1",
		"rev": "8-2951951ea0b4479fc1f2a0210dc83ebe"
	},
	{
		"ok": true,
		"id": "s2",
		"rev": "7-50d7fd758615aa6aefbc6aa2ec86fd8b"
	}
]
```
## 2.4 查询记录

查询记录可以查阅[http://docs.couchdb.org/en/latest/api/database/find.html](http://docs.couchdb.org/en/latest/api/database/find.html)
这里就不再详细写具体的请求过程了。

### 2.4.1 _find查询

`_find`命令用来查询摸个数据库中的文档，类似于`MongoDB`的`find`操作。
`_find`命令也使用`POST`方法请求，提交的数据是一个`JSON`对象，格式介绍如下：

**请求体JSON对象格式**

|字段|值类型|说明|
|:---|:---|:---|
|selector|object|用于选择文档的标准JSON对象。<br>相关语法将在后面详细叙述|
|limit|number|接收返回的最大结果数。默认为25，可选|
|skip|number|跳过前面的N个结果，可选|
|sort|object|排序规则，可选|
|fields|array|指定返回结果含有哪些字段的数组，省略则返回所有字段|
|use_index|string/array|指示查询使用的特定索引。<br>可以是`"<design_document>"`或 `["<design_document>", "<index_name>"]`，可选|
|r|number|读取结果的所需法定数量。(有副本的情况)默认为1，可选|
|bookmarks|string|指定所需的结果页面，用于分页结果集。默认为null，可选|
|update|boolean|是否在范围结果之前更新索引。默认为true，可选|
|stable|boolean|是否从`stable`分片集返回视图结果，可选|
|execution_stats|boolean|是否在查询响应中包含统计结果，默认false，可选|

**curl示例1，查询所有age字段在5到20之间的文档**
```bash
curl --request POST \
  --url http://localhost:5984/dbtest/_find \
  --header 'content-type: application/json' \
  --cookie AuthSession=bzo1QkJEQTU4NDr5_i2OSSoponSuDz4rLu9fh4NHbw \
  --data '{
	"selector": {
		"age": {
			"$gte": 5,
			"$lte": 20
		}
	},
	"fields": ["_id", "_rev", "age", "desc"],
	"execution_stats": true
}'
```

返回结果如下：
```
{
	"docs": [
		{
			"_id": "s2",
			"_rev": "2-50d7fd758615aa6aefbc6aa2ec86fd8b",
			"age": 19.36,
			"desc": "小红2"
		},
		{
			"_id": "s4",
			"_rev": "1-ccd6248a85ad565b2b12a3127c1b82d7",
			"age": 14,
			"desc": "ab"
		}
	],
	"bookmark": "g1AAAAA0eJzLYWBgYMpgSmHgKy5JLCrJTq2MT8lPzkzJBYkXm4BkOGAyULEsAGrUDgc",
	"execution_stats": {
		"total_keys_examined": 0,
		"total_docs_examined": 15,
		"total_quorum_docs_examined": 0,
		"results_returned": 2,
		"execution_time_ms": 1.0
	},
	"warning": "no matching index found, create an index to optimize query time"
}
```
**selector中可以使用的操作符如下：**
具体的还是建议看[官方文档](http://docs.couchdb.org/en/latest/api/database/find.html#find-selectors)，这里也只是一个摘录。

|操作符|说明|示例|
|:---|:---|:---|
|\$eq|等于|`{"selector":{"desc":{"$qe":"s2"}}}`|
|\$ne|不等于|`{"selector":{"age":{"$ne":20}}}`|
|\$gt|大于|`{"selector":{"age":{"$gt":20}}}`|
|\$gte|大于等于|`{"selector":{"age":{"$gte":20}}}`|
|\$lt|小于|`{"selector":{"age":{"$lt":20}}}`|
|\$lte|小于等于|`{"selector":{"age":{"$lte":20}}}`|
|\$regex|正则表达式|`{"selector":{"address":{"$regex":"北京.*"}}}`|
|\$and|参数为数组，表示必须同时匹配|`{"selector":{"$and":[{"age":{"$lt":20}},{"address":{"$regex":"北京.*"}}]}}`|
|\$or|参数为数组，表示必须匹配其中一个|`{"selector":{"$or":[{"age":{"$lt":20}},{"address":{"$regex":"北京.*"}}]}}`|
|\$not|表示不匹配给定选择器的结果|`{"selector":{$not:{"address":{"$regex":"北京.*"}}}}`|
|\$nor|参数为数组，表示所有选择器都不匹配，则匹配|`{"selector":{"$nor":[{"age":{"$lt":20}},{"address":{"$regex":"北京.*"}}]}}`|
|\$all|这个只能用来查询数组字段，如果文档中该数组字段的元素包含了参数数组中全部元素，则匹配|`{"selector":{{"age":{"$all":[13,14]}}`<br>这里我再次入库了一批数据，age字段是一个数组，里面都是一些整数值。对于这个例子，如果某个文档的age字段的数组元素中同时含有13、14这两个值，则匹配上|
|\$elemMatch|匹配数组字段，符合至少一个条件的文档|`{"selector":{"age":{"$elemMatch":{"$gt":5,"$lt":18}}}}`<br>如果某个文档的age字段的数组元素中，含有大于(eq)5或小于(lt)20的值，就匹配上|
|\$allMatch|匹配数组字段，符合全部条件的字段|`{"selector":{"age":{"$allMatch":{"$gt":5,"$lt":18}}}}`<br>如果某个文档的age字段的数组元素全部大于(gt)5且小于(lt)18，则匹配上|

### 2.4.2 获取记录
可以使用`_find`来查询记录，也可以直接获取记录的值。
下面的比较简单，就简单的记录一下。
1、获取单个记录
使用`GET`请求获取，在URL中指定数据库和文档`_id`。
curl示例:
```bash
curl --request GET \
  --url http://localhost:5984/dbtest/s1 \
  --cookie AuthSession=bzo1QkJEQTU4NDr5_i2OSSoponSuDz4rLu9fh4NHbw
```
2、获取数据库全部记录列表
获取全部记录列表需要使用`_all_docs`命令，这里也值演示使用`GET`方法请求的结果，还可以使用`POST`请求来操作，可以传递更多的参数来控制返回结果。
curl示例如下：
```bash
curl --request GET \
  --url http://localhost:5984/dbtest/_all_docs \
  --cookie AuthSession=bzo1QkJEQTU4NDr5_i2OSSoponSuDz4rLu9fh4NHbw
```
返回结果中只包含文档的`_id`和`_rev`字段，如果需要获取包含文档所有字段的json，则需在URL后加上请求参数`include_docs=true`示例如下：
```
{
	"total_rows": 19,
	"offset": 0,
	"rows": [
		{
			"id": "doc1",
			"key": "doc1",
			"value": {
				"rev": "1-c346e9395d9469f4a0359bbc07327a52"
			}
		},
		.... 此处省略很多....
		{
			"id": "键值",
			"key": "键值",
			"value": {
				"rev": "1-4fd4c8f7fe3d72bc579f6cfc45f6b0d4"
			}
		}
	]
}
```
3、使用`_bulk_get`命令来获取多个记录
此处可以参考官方文档[http://docs.couchdb.org/en/latest/api/database/bulk-api.html#db-bulk-get](http://docs.couchdb.org/en/latest/api/database/bulk-api.html#db-bulk-get)

这个使用和前面的批量删除差不多，也是使用`POST`方法请求，然后将请求参数放在`content`中。
JSON参数对象中的`docs`数组中的每一个元素是一个json对象，必须含有`id`键，来指定获取哪一个文档记录。可以含有`rev`键值。

**curl批量获取记录示例：**
```bash
curl --request POST \
  --url http://127.0.0.1:5984/dbtest/_bulk_get \
  --header 'content-type: application/json' \
  --cookie AuthSession=bzo1QkJEQTU4NDr5_i2OSSoponSuDz4rLu9fh4NHbw \
  --data '{
	"docs": [
		{
			"id": "s5",
			"rev":"4-753875d51501a6b1883a9d62b4d33f91"
		},
		{
			"id": "s6",
			"rev": "1-46eeb3a2fb10de49fd7cbb3c74db5361"
		},
		{
			"id": "s7"
		}
	]
}'
```
返回结果示例：
```
{
	"results": [
		{
			"id": "s5",
			"docs": [
				{
					"error": {
						"id": "s5",
						"rev": "4-753875d51501a6b1883a9d62b4d33f91",
						"error": "not_found",
						"reason": "missing"
					}
				}
			]
		},
		{
			"id": "s6",
			"docs": [
				{
					"ok": {
						"_id": "s6",
						"_rev": "1-46eeb3a2fb10de49fd7cbb3c74db5361",
						"desc": "bobo",
						"age": [
							15,
							16
						]
					}
				}
			]
		},
		{
			"id": "s7",
			"docs": [
				{
					"ok": {
						"_id": "s7",
						"_rev": "1-9f2b329717a1d2a6d6193328312e5b83",
						"desc": "小明",
						"age": [
							22,
							23
						]
					}
				}
			]
		}
	]
}
```