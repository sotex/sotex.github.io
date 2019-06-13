---
layout:  post
title:  "CPLSetErrorHandlerEx函数Bug"
date:  2018-02-06
categories:  其它
tags:  其它
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2018/02/06/8422758.html](http://www.cnblogs.com/oloroso/archive/2018/02/06/8422758.html)
CPLSetErrorHandlerEx(gdal/gdal/port/cpl_error.cpp，当前github中代码)当前函数实现如下
```cpp
CPLErrorHandler CPL_STDCALL
CPLSetErrorHandlerEx( CPLErrorHandler pfnErrorHandlerNew, void* pUserData )
{
    CPLErrorContext *psCtx = CPLGetErrorContext();
    if( psCtx == nullptr || IS_PREFEFINED_ERROR_CTX(psCtx) )
    {
        fprintf(stderr, "CPLSetErrorHandlerEx() failed.\n");
        return nullptr;
    }

    if( psCtx->psHandlerStack != nullptr )
    {
        CPLDebug( "CPL",
                  "CPLSetErrorHandler() called with an error handler on "
                  "the local stack.  New error handler will not be used "
                  "immediately." );
    }

    CPLErrorHandler pfnOldHandler = nullptr;
    {
        CPLMutexHolderD( &hErrorMutex );

        pfnOldHandler = pfnErrorHandler;

        if( pfnErrorHandler == nullptr )
            pfnErrorHandler = CPLDefaultErrorHandler;
        else
            pfnErrorHandler = pfnErrorHandlerNew;

        pErrorHandlerUserData = pUserData;
    }

    return pfnOldHandler;
}
```
这里` if( pfnErrorHandler == nullptr )`这一句判断应该改为` if( pfnErrorHandlerNew== nullptr )`。
否则调用过一次`CPLSetErrorHandlerEx(NULL,NULL)`后将无法再设置新的错误处理函数，必须再次调用使之变为`CPLDefaultErrorHandler`后方能重新设置(**如果没有重新设置，程序将出现段错误**)。
自己编译GDAL的时候可以改过了，或者调用的时候传参别传错了即可。
对于这个问题，我已经提交给GDAL开发者了，这个问题已经修正了。[CPLSetErrorHandler(): avoid later crashes when passing a null callback.](https://github.com/OSGeo/gdal/commit/e647e14bbbe3d799f54c086505baded8ad78cd6c#diff-1bb736ea00f834923a3e4176ade7c3f5)