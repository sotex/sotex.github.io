---
layout:  post
title:  "GDALBuildVRT异构波段的支持"
date:  2018-01-12
categories:  地理信息
tags:  地理信息
comments: 1
---
[TOC]
[博客园原文地址 http://www.cnblogs.com/oloroso/archive/2018/01/12/8275454.html](http://www.cnblogs.com/oloroso/archive/2018/01/12/8275454.html)



## 简述
GDALBuildVRT工具和函数默认都是不支持异构波段的（即要建立虚拟影像目录的文件中有波段与其它有异的，少于指定波段等特征的），公司一个项目中需要支持这个特性，不足的以第1波段替代（除非波段数为0，那这在影像质检的时候就已经检查完了），这里记录一下修改方式。

## 修改源码
使用的GDAL版本是`2.2.3`，修改的源码文件为`gdal-2.2.3\apps\gdalbuildvrt_lib.cpp`。

### 1、修改DatasetProperty结构体
修改这个结构体，增加用于记录每个影像波段数的成员变量，修改结果如下

大致在源文件的`75-90`行
```cpp
typedef struct
{
    int    isFileOK;
    int    nRasterXSize;
    int    nRasterYSize;
    double adfGeoTransform[6];
    int    nBlockXSize;
    int    nBlockYSize;
    GDALDataType firstBandType;
    int*         panHasNoData;
    double*      padfNoDataValues;
    int    bHasDatasetMask;
    int    nMaskBlockXSize;
    int    nMaskBlockYSize;
    int    nRasterBandCount;   /*新增，记录影像波段数*/
} DatasetProperty;
```

### 2、修改VRTBuilder::AnalyseRaster函数

大致在源文件的`406-789`行
修改这个函数主要是去除对异构波段的检测，同时加上对来源影像波段数的记录。

将以下代码
大致在源文件的`543`行
```cpp
int _nBands = GDALGetRasterCount(hDS);
```
修改为如下，记录来源影像的波段数
```cpp
int _nBands = psDatasetProperties->nRasterBandCount =GDALGetRasterCount(hDS);
```

将以下代码注释掉，跳过波段数的检测
大致在源文件的`685-690`行
```cpp
        if (!bSeparate)
        {
            // if (nMaxBandNo > _nBands)
            // {
            //     CPLError(CE_Warning, CPLE_NotSupported,
            //                 "gdalbuildvrt does not support heterogeneous band numbers. Skipping %s",
            //             dsFileName);
            //     return FALSE;
            // }
```
将以下代码注释，去掉对波段颜色特征信息的匹配
大致在源文件的`695`行
```cpp
            for(j=0;j<nMaxBandNo;j++)
            {
				int bandNo = j + 1;
				if(bandNo > _nBands){ bandNo = 1;}
                GDALRasterBandH hRasterBand = GDALGetRasterBand( hDS, bandNo );
                /*去掉对波段颜色特征信息的比对*/
                if (/* pasBandProperties[j].colorInterpretation != GDALGetRasterColorInterpretation(hRasterBand) || */
                    pasBandProperties[j].dataType != GDALGetRasterDataType(hRasterBand))
                {
```

###3、修改VRTBuilder::CreateVRTNonSeparate函数
因为我们不做每个影像单独一个波段的版本，所以这里只修改NonSeparate版本。
这里主要修改VRT波段配置源(poVRTBand->ConfigureSource)时候的来源影像波段的选择，修改的地方不多，这里贴出稍微完整点的带注释的代码。
大致在源文件的`950-978`行
```cpp
        for(int j=0;j<nBands;j++)
        {
           /* 获取VRT的第j+1波段 */
            VRTSourcedRasterBandH hVRTBand =
                    (VRTSourcedRasterBandH)GDALGetRasterBand(hVRTDS, j + 1);
            /* Place the raster band at the right position in the VRT 将栅格波段置于VRT中的正确位置 */
           /* 获取源影像的波段索引，如果源影像的波段数小于j+1了，那就使用第一波段 */
	    int sourceBandIndex = (j + 1) > psDatasetProperties->nRasterBandCount ? 0 : j;
            int nSelBand = panBandList[sourceBandIndex ] - 1;  // 修改的位置也就这两行

            VRTSourcedRasterBand* poVRTBand = (VRTSourcedRasterBand*)hVRTBand;

           /* 检测源影像的Nodata信息 */
            VRTSimpleSource* poSimpleSource;
            if (bAllowSrcNoData && psDatasetProperties->panHasNoData[nSelBand])
            {
                poSimpleSource = new VRTComplexSource();
                poSimpleSource->SetNoDataValue( psDatasetProperties->padfNoDataValues[nSelBand] );
            }
            else
                poSimpleSource = new VRTSimpleSource();
            if( pszResampling )
                poSimpleSource->SetResampling(pszResampling);
            /*对VRT波段配置正确的源信息*/
            poVRTBand->ConfigureSource( poSimpleSource,
                                        (GDALRasterBand*)GDALGetRasterBand((GDALDatasetH)hProxyDS, nSelBand + 1),
                                        FALSE,
                                        dfSrcXOff, dfSrcYOff,
                                        dfSrcXSize, dfSrcYSize,
                                        dfDstXOff, dfDstYOff,
                                        dfDstXSize, dfDstYSize );
            /*添加源*/
            poVRTBand->AddSource( poSimpleSource );
        }
```