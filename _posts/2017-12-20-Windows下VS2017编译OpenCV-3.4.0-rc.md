---
layout:  post
title:  "Windows下VS2017编译OpenCV 3.4.0-rc"
date:  2017-12-20
categories:  编程
tags:  编程
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2017/12/20/8074783.html](http://www.cnblogs.com/oloroso/archive/2017/12/20/8074783.html)
## 简述
很久没有用过OpenCV了，这次需要做一点图像处理相关的工作，又需要用起来，这里记录一下编译的过程。[之前介绍过使用vs2015编译opencv2.4的帖子在这里](http://www.cnblogs.com/oloroso/p/5689987.html)。
编译好的文件在这里[https://pan.baidu.com/s/1qXCWxkw](https://pan.baidu.com/s/1qXCWxkw)

## 1、下载源码
这里就不下载源码压缩包了，直接从github上克隆一下。
```bash
# 因为访问github较慢，这里直接使用的国内码云同步仓库
git clone https://gitee.com/mirrors/opencv.git
# github上的地址为：https://github.com/opencv/opencv.git
```

克隆之后可以将3.4.0-rc分支打包出来，你也可以直接切换到3.4.0-rc分支或者直接使用master的代码。

```bash
cd opencv
git archive -o ../opencv3.4.0.zip 3.4.0-rc
```
打包出来的压缩包是不含git仓库的相关文件的，不是很大，可以解压到你想解压的目录。

## 2、使用cmake生成VS工程
我这里就没有使用命令行，直接使用的`cmake-gui`进行的配置。
选择源码目录和构建目录之后，点击`configure`按钮，然后选择编译器为`visual studio 2015 2017 win64`，等待配置结束。（配置的过程中会去下载**opencv_ffmpeg.dll**等文件，过程可能比较慢）

我使用的构建选项改变如下（Tools-->My Changes）
```
Commandline options:
-DBUILD_JAVA:BOOL="0" -DENABLE_LTO:BOOL="1" -DWITH_GSTREAMER:BOOL="0" -DCPACK_BINARY_ZIP:BOOL="1" -DBUILD_TESTS:BOOL="0" -DENABLE_CXX11:BOOL="1" -DBUILD_PERF_TESTS:BOOL="0" -DCPACK_SOURCE_7Z:BOOL="0" -DCPACK_BINARY_NSIS:BOOL="0" 


Cache file:
BUILD_JAVA:BOOL=0
ENABLE_LTO:BOOL=1
WITH_GSTREAMER:BOOL=0
CPACK_BINARY_ZIP:BOOL=1
BUILD_TESTS:BOOL=0
ENABLE_CXX11:BOOL=1
BUILD_PERF_TESTS:BOOL=0
CPACK_SOURCE_7Z:BOOL=0
CPACK_BINARY_NSIS:BOOL=0
```
配置输出信息如下
```
General configuration for OpenCV 3.4.0-rc =====================================
  Version control:               unknown

  Platform:
    Timestamp:                   2017-12-20T08:35:12Z
    Host:                        Windows 10.0.14393 AMD64
    CMake:                       3.7.2
    CMake generator:             Visual Studio 15 2017 Win64
    CMake build tool:            C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/MSBuild/15.0/Bin/MSBuild.exe
    MSVC:                        1912

  CPU/HW features:
    Baseline:                    SSE SSE2 SSE3
      requested:                 SSE3
    Dispatched code generation:  SSE4_1 SSE4_2 FP16 AVX AVX2
      requested:                 SSE4_1 SSE4_2 AVX FP16 AVX2
      SSE4_1 (3 files):          + SSSE3 SSE4_1
      SSE4_2 (1 files):          + SSSE3 SSE4_1 POPCNT SSE4_2
      FP16 (1 files):            + SSSE3 SSE4_1 POPCNT SSE4_2 FP16 AVX
      AVX (5 files):             + SSSE3 SSE4_1 POPCNT SSE4_2 AVX
      AVX2 (9 files):            + SSSE3 SSE4_1 POPCNT SSE4_2 FP16 FMA3 AVX AVX2

  C/C++:
    Built as dynamic libs?:      YES
    C++11:                       YES
    C++ Compiler:                C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.12.25827/bin/Hostx86/x64/cl.exe  (ver 19.12.25831.0)
    C++ flags (Release):         /DWIN32 /D_WINDOWS /W4 /GR  /EHa  /D _CRT_SECURE_NO_DEPRECATE /D _CRT_NONSTDC_NO_DEPRECATE /D _SCL_SECURE_NO_WARNINGS /Gy /bigobj /GL /Oi      /wd4251 /wd4324 /wd4275 /wd4512 /wd4589 /MP4  /MD /O2 /Ob2 /DNDEBUG  /Zi
    C++ flags (Debug):           /DWIN32 /D_WINDOWS /W4 /GR  /EHa  /D _CRT_SECURE_NO_DEPRECATE /D _CRT_NONSTDC_NO_DEPRECATE /D _SCL_SECURE_NO_WARNINGS /Gy /bigobj /GL /Oi      /wd4251 /wd4324 /wd4275 /wd4512 /wd4589 /MP4  /D_DEBUG /MDd /Zi /Ob0 /Od /RTC1 
    C Compiler:                  C:/Program Files (x86)/Microsoft Visual Studio/2017/Community/VC/Tools/MSVC/14.12.25827/bin/Hostx86/x64/cl.exe
    C flags (Release):           /DWIN32 /D_WINDOWS /W3  /D _CRT_SECURE_NO_DEPRECATE /D _CRT_NONSTDC_NO_DEPRECATE /D _SCL_SECURE_NO_WARNINGS /Gy /bigobj /GL /Oi        /MP4  /MD /O2 /Ob2 /DNDEBUG  /Zi
    C flags (Debug):             /DWIN32 /D_WINDOWS /W3  /D _CRT_SECURE_NO_DEPRECATE /D _CRT_NONSTDC_NO_DEPRECATE /D _SCL_SECURE_NO_WARNINGS /Gy /bigobj /GL /Oi        /MP4  /D_DEBUG /MDd /Zi /Ob0 /Od /RTC1 
    Linker flags (Release):      /machine:x64  /LTCG /INCREMENTAL:NO  /debug
    Linker flags (Debug):        /machine:x64  /LTCG /debug /INCREMENTAL 
    ccache:                      NO
    Precompiled headers:         YES
    Extra dependencies:
    3rdparty dependencies:

  OpenCV modules:
    To be built:                 calib3d core dnn features2d flann highgui imgcodecs imgproc ml objdetect photo python_bindings_generator shape stitching superres ts video videoio videostab
    Disabled:                    js world
    Disabled by dependency:      -
    Unavailable:                 cudaarithm cudabgsegm cudacodec cudafeatures2d cudafilters cudaimgproc cudalegacy cudaobjdetect cudaoptflow cudastereo cudawarping cudev java python2 python3 viz
    Applications:                apps
    Documentation:               YES (C:/Program Files/doxygen/bin/doxygen.exe 1.8.10)
    Non-free algorithms:         NO

  Windows RT support:            NO

  GUI: 
    Win32 UI:                    YES
    VTK support:                 NO

  Media I/O: 
    ZLib:                        build (ver 1.2.11)
    JPEG:                        build (ver 90)
    WEBP:                        build (ver encoder: 0x020e)
    PNG:                         build (ver 1.6.34)
    TIFF:                        build (ver 42 - 4.0.9)
    JPEG 2000:                   build (ver 1.900.1)
    OpenEXR:                     build (ver 1.7.1)

  Video I/O:
    Video for Windows:           YES
    DC1394:                      NO
    FFMPEG:                      YES (prebuilt binaries)
      avcodec:                   YES (ver 57.89.100)
      avformat:                  YES (ver 57.71.100)
      avutil:                    YES (ver 55.58.100)
      swscale:                   YES (ver 4.6.100)
      avresample:                YES (ver 3.5.0)
    DirectShow:                  YES

  Parallel framework:            Concurrency

  Trace:                         YES (with Intel ITT)

  Other third-party libraries:
    Intel IPP:                   2017.0.3 [2017.0.3]
           at:                   C:/OpenCV/opencv3.4.0/build/3rdparty/ippicv/ippicv_win
    Intel IPP IW:                sources (2017.0.3)
              at:                C:/OpenCV/opencv3.4.0/build/3rdparty/ippicv/ippiw_win
    Lapack:                      NO
    Eigen:                       NO
    Custom HAL:                  NO

  NVIDIA CUDA:                   NO

  OpenCL:                        YES (no extra features)
    Include path:                C:/OpenCV/opencv3.4.0/3rdparty/include/opencl/1.2
    Link libraries:              Dynamic load

  Python (for build):            C:/Program Files/Python36/python.exe

  Matlab:                        NO

  Install to:                    C:/OpenCV/opencv3.4.0/build/install
-----------------------------------------------------------------

Configuring done
```
配置完成后点击`Generate`按钮创建VS解决方案即可。

## 3、编译
你可以直接进入cmake的构建输出目录，双击`OpenCV.sln`使用VS2017打开，然后构建。
也可以使用命令行进行编译
```cmd
# Release版构建命令如下
C:\OpenCV\opencv3.4.0\build>msbuild /p:configuration=Release /maxcpucount:4 OpenCV.sln
# Debug版本只需要将上面的Release改为Debug即可
# 构建完成后，使用下面命令进行安装(安装输出到build下的install目录，实际上面构建完成就已经安装)
C:\OpenCV\opencv3.4.0\build>msbuild /p:configuration=Release /maxcpucount:4  INSTALL
```
编译完成安装后，即可到`build/install`目录下查看相关的头文件和库文件。


## 4、遇到的错误和解决办法
1、`perl --version`错误
```cmd
       “C:\OpenCV\opencv3.4.0\build\OpenCV.sln”(默认目标) (1) ->
       “C:\OpenCV\opencv3.4.0\build\doc\doxygen_cpp.vcxproj.metaproj”(默认目标) (50) ->
       “C:\OpenCV\opencv3.4.0\build\doc\doxygen_cpp.vcxproj”(默认目标) (52) ->
       (CustomBuild 目标) ->
         CUSTOMBUILD : error : Problems running bibtex. Verify that the command 'perl --version' works from the command
        line. Exit code: 1 [C:\OpenCV\opencv3.4.0\build\doc\doxygen_cpp.vcxproj]

    46 个警告
    1 个错误
```
这个错误与`docgen_cpp.vcxproj`这个工程相关，那么是生成文档相关的，这里就不该了，直接去掉`doxygen`文档生成选项。
在命令行参数中添加`-DBUILD_DOCS:BOOL="0"`或者在`cmake-gui`中将`BUILD_DOC`选中的勾去掉。
然后重新`Configure`再`Generate`一下。然后重新编译即可。