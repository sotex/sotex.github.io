---
layout: post
title:  "Qt Multimedia Backends（多媒体后端）翻译"
date:   2019-04-30
categories: Qt
tags: Qt QtAV Multimedia
comments: 1
---
# Qt Multimedia Backends（多媒体后端）翻译

[TOC]

[博客园文章地址 https://www.cnblogs.com/oloroso/p/10795485.html](https://www.cnblogs.com/oloroso/p/10795485.html)

**原文地址：**

[Qt Multimedia Backends](https://wiki.qt.io/Qt_Multimedia_Backends)

[Qt 5.11 Multimedia Backends](https://wiki.qt.io/Qt_5.11_Multimedia_Backends)

----

对于大多数功能，Qt Multimedia建立在底层系统的多媒体框架之上。因此，有基于不同技术和API的多个多媒体后端。平台特定的库和Qt Multimedia之间使用插件进行结合。
Qt Multimedia目前有三种插件：

-   **MediaService**（媒体服务）插件，提供媒体播放器，摄像头，收音机和录音功能。
-   **Audio**（音频）插件，提供低延迟（low-latency）音频支持。
-   **PlaylistFormat**（播放列表格式）插件，支持特定的播放列表文件格式。

插件不一定实现所有可能的功能, 不同的后端具有不同的功能。下表概述了 Qt 5.11 中每个后端所支持的内容。

## MediaService plugins 媒体服务插件

### 不同后端支持的媒体播放器功能:

|    | DirectShow (Windows) | Media Foundation (Windows) | AV Foundation (OSX/ iOS) | GStreamer (Unix) | Android | BlackBerry | WinRT |
| ----- | ---- | ------ | ------ | ---- | ------- | ------ | ----- |
| 媒体播放控制（MediaPlayer control）|√|√|√|√|√|√|√|
| URL 媒体源 (本地和远程)|√|√|√|√|√|√|√|
| 流媒体源（Stream source） |√|√|    |√|    |    |√|
| 媒体元信息（Metadata） |√|√| 部分    |√|√|√|    |
| 播放速率（Playback rate）|√|√|√|√|    |√|√|
| 轨道选择（Track selection）|    |    |    |√|    |    |    |
| 硬件解码（HW decoding）|√|√|√|    |√|√|√|
| 视频窗口(输出)控制（Video window control）|√|√|√|√|    |√|    |
| 视频部件(输出)控制（Video widget control）|    |    |√|√|    |    |    |
| 视频渲染控制（Video renderer control）(包括OpenGL纹理) |√|√|√|√|√|√|√|
| 音频Audio probe    |√|√|    |√|    |    |    |
| 视频探针（Video Probe） |√|√|    |√|    |    |    |

### 后端支持的摄像头（相机）功能

|    | DirectShow (Windows)    | Media Foundation (Windows) | AV Foundation (OSX/ iOS)   | GStreamer (Unix)   | Android    | BlackBerry    | WinRT    |
| ----- | ---- | ------ | ---- | ---- | ------- | ------- | ------ |
| s摄像头控制（Camera control） |√|    |√|√|√|√|√|
| 视频窗口(输出)控制（Video window control） |    |    |    |√|    |    |    |
| 视频部件(输出)控制（Video widget control） |√|    |    |√|    |    |    |
| 视频渲染控制（Video renderer control）(包括OpenGL纹理) |√|    |√|√|√|√|√|
| 音频探针（Audio probe） |    |    |    |    |    |    |    |
| 视频探针（Video probe） |√|    |√|    |√|    |√|
| 视口查找设置（ViewFinder settings） |√|    |√|√|√|√|    |
| 影像捕获（Image capture） |√|    |√|√|√|√|√|
| 捕获目标（Capture destination） | 文件, 内存缓存区    |    | 文件    | 文件, 内存缓存区    | 文件, 内存缓存区    | 文件, 内存缓存区    | 文件    |
| 影像设置（Image settings） |    |    | 分辨率 | 分辨率 | 分辨率, 质量 | 分辨率, 质量 | 分辨率 |
| 缩放（Zoom） |√(depends on HW)    |    |√(only iOS >= 7.0)    |√|√|√|    |
| 动画（Flash） |    |    |√|√(取决于硬件平台, 在桌面 Linux 上不可用)   |√|√|    |
| 聚焦（Focus） |    |    | 模式、自定义点（mode, custom point） | 模式、自定义点、焦点区域（mode, custom point, focus zones） (取决于硬件平台, 在桌面 Linux 上不可用) | 模式、自定义点、焦点区域（mode, custom point, focus zones） | 模式、自定义点、焦点区域（mode, custom point, focus zones） | 模式、自定义点（mode, custom point） |
| 曝光（Exposure） | 光圈, 快门速度（Aperture, ShutterSpeed） (依赖硬件) |    | 仅iOS >= 8.0： ISO, 快门速度，补偿（ShutterSpeed, compensation） | Scene mode, compensation, ISO, aperture, ShutterSpeed (取决于硬件平台, 在桌面 Linux 上不可用) | 场景模式, 补偿（Scene mode, compensation） | 场景模式（Scene mode） |    |
| 影像处理（Image Processing） | 手动白平衡, 对比度, 亮度, 饱和度, 锐化（Manual White Balance, Contrast, Brightness, Saturation, Sharpening） |    |    | 白平衡, 对比度, 亮度, 饱和度（White Balance, Contrast, Brightness, Saturation） | 白平衡（White Balance） | 白平衡（White Balance） |    |
| 锁定（Locks） |    |    |    | 聚焦、曝光、白平衡（Focus, Exposure, White Balance） (取决于硬件平台, 在桌面 Linux 上不可用) | 聚焦、曝光、白平衡（Focus, Exposure, White Balance） | 聚焦、曝光、白平衡（Focus, Exposure, White Balance） | 聚焦（Focus） |

### 后端支持的音频解码功能

|    | DirectShow (Windows) | Media Foundation (Windows) | AV Foundation (OSX/ iOS) | GStreamer (Unix) | Android | BlackBerry | WinRT |
| ---- | ---- | ------ | ------ | ---- | ------- | ------ | ----- |
| 解码音频（Decode audio） |    |√|    |√|    |    |    |

## Audio plugins 音频插件

音频后端实现`QAudioInput`，`QAudioOutput`，`QAudioDeviceInfo`和`QSoundEffect`。

以下是当前音频后端的列表：  

- Windows Multimedia
- WASAPI (WinRT)
- CoreAudio (OSX / iOS)
- PulseAudio (Unix)
- Alsa (Unix)
- OpenSL ES (Android)
- QNX