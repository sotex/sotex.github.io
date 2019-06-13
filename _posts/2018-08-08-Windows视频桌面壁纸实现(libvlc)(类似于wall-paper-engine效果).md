---
layout:  post
title:  "Windows视频桌面壁纸实现(libvlc)(类似于wall paper engine效果)"
date:  2018-08-08
categories:  其它
tags:  其它
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2018/08/08/9446299.html](http://www.cnblogs.com/oloroso/archive/2018/08/08/9446299.html)
# 简介
这个项目是很久之前的事情了，当时一个朋友正在研究一个国外的软件（wall paper engine ），可以在桌面壁纸层播放视频，也就差不多是动态壁纸的意思。
后来我也动手来实现这个功能，因为手头一直有别的事，也就没有一直弄。
最近看到这个项目的工程文件，就继续拿起来改了改了，稍微完善了一下，基本能用了。

编译后的程序下载链接: [Win64](https://pan.baidu.com/s/1HN3wjQu8AX2L2x9JfV0UtQ) 密码: 6bxk

2018年8月22日更新
代码已经上传到Gitee [https://gitee.com/solym/video_wallpaper.git](https://gitee.com/solym/video_wallpaper.git)，GitHub地址[https://github.com/sotex/VideoWallpaper](https://github.com/sotex/VideoWallpaper)。

## 原理
原理比较简单，就是找到桌面壁纸层的句柄，然后将VLC的播放窗口设置为桌面壁纸层窗口。
在windows 10上使用的时候，会发现找不到桌面壁纸层，或者说找不到与图标层分离的壁纸层。这个解决的办法在这里[https://www.codeproject.com/articles/856020/draw-behind-desktop-icons-in-windows](https://www.codeproject.com/articles/856020/draw-behind-desktop-icons-in-windows)可以找到。

## 效果如下
![](https://images2018.cnblogs.com/blog/693958/201808/693958-20180808225926006-2055719741.jpg)