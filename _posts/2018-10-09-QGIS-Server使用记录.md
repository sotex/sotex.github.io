---
layout:  post
title:  "QGIS Server使用记录"
date:  2018-10-09
categories:  地理信息
tags:  地理信息
comments: 1
---
[TOC]
[博客园原文地址 http://www.cnblogs.com/oloroso/archive/2018/10/09/QGIS.html](http://www.cnblogs.com/oloroso/archive/2018/10/09/QGIS.html)



## 0. 简述
关于QGIS Server相关的文档很少，我也没有找到其源码在哪里，所以有些问题也不知道怎么解决，只能慢慢摸索。
这里只记录了在windows 10上安装使用的过程，在linux下过程也差不多，但是简单多了，很多缺失的东西可以直接命令安装。
我这里使用了最高版本的，但是最好还是使用长期版本，没有这么多问题。

参考:
- https://anitagraser.com/category/gis/qgis-server/
- https://github.com/qgis/docker-qgis-server

## 1. 下载QGIS桌面64位版本

也可以不下载，使用OSGeo4W在线安装程序，只安装qgis-server但我测试这个安装不全。我安装是最新的3.2.3版本。
![](https://img2018.cnblogs.com/blog/693958/201810/693958-20181009105538918-539805949.png)
使用在线安装的时候，下载站点最好选择`http://download.osgeo.org`，这个网站下载最快。

下载地址： https://qgis.org/downloads/QGIS-OSGeo4W-3.2.3-1-Setup-x86_64.exe 
安装的时候最好不要安装在C盘（win10下会很多地方有权限问题），安装路径中也最好不要有空格，原因后面会提到。

## 2. 下载安装QGIS Server程序

下载地址：http://download.osgeo.org/osgeo4w/x86_64/release/qgis/qgis-server/
这里没有找到32位版本程序的下载，只能下载64位版本。下载的版本要与桌面版本一致。

下载之后解压到QGIS的安装目录即可，压缩包内的目录结构与QGIS安装目录结构是对应的。
然后复制一份`httpd.d`目录下的`httpd_qgis.conf.tmpl`文件,改名为`httpd_qgis.conf`。
![](https://img2018.cnblogs.com/blog/693958/201810/693958-20181009105248015-582934325.jpg)

然后编辑这个新文件
-  将里面的`@osgeo4w@`替换为`QGIS的安装目录`
-  将`@grassversion@`替换为`grass的版本号`
- 将`@windir@`替换为windows的目录。
或者直接运行一下QGIS安装目录下的`etc/postinstall`下的`qgis-server.bat`脚本即可。
修改的结果大致如下：
![](https://img2018.cnblogs.com/blog/693958/201810/693958-20181009110355979-1181228423.jpg)

## 3. 下载安装Apache服务器

qgis server实质上是一个cgis程序，所以需要Apache服务器来调用。
Apache下载地址:https://www.apachelounge.com/download/
下载后直接解压即可，注意不要解压到有空格的目录。

解压之后修改`conf`目录下在`httpd.conf`
首先修改最前面的`SVRROOT`变量值
```
Define SRVROOT "C:/Apache24"
ServerRoot "${SRVROOT}"
``` 
然后在最后位置，把`httpd_qgis.conf`包含进去
```
include "C:/Program Files/QGIS 3.2/httpd.d/*.conf"
```
因为`qgis server`是一个`fastcgi`程序，所以这里需要下载apache的fastcgi模块
下载地址：https://www.apachelounge.com/download/
注意要下载与apache对应的版本。
下载之后解压到apache目录下的`modules`目录下即可。

## 4、使用及问题处理
完成上面的步骤之后，可以启动apache安装目录下的`bin/httpd.exe`程序了。
如果没有报错，则可以获取一下`GetCapabilities`试试
![](https://img2018.cnblogs.com/blog/693958/201810/693958-20181009112444180-1814600847.png)
**这里我在httpd.conf中修改了端口，所以访问的是8080端口。**


```
Forbidden

You don't have permission to access /qgis/qgis_mapserv.fcgi.exe on this server.
```
这里可以看到，返回了`403`错误，这里说明这个文件是存在的，只是被apache禁止访问了。
这里修改下apache的`httpd.conf`文件
```
<Directory />
    AllowOverride none
    Require all denied
</Directory>
```
修改为
```
<Directory />
    AllowOverride none
    Require all granted
</Directory>
```
或者修改`httod_qgis.conf`文件，添加`Require all granted`即可。

重启`httpd.exe`后继续测试，这时候发现返回的错误码变成了`503`。
![](https://img2018.cnblogs.com/blog/693958/201810/693958-20181009112720143-1346373173.png)

这时候可以打开apache的日志看看错误的原因。
![](https://img2018.cnblogs.com/blog/693958/201810/693958-20181009112941317-1859090827.png)

这里可以看到，还是这个路径中有空格的原因。
**这个问题两个解决办法**
- 一是把`qgis_mapserv.fcgi.exe`程序文件复制到Apache安装目录下的`cgi-bin`目录中，然后访问`http://127.0.0.1:8080/cgi-bin/qgis_mapserv.fcgi.exe?SERVICE=WMS&VERSION=1.3.0&REQUEST=GetCapabilities`。
- 二是重新安装qgis到没有空格的路径中去。

然后继续测试，还可以遇到问题。
如下：
![](https://img2018.cnblogs.com/blog/693958/201810/693958-20181009130936659-1729794829.png)
这个问题也很好解决，这是因为`qgis_mapserv.fcgi.exe`编译的时候依赖的这些动态库并没有打包到里面来，需要自己去下载过来。同样缺失的还有`QtXml4.dll`、`QtCore4.dll`、`QtGui4.dll`、`QtNetwork4.dll`、`QtSvg4.dll`，这些dll在QGIS2.18的安装目录下可以找到。我这里打包上传一下，地址在这里[qgis_mapserv.fcgi.exe-depends.7z](https://files.cnblogs.com/files/oloroso/qgis_mapserv.fcgi.exe-depends.7z)。也可以使用`OSGeo4W在线安装`程序下载。

继续测试，这里还会碰到问题：
![](https://img2018.cnblogs.com/blog/693958/201810/693958-20181009133532220-1372011214.png)
`offscreen`插件是Qt用于**离屏渲染**的，在Qt5.1版本才提供。
打开`httpd_qgis.conf`文件，可以看到在定义Qt插件路径变量`QT_PLUGIN_PATH`的值的时候，里面对应路径是`qt4`的，这里只需要把`4`改为`5`即可。
修改完成之后，重启`httpd.exe`继续测试。

**经过上面处理之后，qgis_mapserv.fcgi.exe确实能够被调用运行了，这个通过`ProcessMinitor`可以监测到调用过程。**
按照网上一些资料的说法，将QGIS的工程文件`xxx.qgs`放入`qgis_mapserv.fcgi.exe`所在目录，就可以用于提供默认的服务，不必指定`map`选项参数。但是我测试GetCapabilities的结果没有成功，程序执行后返回500错误，也就是`qgis_mapserv.fcgi.exe`程序没有正确返回对应的XML内容，在请求的时候添加`map=xxx.qgs`或`map=xxx`也都是失败的。因为没有相关的文档和源码，这个还需要继续测试。