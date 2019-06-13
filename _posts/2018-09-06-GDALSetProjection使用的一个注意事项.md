---
layout:  post
title:  "GDALSetProjection使用的一个注意事项"
date:  2018-09-06
categories:  地理信息
tags:  地理信息
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2018/09/06/9597067.html](http://www.cnblogs.com/oloroso/archive/2018/09/06/9597067.html)
## GDALSetProjection 简述
`GDALSetProjection`是用来给`GDALDataset`设定投影信息(坐标系统)的接口，实际上是`GDALDataset::SetProjection`这个虚函数的转调而已。官网文档描述如下：
   > **CPLErr GDALDataset::SetProjection (const char *  	pszProjection	) **
   > Set the projection reference string for this dataset.
   > The string should be in OGC WKT or PROJ.4 format. An error may occur because of incorrectly specified projection strings, because the dataset is not writable, or because the dataset does not support the indicated projection. Many formats do not support writing projections.
   > This method is the same as the C `GDALSetProjection()` function.
   > **Parameters**
   >        pszProjection	projection reference string.
   >**Returns**
   >        CE_Failure if an error occurs, otherwise CE_None.

按照这里描述的，传入的参数可以是`OGC WKT`或者`PROJ.4`格式的字符串，但是可能在写入坐标系信息的时候发生错误，因为数据集不可写。或者数据集不支持对应的坐标系，很多格式不支持写入。

## 注意事项

这里要说的就是这个函数使用的时候要注意的地方，因为这是一个虚函数，在基类`GDALDataset`中的定义是直接返回`CE_Failure`的，派生出的各个格式有自己的实现。
因为每个派生出的格式对`SetProjection`的实现可能都不一致，所以这里使用的时候就需要根据不同的格式传入不同的参数。
例如`GTiff`就不支持`PROJ.4`格式的字符串，只能是`OGC WKT`格式的。
又比如，`MBTiles`格式支持`EPSG:3857`，其他的就不支持了。
又比如，`NTIF`格式，也仅支持`WKT`格式字符串传入，且只支持`WGS84`地理坐标或者`UTM`投影坐标。
OGC出的`GeoPackage`格式是支持最好的，支持各种用户输入格式，支持各种坐标系，几乎没有限制。
`HDF4`格式就很简单粗暴了，不做任何验证，传入什么就复制什么.

下面贴出`GTiff`和`MBTiles`两种格式对`SetProjection`的实现。
**GDALDataset::SetProjection**
```cpp
CPLErr GDALDataset::SetProjection( CPL_UNUSED const char *pszProjection )
{
    if( !(GetMOFlags() & GMO_IGNORE_UNIMPLEMENTED) )
        ReportError(CE_Failure, CPLE_NotSupported,
                    "Dataset does not support the SetProjection() method.");
    return CE_Failure;
}
```

**GTiffDataset::SetProjection**
```cpp
CPLErr GTiffDataset::SetProjection( const char * pszNewProjection )

{
    if( bStreamingOut && bCrystalized )
    {
        CPLError(
            CE_Failure, CPLE_NotSupported,
            "Cannot modify projection at that point in "
            "a streamed output file" );
        return CE_Failure;
    }

    LoadGeoreferencingAndPamIfNeeded();
    LookForProjection();

    if( !STARTS_WITH_CI(pszNewProjection, "GEOGCS")
        && !STARTS_WITH_CI(pszNewProjection, "PROJCS")
        && !STARTS_WITH_CI(pszNewProjection, "LOCAL_CS")
        && !STARTS_WITH_CI(pszNewProjection, "COMPD_CS")
        && !STARTS_WITH_CI(pszNewProjection, "GEOCCS")
        && !EQUAL(pszNewProjection,"") )
    {
        CPLError( CE_Failure, CPLE_AppDefined,
                "Only OGC WKT Projections supported for writing to GeoTIFF.  "
                "%s not supported.",
                  pszNewProjection );

        return CE_Failure;
    }

    if( EQUAL(pszNewProjection, "") &&
        pszProjection != NULL &&
        !EQUAL(pszProjection, "") )
    {
        bForceUnsetProjection = true;
    }

    CPLFree( pszProjection );
    pszProjection = CPLStrdup( pszNewProjection );

    bGeoTIFFInfoChanged = true;

    return CE_None;
}
```


**MBTilesDataset::SetProjection**
```cpp
CPLErr MBTilesDataset::SetProjection( const char* pszProjection )
{
    if( eAccess != GA_Update )
    {
        CPLError(CE_Failure, CPLE_NotSupported,
                 "SetProjection() not supported on read-only dataset");
        return CE_Failure;
    }

    OGRSpatialReference oSRS;
    if( oSRS.SetFromUserInput(pszProjection) != OGRERR_NONE )
        return CE_Failure;
    if( oSRS.GetAuthorityName(NULL) == NULL ||
        !EQUAL(oSRS.GetAuthorityName(NULL), "EPSG") ||
        oSRS.GetAuthorityCode(NULL) == NULL ||
        !EQUAL(oSRS.GetAuthorityCode(NULL), "3857") )
    {
        CPLError(CE_Failure, CPLE_NotSupported,
                 "Only EPSG:3857 supported on MBTiles dataset");
        return CE_Failure;
    }
    return CE_None;
}
```