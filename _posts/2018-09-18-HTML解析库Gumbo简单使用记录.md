---
layout:  post
title:  "HTML解析库Gumbo简单使用记录"
date:  2018-09-18
categories:  其它
tags:  其它
comments: 1
---
[TOC]
[博客园原文地址 http://www.cnblogs.com/oloroso/archive/2018/09/18/9667642.html](http://www.cnblogs.com/oloroso/archive/2018/09/18/9667642.html)



## Gumbo简介
Gumbo是谷歌开源的一个纯C编写的HTML解析库，性能很好，就是用起来比较麻烦。
[github地址https://github.com/google/gumbo-parser](https://github.com/google/gumbo-parser)
还有一个C++封装的版本[https://github.com/lazytiger/gumbo-query.git](https://github.com/lazytiger/gumbo-query.git)

关于HTML的参考，可见[https://developer.mozilla.org/zh-CN/docs/Web/HTML](https://developer.mozilla.org/zh-CN/docs/Web/HTML)

最近准备写一个爬虫，用于爬取[epsg.io](http://epsg.io)上的数据，所以找了这个库用于HTML的解析。其实我这个简单的爬取固定位置的内容，用这个实在是有点杀鸡用牛刀了，直接做字符串搜索会更方便。

## 使用记录
关于这个的使用，网上找不到太多的资料。
这里有一个[https://blog.csdn.net/fjb2080/article/details/78992851](https://blog.csdn.net/fjb2080/article/details/78992851)

![](https://img2018.cnblogs.com/blog/693958/201809/693958-20180918101341410-512644719.png)

这个图片上实际已经把相关的关系描述清楚了，我这里只做简单的补充。


### 1、GumboNode的类型
对于一个`GumboNode`结构体对象，需要通过它的`GumboNodeType type`字段判断其类型后，可根据类型对成员`v`进行操作。
`v`是一个`union`对象，它可以是`GumboDocument`、`GumboElement`、`GumboText`三种类型之一。


#### 1、GUMBO_NODE_DOCUMENT 文档节点
文档节点表示的是一个完整的html文档，就是从`<html>`到`</html>`之间的全部信息。对于`v`可取`GumboDocument`类型的成员`document`。
对于文档节点，其内部包含的元素节点都在`GumboVector children`中。

#### 2、GUMBO_NODE_ELEMENT 元素节点
只要是含有标签`tag`的部分，都是元素节点。这个可以简单的理解为，只要是`<标签名>`到`</标签名>`之间的就是一个元素节点的内容(有的元素是单标签的)，元素节点可以有包含嵌套关系，子节点都在`GumboVector children`中。
```c
/**
 * 用于表示所有HTML元素的结构。 它包含有关标记，属性和子节点的信息。
 */
typedef struct {
  /**
   * GumboNodes数组，包含此元素的子元素。 保存的是GumboNode的指针。
   */
  GumboVector /* GumboNode* */ children;

  /** 这个元素的GumboTag(标签，HTML的标签是定义好的)枚举值 */
  GumboTag tag;

  /**  此元素的GumboNamespaceEnum值(表示这个是HTML、SVG还是MATHML)*/
  GumboNamespaceEnum tag_namespace;

  /**
   * 指向此元素的原始标记文本的GumboStringPiece，直接指向源缓冲区。
   * 如果标记是通过算法插入的（例如，<head>或<tbody>插入），则这将是一个零长度字符串。
   */
  GumboStringPiece original_tag;

  /**
   * 指向此元素的原始结束标记文本的GumboStringPiece。
   * 如果以算法方式插入结束标记（例如，关闭自闭标记），则这将是一个零长度字符串。
   */
  GumboStringPiece original_end_tag;

  /** 记录元素开始标签在来源字符串中的起始位置。 */
  GumboSourcePosition start_pos;

  /** 记录元素结束标签在来源字符串中的起始位置。  */
  GumboSourcePosition end_pos;

  /**
   * GumboAttributes数组，按照解析顺序包含此元素标签的属性
   * 数组保存的是GumboAttribute的指针
   */
  GumboVector /* GumboAttribute* */ attributes;
} GumboElement;
```

#### 3、GUMBO_NODE_TEXT 文本节点
文本节点，对于`v`可取`GumboText`类型的成员`text`。
`Gumbo`在解析的时候，对于`\r\n`这种都会解析为一个独立的文件节点。


#### 4、GUMBO_NODE_CDATA
`CDATA`节点是一个比较特殊的节点，这个节点用于传输需要浏览器**不做解析，原封不动的当做文本**的内容。所以对于`v`也是取`GumboText`类型的成员`text`。

#### 5、GUMBO_NODE_COMMENT
注释节点，这个用于保存html中的注释，对于这个节点，对于`v`也是取`GumboText`类型的成员`text`。取出来的`GumboText`对象中的`text`成员**不包含注释分隔符**。

#### 6、GUMBO_NODE_WHITESPACE
这是一个文本节点的特例，文本的内容都是空白字符(空格、TAB、回车)。`v`也是取`GumboText`。

#### 7、GUMBO_NODE_TEMPLATE
模板节点。就是标签`<template>`包含的部分。
这与GUMBO_NODE_ELEMENT是分开的，因为许多客户端库都希望忽略模板节点的内容，如规范所示。 在GUMBO_NODE_ELEMENT上递归会在这里做正确的事情，而想要包含模板内容的客户端也应该检查GUMBO_NODE_TEMPLATE。 v将是一个GumboElement。

### 2、简单的使用
为了方便使用，我简单的封装了两个函数，对于我的使用已经足够了。如果需要更方便的使用，可以考虑[https://github.com/lazytiger/gumbo-query.git](https://github.com/lazytiger/gumbo-query.git)

#### 1、用于方便一点的查找子节点的
```cpp
std::vector<GumboNode*> find_sub_node(const GumboNode* parentNode,
	GumboNodeType type, GumboTag tag, int attrCount, ...)
{
	std::vector<GumboNode*> subNode;
	const GumboVector* vec = &(parentNode->v.element.children);
	for (int i = 0; i < vec->length; ++i) {
		GumboNode* node = (GumboNode*)vec->data[i];
		if (node->type != type || (type == GUMBO_NODE_ELEMENT && node->v.element.tag != tag)) {
			continue;
		}
		int pattend = 0;
		va_list vl;
		va_start(vl, attrCount);
		for (int ai = 0; ai < attrCount; ++ai) {
			const char* name = va_arg(vl, char*);
			const char* value = va_arg(vl, char*);
			GumboAttribute* attr = gumbo_get_attribute(&(node->v.element.attributes), name);
			if (attr == NULL || strcmp(value, attr->value) != 0) { continue; }
			pattend += 1;
		}
		va_end(vl);
		if (pattend == attrCount) {
			subNode.push_back(node);
		}
	}

	return subNode;
}
```
#### 2、用于方便的查找文本子节点的
```cpp
std::vector<std::string> find_sub_text(const GumboNode* parentNode)
{
	std::vector<std::string> subText;
	const GumboVector* vec = &(parentNode->v.element.children);
	for (int i = 0; i < vec->length; ++i) {
		GumboNode* node = (GumboNode*)vec->data[i];
		if (GUMBO_NODE_TEXT == node->type) {
			subText.push_back(node->v.text.text);
			continue;
		}
	}
	return subText;
}
```