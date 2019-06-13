---
layout:  post
title:  "修改QGIS来支持DPI为96的WMTS/WMS服务"
date:  2018-08-29
categories:  地理信息
tags:  地理信息
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2018/08/29/9553207.html](http://www.cnblogs.com/oloroso/archive/2018/08/29/9553207.html)
## 缘由
因为各种各种wmts地图客户端产品的标准的支持不一定是一致的，就像ArcGIS不同版本加载WMTS图层的时候计算的规则就有差别（米和经纬度之间转换系数的区别），导致会出现适应各个客户端而出的WMTS服务，里面的`ScaleDenominator`值有细微的差别。
例如都是Google经纬度(天地图经纬度直投)，其切分规则是一样的，但其DPI不一致导致计算的`ScaleDenominator`是不一致的。
虽然OGC标准里面对DPI大小的定义就是`90.714`，但是国内天地图使用的DPI标准为`96`，这里也有一个计算的差值。DPI不同对应的也就是每个像素的大小不一致，也就是分辨率不一致，造成瓦片的行列号计算有差异。

QGIS是支持`OGC WMTS标准`的，其DPI的大小就是`90.714`(像素大小0.00028)。为了让QGIS能够支持DPI为`96`的WMTS服务，需要对QGIS进行一点修改。

## 解决过程

为了解决这个问题，看了一下`QGIS`的源码，相关的定义如下：
**文件[qgis\src\providers\wms\qgswmscapabilities.cpp](https://github.com/qgis/QGIS/blob/master/src/providers/wms/qgswmscapabilities.cpp)  1382行前后**
```cpp
      m.tileWidth    = e1.firstChildElement( QStringLiteral( "TileWidth" ) ).text().toInt();
      m.tileHeight   = e1.firstChildElement( QStringLiteral( "TileHeight" ) ).text().toInt();
      m.matrixWidth  = e1.firstChildElement( QStringLiteral( "MatrixWidth" ) ).text().toInt();
      m.matrixHeight = e1.firstChildElement( QStringLiteral( "MatrixHeight" ) ).text().toInt();

      // the magic number below is "standardized rendering pixel size" defined
      // in WMTS (and WMS 1.3) standard, being 0.28 pixel
      m.tres = m.scaleDenom * 0.00028 / metersPerUnit;

      QgsDebugMsg( QString( " %1: scale=%2 res=%3 tile=%4x%5 matrix=%6x%7 topLeft=%8" )
                   .arg( m.identifier )
                   .arg( m.scaleDenom ).arg( m.tres )
                   .arg( m.tileWidth ).arg( m.tileHeight )
                   .arg( m.matrixWidth ).arg( m.matrixHeight )
                   .arg( m.topLeft.toString() )
                 );

      s.tileMatrices.insert( m.tres, m );
```
这里可以看到通过`ScaleDenominator`计算瓦片分辨率的过程，这里使用的像素大小是`0.00028`。
因为像素大小`0.00028`的计算是`0.0254/DPI`(这个原理网上搜就有了)，所以我们可以算出DPI为96时候的像素大小为`0.000264583`。
如果这里我们把`0.00028`修改为`0.000264583`再编译就可以支持天地图的了。这里我不打算改它，因为编译太麻烦。

这里我们直接改`QGIS`的二进制程序文件好了。
使用十六进制编辑器打开`C:\Program Files\QGIS 3.0\apps\qgis\plugins`目录下在`wmsprovider.dll`(最好先备份)。
然后搜索`0.00028`

![](https://images2018.cnblogs.com/blog/693958/201808/693958-20180829110951077-275889507.jpg)

整个文件就这一处使用了`0.00028`这个数值，这里直接将其修改为`0.000264583`保存即可。

再打开`QGIS`就可以按照96的DPI加载WMTS和WMS图层了。

还可以保留着两个`dll`文件，写一个批处理来决定加载的时候使用哪一个来运行。

## ArcGIS支持DPI 96
ARCGIS中一样没有找到相关的设置，猜测也是写到代码里面了。听闻国内某些单位购买的ArcGIS是支持96的DPI的，但我们没有。
ArcGIS安装目录下找到`WMSLayer.dll`和`WMSService.dll`的文件，还是一样的套路，搜索`0.00028`修改为`0.000264583`即可。

![](https://images2018.cnblogs.com/blog/693958/201808/693958-20180829113223416-1750621936.jpg)