---
layout:  post
title:  "Qt Installer Framework 使用说明（一）"
date:  2017-04-27
categories:  其它
tags:  其它
comments: 1
---
[TOC]
[博客园原文地址 http://www.cnblogs.com/oloroso/archive/2017/04/27/6775220.html](http://www.cnblogs.com/oloroso/archive/2017/04/27/6775220.html)



# Qt Installer Framework 使用说明

这份教程是去年翻译的，有很多不准确或不正确的地方，请别介意。

原文链接 [http://doc.qt.io/qtinstallerframework/index.html](http://doc.qt.io/
qtinstallerframework/index.html)


以下offline翻译为离线，online翻译为联机。widgets翻译为小部件。


版本 2.0.3

The Qt Installer Framework提供了一套工具和实用程序用于创建桌面Qt安装程序。支持的平台有: Linux, Microsoft 
Windows, 和 OS X.

注意: 对于Qt Installer Framework中遇到的bug和错误可以通过[Qt 
Bugtracker(Qt错误追踪)](https://bugreports.qt.io/browse/QTIFW)进行反馈。

# 1、Qt Installer Framework概述

Qt安装程序框架提供了一组工具和实用程序，只需创建安装程序一次，无需改动源码，即可将它们部署在所有支持桌面QT的平台。 安装程序将在运行它们的平台上具有原生外观和感觉，支持：Linux，Microsoft Windows和OS X.

Qt安装程序框架工具生成安装程序，其中包含一组在安装，更新或卸载过程中指导用户的页面。 您提供可安装的内容并指定有关它的信息，例如产品和安装程序的名称以及许可协议的文本。

您可以通过向预定义的页面添加窗口小部件或添加整个页面来为用户提供其他选项来自定义安装程序。 您可以创建脚本以向安装程序添加操作。

## 选择安装包类型

你可以为最终用户提供一个`离线(offline)`或`联机(online)`安装包，或者同时支持。具体取决于您的使用情况。
![安装类型说明图](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427170357350-1265925878.png)


两个安装程序都安装了一个`维护工具(maintenance tool)`，以后可用于添加，更新和删除组件。 离线安装程序包含所有可安装的组件，并且在安装期间 **不需要网络连接** 。 联机安装程序仅安装维护工具，然后从Web服务器上的联机存储库下载和安装组件。 因此，在线安装程序二进制文件的大小较小，其下载时间比离线安装程序二进制文件的大。 如果最终用户没有安装所有可用的组件，则下载和运行联机安装程序所花费的总时间也可能短于下载和运行离线安装程序。

最终用户可以在`初次安装`后使用`维护工具`从 **服务器安装其他组件** ，以及在服务器上发布更新后立即接收内容的自动更新。 但是，只有在离线安装程序配置中指定`存储库地址`或最终用户在维护工具设置中指定存储库地址时，这才适用于离线安装。

创建离线安装程序以便用户可以直接下载安装软件包在介质上，稍后在计算机上进行安装。 例如，您还可以将安装软件包分发到CD-ROM或USB盘。
创建联机安装程序以使用户始终安装最新版本的内容二进制文件。

## 促进更新

使在线存储库可用于向安装产品的最终用户推广更新。 提供更新的最简单方法是重新创建存储库并将其上传到Web服务器。 对于大型存储库，只能更新已更改的组件。

## 提供安装内容

您可以启用其他内容提供程序将组件作为附加组件添加到安装程序。 组件提供程序必须设置包含可安装组件的存储库，并将指向存储库的URL提供给最终用户。 最终用户必须在安装程序中配置URL。 该附加组件在包管理器中可见。

# 2、入门指南

Qt安装程序框架是作为Qt项目的一部分开发的。 框架本身使用Qt。 但是，它可以用于创建安装所有类型应用程序的安装程序，包括（但不限于）使用Qt构建的应用程序。

## 支持的平台

您可以使用Qt Installer Framework为桌面Qt支持的所有平台创建安装程序。

**安装程序已在以下平台上测试：**
- Microsoft Windows XP及更高版本
- Ubuntu Linux 11.10及更高版本
- OS X 10.7及更高版本

## 从源代码构建

以下步骤描述如何自己构建Qt Installer Framework。 如果已下载框架的预构建(编译)版本，则可以跳过此步骤。
最新的2.0.3 64位linux安装包下载地址[http://download.qt.io/official_releases/qt-installer-framework/2.0.3/QtInstallerFramework-linux-x64.run](http://download.qt.io/official_releases/qt-installer-framework/2.0.3/QtInstallerFramework-linux-x64.run)

### 支持的编译器

Qt安装程序框架可以使用Microsoft Visual Studio 2013、GCC 4.7、Clang 3.1及其更高版本进行编译。

### 配置Qt

如果使用静态构建Qt来构建Qt安装程序框架，则不必提供Qt库，这使您可以将安装程序作为一个文件进行分发。

**所需的Qt版本最低为5.5。**

#### Windows上配置Qt
我们建议您在Windows配置Qt时使用以下选项：

```shell
configure -prefix %CD%\qtbase -release -static -static-runtime -target xp -accessibility -no-opengl -no-icu -no-sql-sqlite -no-qml-debug -nomake examples -nomake tests -skip qtactiveqt -skip qtenginio -skip qtlocation -skip qtmultimedia -skip qtserialport -skip qtquick1 -skip qtquickcontrols -skip qtscript -skip qtsensors -skip qtwebkit -skip qtwebsockets -skip qtxmlpatterns -skip qt3d
```

#### Linux上配置Qt
我们建议您在Linux配置Qt时使用以下选项：

```shell
configure -prefix $PWD/qtbase -release -static -accessibility -qt-zlib -qt-libpng -qt-libjpeg -qt-xcb -qt-pcre -qt-freetype -no-glib -no-cups -no-sql-sqlite -no-qml-debug -no-opengl -no-egl -no-xinput -no-xinput2 -no-sm -no-icu -nomake examples -nomake tests -skip qtactiveqt -skip qtenginio -skip qtlocation -skip qtmultimedia -skip qtserialport -skip qtquick1 -skip qtquickcontrols -skip qtscript -skip qtsensors -skip qtwebkit -skip qtwebsockets -skip qtxmlpatterns -skip qt3d
```

#### OS X上配置Qt
我们建议您在OS X使用以下配置选项：

```shell
configure -prefix $PWD/qtbase -release -static -accessibility -qt-zlib -qt-libpng -qt-libjpeg -no-cups -no-sql-sqlite -no-qml-debug -nomake examples -nomake tests -skip qtactiveqt -skip qtenginio -skip qtlocation -skip qtmultimedia -skip qtserialport -skip qtquick1 -skip qtquickcontrols -skip qtscript -skip qtsensors -skip qtwebkit -skip qtwebsockets -skip qtxmlpatterns -skip qt3d
```
### 设置Qt安装程序框架

1. 从[http://code.qt.io/cgit/installer-framework/installer-framework.git](http://code.qt.io/cgit/installer-framework/installer-framework.git)仓库`clone`Qt安装框架工具源码。
2. 通过运行一个静态编译Qt的`qmake`来生成`Makefile`，然后通过`make`或`nmake`进行构建。

**注意:** 要向Qt Installer Framework提供补丁，请遵循标准的Qt流程和指南。 欲了解更多信息，请参阅贡献的Qt。




# 3、最终用户工作流程

离线和在线安装程序的最终用户体验类似。 与您的应用程序一起，安装程序将安装一个维护工具，该工具由程序包管理器，更新程序和卸载程序组成。 最终用户可以使用维护工具添加，更新和删除组件。 维护工具连接到外部存储库以获取要添加或更新的组件。 您可以在配置文件中指定存储库，或者最终用户可以在维护工具设置中指定它。

您可以支持以下最终用户工作流：

## 初次安装

下图说明了安装应用程序的默认工作流程：
![默认工作流程](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427170438819-1739476998.png)

本节使用在OS X上运行的应用程序安装程序示例说明最终用户的默认工作流程。 安装程序在每个受支持的桌面平台上具有原生外观，因此在Linux和Windows上运行时，它们的外观和感觉不同。

示例文件存储在Qt Installer Framework存储库中的`examples\tutorial`目录中。 您可以使用`binarycreator`工具来创建应用程序安装程序。

### 启动安装程序

当最终用户启动安装程序时，将打开介绍页面：
![介绍页面](Image/QtIFW/ifw-introduction-page.png)

您可以在`config.xml`配置文件中指定安装程序的名称和要安装的产品。
当最终用户选择继续时，将打开`目标目录`选择页面。

### 选择目标目录

最终用户必须指定安装的目标目录。 您可以在`config.xml`配置文件中指定默认值。
![目标目录选择](Image/QtIFW/ifw-target-directory-page.png)

当最终用户选择继续时，将打开组件选择页面。

如果该目录已存在文件，将打开如下警告页面：
![目标目录已存在文件警告](Image/QtIFW/ifw-warning-existing-installation.png)

### 选择组件

组件选择页面列出了 **可用于安装的组件** 以及每个组件的简短描述。 最终用户 **选择要安装的组件** 。 他们可以选择全选以选择所有组件，取消选择全部可取消选择它们，或选择默认以恢复为默认选择。

![选择组件](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427170516944-2002929746.png)



将可安装的组件添加到包目录中的`data`目录。 您可以在`meta`目录中的`package.xml`文件中指定有关组件的信息。

您可以使用布尔运算符或脚本来指定是否默认选择组件。

当最终用户选择继续时，将打开许可证检查页面。

### 接受许可协议

在许可检查页面上，最终用户 **必须接受许可协议** 的条款才能继续安装。
![许可协议检查](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427170539303-38433567.png)



如果在package.xml文件中指定许可证文件并将文件复制到元目录，则会显示许可证检查页面。

### 选择Windows程序组

在Windows上，“开始”菜单目录选择页面 允许最终用户在Windows开始菜单中选择产品的 **程序组**。

![win程序组](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427170606740-1182353188.png)



您可以在`config.xml`配置文件中为程序组指定 **默认值** 。

当最终用户选择`下一步`时，将打开`准备安装`页面。

### 安装组件

在准备安装页面通知最终用户，当用户选择`安装`时，安装开始。

![准备安装](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427170623631-1863598746.png)



在安装过程中，执行安装页面显示有关安装进度的信息。 最终用户可以选择“显示详细信息”以查看更多信息。
![安装中](![](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427170646240-791801012.png)



安装完成后，将打开安装完成页面。
![安装完成](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427170709006-854285744.png)



在此页面中，您可以添加选项以在 **关闭安装程序时** 启动已安装的产品。在`config.xml`配置文件中指定要启动的程序和显示的文本。

## 添加组件

如果最终用户在`初次安装`期间未选择可用于安装的所有组件，则可以使用`软件包管理器(Package Manager)`稍后从存储库安装其余组件。 软件包管理器是在初次安装期间与应用程序一起安装的维护工具的一部分。 仅当包含组件的存储库可在本地或外部可用时，此操作才有效。

下图说明了安装附加组件的默认工作流程：
![附加工作流](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427170732162-350129818.png)



本节使用由在OS X上运行的Qt 5安装程序安装的`维护工具`作为最终用户在初次安装后`如何添加组件`实现的示例。 维护工具包含`程序包管理器`，`更新程序`和`卸载程序`。

### 启动程序包管理器

当最终用户启动维护工具时，将打开介绍页面：
![添加组件介绍](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427170751506-1961719280.png)


当最终用户选择`包管理器(Package Manager)`，然后单击`继续`时，将打开组件选择页面。

### 选择其他组件

组件选择页面列出了可用于安装的组件以及每个组件的简短描述。 已安装的组件在列表中显示为已选择。 最终用户选择要安装的其他组件。 他们可以选择重置以再次显示当前安装的组件。
![添加组件选择](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427170808100-262976424.png)



当最终用户选择继续时，将打开准备更新页面。

### 安装所选组件

准备更新页面通知最终用户，在用户选择 `更新(Update)` 时安装组件。

![准备更新](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427170841537-2114661372.png)

更新页面显示有关安装进度的信息。 最终用户可以选择“显示详细信息”以查看更多信息。

![更新中](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427170955522-1073513701.png)



安装完成后，将打开更新完成页面。
![更新完成](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427171031475-1858291990.png)

## 移除组件
下图说明了删除全部或部分已安装组件的默认工作流程：
![用户删除组件流程](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427171055475-1161344478.png)



本节使用在OS X上运行的Qt 5维护工具，作为最终用户如何删除选定或所有组件实现的示例。

### 删除所有组件

当最终用户启动维护工具时，将打开介绍页面：
![添加组件介绍](![](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427171123787-405556435.png)

最终用户可以选择`删除所有组件(Remove all components)`，然后单击`继续`以删除所有已安装的组件。

准备卸载页面通知最终用户，当用户选择卸载时，可以开始 **卸载** 。

![准备卸载](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427171143944-83716539.png)


### 删除所选组件

最终用户可以选择`包管理器(Package Manager)`，然后选择 **继续** ，以在 **组件选择页面** 上选择要删除的组件：

![添加组件选择](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427171223537-1243518342.png)


当最终用户取消要删除的组件的选中状态，然后选择继续时，将打开即将更新的页面。 它通知最终用户在用户选择 `更新(Update)`时删除组件。

![准备更新](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427171240850-1534908340.png)


更新页面显示有关安装进度的信息。 最终用户可以选择“显示详细信息”以查看更多信息。

![更新中](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427171302772-1319108653.png)

安装完成后，将打开更新完成页面。
![更新完成](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427171328006-859432810.png)


## 更新组件
下图说明了用于更新已安装组件的默认工作流程：
![更新组件流程](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427171350022-396291776.png)


本节使用在OS X上运行的Qt 5维护工具，作为最终用户如何删除选定或所有组件实现的示例。

### 启动更新程序

当最终用户启动维护工具时，将打开介绍页面：

![更新组件介绍](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427171417444-462526834.png)



当最终用户选择`更新组件(Update components)`，然后单击`继续`时，将打开组件选择页面。

### 选择要更新的组件

更新选择列表显示最终用户 **可以选择** 的 **可用更新列表** 。
![可更新组件](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427171434537-485392204.png)


当用户选择`继续`，将打开准备更新页面。

### 更新所选组件

准备更新页面通知最终用户，选择`更新(Update)`进行组件更新。
![准备更新](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427171459225-306456724.png)

更新页面显示有关安装进度的信息。 最终用户可以选择“显示详细信息”以查看更多信息。

![更新中](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427171521772-1610722883.png)



安装完成后，将打开更新完成页面。
![更新完成](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427171541756-1079011290.png)




## 特定设置

设置页面允许最终用户指定代理设置或安装附加组件。 最终用户在介绍页面上选择以进行指定设置。

### 指定代理设置

默认情况下，安装程序不使用任何代理设置。 最终用户可以选择使用系统代理设置或手动指定代理设置。

![设置网络代理](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427171602303-1986716214.png)


### 安装附加组件

要安装附加组件，最终用户需选择`存储库(Repositories)`选项卡。

![设置存储库](http://images2015.cnblogs.com/blog/693958/201704/693958-20170427171620537-704251553.png)



如果Web服务器需要身份验证，则最终用户可以添加其用户名和密码。 如要显示密码，最终用户可以选择显示密码。

要将其自己的存储库添加到安装程序，最终用户可以选择添加并指定指向存储库的URL。

临时存储库只能在初次安装时使用一次。 安装后，只有默认和用户定义的存储库可用。