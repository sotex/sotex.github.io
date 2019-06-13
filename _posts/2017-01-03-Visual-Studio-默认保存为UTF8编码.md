---
layout:  post
title:  "Visual Studio 默认保存为UTF8编码"
date:  2017-01-03
categories:  编程
tags:  编程
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2017/01/03/6245772.html](http://www.cnblogs.com/oloroso/archive/2017/01/03/6245772.html)
Visual Studio (中文版)默认保存的文本文件是`GB2312`编码（代码页936）的，默认的行尾（End of line）是CRLF的。
如果仅仅是在windows下开发问题也不大，但是涉及到跨平台开发的时候，就不是很满意了。

VS本身的 **文件 -> 高级保存选项** 中是可以选择保存的编码和行尾的，但是不支持为默认的。
还有一个问题是`cl`编译的时候，对`utf-8`格式支持不好（需要添加`/source-charset:utf-8`选项，默认是当作本地字符集的），对于带`BOM`标记的文件则没有问题。

所以我们在项目中统一规定使用`UTF-8 with BOM`编码，行尾为`LF`(\n)。

这里介绍两个插件

### ForceUTF8 (with BOM)

这个插件还有两个版本，一个是带`BOM`的，一个是不带的。
插件是开源的，代码很简单。就是在文档保存的时候，判断是否是文本文件。如果是的话，那就先转编码为`UTF-8 with BOM`，再写入文件。

下载地址 [https://marketplace.visualstudio.com/items?itemName=jz5.ForceUTF8withBOM](https://marketplace.visualstudio.com/items?itemName=jz5.ForceUTF8withBOM)

其实可以直接在这个项目上改，在保存文件前把`\r\n`、`\r`、`\n`都替换为`\n`即可(要注意替换次序)。

### Line Endings Unifier

这个插件用来统一行尾。
可以设置针对的文件和目标行尾。它也是开源的。

下载地址 [https://marketplace.visualstudio.com/items?itemName=JakubBielawa.LineEndingsUnifier](https://marketplace.visualstudio.com/items?itemName=JakubBielawa.LineEndingsUnifier)

![LEU_SCREEN](https://jakubbielawa.gallerycdn.vsassets.io/extensions/jakubbielawa/lineendingsunifier/1.4.9/1482141663380/228425/1/LEU_SCREEN.JPG)