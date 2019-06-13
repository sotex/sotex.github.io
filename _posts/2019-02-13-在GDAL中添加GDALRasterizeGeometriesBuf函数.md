---
layout:  post
title:  "在GDAL中添加GDALRasterizeGeometriesBuf函数"
date:  2019-02-13
categories:  地理信息
tags:  地理信息
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2019/02/13/10368595.html](http://www.cnblogs.com/oloroso/archive/2019/02/13/10368595.html)
# 缘起

GDAL的栅格化算法中有`GDALRasterizeLayers`、`GDALRasterizeLayersBuf`和`GDALRasterizeGeometries`函数，但是没有`GDALRasterizeGeometriesBuf`函数（GDAL项目不打算添加这个函数，因为增加一个函数会增加维护成本）。而栅格化算法的实际实现函数`gv_rasterize_one_shape`并不导出，所以在使用的时候造成了一定的不便。
虽然可以通过`MEMDataset`的方式，调用`GDALRasterizeGeometries`达到目的，但是不够直接和高效，所以我写了`GDALRasterizeGeometriesBuf`函数。
个人认为比较灵活的方式，还是将`gv_rasterize_one_shape`函数导出，以便自由使用。

# 代码

修改`gdal/alg/gdal_alg.h`头文件，在`GDALRasterizeGeometries`函数声明下添加`GDALRasterizeGeometriesBuf`函数声明。

```c
CPLErr CPL_DLL
GDALRasterizeGeometriesBuf( void *pData, int nBufXSize, int nBufYSize,
                            GDALDataType eBufType, int nPixelSpace, int nLineSpace,
                            int nGeomCount, OGRGeometryH *pahGeometries,
                            const char *pszGeomProjection,
                            const char *pszDstProjection,
                            double *padfDstGeoTransform,
                            GDALTransformerFunc pfnTransformer,
                            void *pTransformArg, double dfBurnValue,
                            char **papszOptions, GDALProgressFunc pfnProgress,
                            void *pProgressArg );
```

修改`gdal/alg/gdalrasterize.cpp`文件，添加`GDALRasterizeGeometriesBuf`函数的实现，代码如下：
```cpp
/************************************************************************/
/*                        GDALRasterizeGeometriesBuf()                      */
/************************************************************************/

/**
 * Burn geometries into raster.
 *
 * Rasterize a list of geometric objects into a raster dataset.  The
 * geometries are passed as an array of OGRGeometryH handlers.  
 *
 * If the geometries are in the georeferenced coordinates of the raster
 * dataset, then the pfnTransform may be passed in NULL and one will be
 * derived internally from the geotransform of the dataset.  The transform
 * needs to transform the geometry locations into pixel/line coordinates
 * of the target raster.
 *
 * The output raster may be of any GDAL supported datatype, though currently
 * internally the burning is done either as GDT_Byte or GDT_Float32.  This
 * may be improved in the future.
 *
 * @param pData pointer to the output data array.
 * @param nBufXSize width of the output data array in pixels.
 * @param nBufYSize height of the output data array in pixels.
 * @param eBufType data type of the output data array.
 *
 * @param nPixelSpace The byte offset from the start of one pixel value in
 * pData to the start of the next pixel value within a scanline.  If defaulted
 * (0) the size of the datatype eBufType is used.
 *
 * @param nLineSpace The byte offset from the start of one scanline in
 * pData to the start of the next.  If defaulted the size of the datatype
 * eBufType * nBufXSize is used.
 *
 * @param nGeomCount the number of geometries being passed in pahGeometries.
 * @param pahGeometries the array of geometries to burn in. 
 * @param pszGeomProjection WKT defining the coordinate system of the geometries.
 *
 * @param pszDstProjection WKT defining the coordinate system of the target
 * raster.
 *
 * @param padfDstGeoTransform geotransformation matrix of the target raster.
 *
 * @param pfnTransformer transformation to apply to geometries to put into
 * pixel/line coordinates on raster.  If NULL a geotransform based one will
 * be created internally.
 *
 * @param pTransformArg callback data for transformer.
 * @param dfBurnValue the value to burn into the raster.
 *
 * @param papszOptions special options controlling rasterization:
 * <ul>
 * <li>"ALL_TOUCHED": May be set to TRUE to set all pixels touched
 * by the line or polygons, not just those whose center is within the polygon
 * or that are selected by brezenhams line algorithm.  Defaults to FALSE.</li>
 * <li>"BURN_VALUE_FROM": May be set to "Z" to use
 * the Z values of the geometries. dfBurnValue or the attribute field value is
 * added to this before burning. In default case dfBurnValue is burned as it
 * is. This is implemented properly only for points and lines for now. Polygons
 * will be burned using the Z value from the first point. The M value may
 * be supported in the future.</li>
 * <li>"MERGE_ALG": May be REPLACE (the default) or ADD.  REPLACE
 * results in overwriting of value, while ADD adds the new value to the
 * existing raster, suitable for heatmaps for instance.</li>
 * </ul>
 *
 * @param pfnProgress the progress function to report completion.
 *
 * @param pProgressArg callback data for progress function.
 *
 *
 * @return CE_None on success or CE_Failure on error.
 */

CPLErr GDALRasterizeGeometriesBuf( void *pData, int nBufXSize, int nBufYSize,
                                   GDALDataType eBufType, int nPixelSpace, int nLineSpace,
                                   int nGeomCount, OGRGeometryH *pahGeometries,
                                   const char *pszGeomProjection,
                                   const char *pszDstProjection,
                                   double *padfDstGeoTransform,
                                   GDALTransformerFunc pfnTransformer,
                                   void *pTransformArg, double dfBurnValue,
                                   char **papszOptions, GDALProgressFunc pfnProgress,
                                   void *pProgressArg )

{
/* -------------------------------------------------------------------- */
/*      If pixel and line spaceing are defaulted assign reasonable      */
/*      value assuming a packed buffer.                                 */
/* -------------------------------------------------------------------- */
    if( nPixelSpace != 0 )
    {
        nPixelSpace = GDALGetDataTypeSizeBytes( eBufType );
    }
    if( nPixelSpace != GDALGetDataTypeSizeBytes( eBufType ) )
    {
        CPLError(CE_Failure, CPLE_NotSupported,
                    "GDALRasterizeGeometriesBuf(): unsupported value of nPixelSpace");
        return CE_Failure;
    }

    if( nLineSpace == 0 )
    {
        nLineSpace = nPixelSpace * nBufXSize;
    }
    if( nLineSpace != nPixelSpace * nBufXSize )
    {
        CPLError(CE_Failure, CPLE_NotSupported,
                    "GDALRasterizeGeometriesBuf(): unsupported value of nLineSpace");
        return CE_Failure;
    }

    if( pfnProgress == nullptr )
      pfnProgress = GDALDummyProgress;

/* -------------------------------------------------------------------- */
/*      Do some rudimentary arg checking.                               */
/* -------------------------------------------------------------------- */
    if( nGeomCount == 0 )
      return CE_None;

/* -------------------------------------------------------------------- */
/*      Options                                                         */
/* -------------------------------------------------------------------- */
    int bAllTouched = FALSE;
    GDALBurnValueSrc eBurnValueSource = GBV_UserBurnValue;
    GDALRasterMergeAlg eMergeAlg = GRMA_Replace;
    GDALRasterizeOptim eOptim = GRO_Auto;
    if( GDALRasterizeOptions(papszOptions, &bAllTouched,
                    &eBurnValueSource, &eMergeAlg,
                    &eOptim) == CE_Failure )
    {
        return CE_Failure;
    }


/* -------------------------------------------------------------------- */
/*      If we have no transformer, create the one from input file       */
/*      projection. Note that each layer can be georefernced            */
/*      separately.                                                     */
/* -------------------------------------------------------------------- */
    bool bNeedToFreeTransformer = false;

    if( pfnTransformer == nullptr )
    {
        if( !pszGeomProjection )
        {
            CPLError( CE_Warning, CPLE_AppDefined,
                        "There is no specified spatial reference for the"
                        " geometry to build transformer, assuming"
                        " matching coordinate systems.");
        }

        pTransformArg =
            GDALCreateGenImgProjTransformer3( pszGeomProjection, nullptr,
                        pszDstProjection,
                        padfDstGeoTransform );
        pfnTransformer = GDALGenImgProjTransform;
    }
/* ==================================================================== */
/*      Write geometry to the raster individually.                      */
/* ==================================================================== */
    CPLErr eErr = CE_None;

    pfnProgress( 0.0, nullptr, pProgressArg );

    int iGeometry;
    for(iGeometry = 0; iGeometry < nGeomCount; ++iGeometry )
    {
        OGRGeometry *poGeom = reinterpret_cast<OGRGeometry*>(pahGeometries[iGeometry]);

        // 后期gv_rasterize_one_shape函数可能会变，此处后期需要修改
        gv_rasterize_one_shape( static_cast<unsigned char *>(pData), 0, 0,
                    nBufXSize, nBufYSize,
                    1, eBufType, bAllTouched, poGeom,
                    &dfBurnValue, eBurnValueSource,
                    eMergeAlg,
                    pfnTransformer, pTransformArg );

        if( !pfnProgress((iGeometry+1)/static_cast<double>(nGeomCount),
                         "", pProgressArg) )
        {
            CPLError( CE_Failure, CPLE_UserInterrupt, "User terminated" );
            eErr = CE_Failure;
        }
    }

    if( bNeedToFreeTransformer )
        GDALDestroyTransformer( pTransformArg );

    return eErr;
}

```