---
layout:  post
title:  "Qt Installer Framework 使用说明"
date:  2017-02-24
categories:  其它
tags:  其它
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2017/02/24/6437542.html](http://www.cnblogs.com/oloroso/archive/2017/02/24/6437542.html)
# Qt Installer Framework 使用说明

翻译这个有一段时间了，大概是去年10月份开始的吧。英文比较差，翻译的不好。后面还有一部分没有翻译，暂时没有太多时间去做这个，就先留着吧。

我的邮箱:solym@sohu.com

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
![安装类型说明图](Image/QtIFW/ifw-overview.png)

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
![默认工作流程](Image/QtIFW/ifw-user-flow-installing.png)

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

![选择组件](Image/QtIFW/ifw-select-components.png)

将可安装的组件添加到包目录中的`data`目录。 您可以在`meta`目录中的`package.xml`文件中指定有关组件的信息。

您可以使用布尔运算符或脚本来指定是否默认选择组件。

当最终用户选择继续时，将打开许可证检查页面。

### 接受许可协议

在许可检查页面上，最终用户 **必须接受许可协议** 的条款才能继续安装。
![许可协议检查](Image/QtIFW/ifw-license-check-page.png)

如果在package.xml文件中指定许可证文件并将文件复制到元目录，则会显示许可证检查页面。

### 选择Windows程序组

在Windows上，“开始”菜单目录选择页面 允许最终用户在Windows开始菜单中选择产品的 **程序组**。

![win程序组](Image/QtIFW/ifw-win-program-group.png)

您可以在`config.xml`配置文件中为程序组指定 **默认值** 。

当最终用户选择`下一步`时，将打开`准备安装`页面。

### 安装组件

在准备安装页面通知最终用户，当用户选择`安装`时，安装开始。

![准备安装](Image/QtIFW/ifw-ready-for-installation.png)

在安装过程中，执行安装页面显示有关安装进度的信息。 最终用户可以选择“显示详细信息”以查看更多信息。
![安装中](Image/QtIFW/ifw-perform-installation.png)

安装完成后，将打开安装完成页面。
![安装完成](Image/QtIFW/ifw-installation-finished.png)

在此页面中，您可以添加选项以在 **关闭安装程序时** 启动已安装的产品。在`config.xml`配置文件中指定要启动的程序和显示的文本。

## 添加组件

如果最终用户在`初次安装`期间未选择可用于安装的所有组件，则可以使用`软件包管理器(Package Manager)`稍后从存储库安装其余组件。 软件包管理器是在初次安装期间与应用程序一起安装的维护工具的一部分。 仅当包含组件的存储库可在本地或外部可用时，此操作才有效。

下图说明了安装附加组件的默认工作流程：
![附加工作流](Image/QtIFW/ifw-user-flow-adding.png)

本节使用由在OS X上运行的Qt 5安装程序安装的`维护工具`作为最终用户在初次安装后`如何添加组件`实现的示例。 维护工具包含`程序包管理器`，`更新程序`和`卸载程序`。

### 启动程序包管理器

当最终用户启动维护工具时，将打开介绍页面：
![添加组件介绍](Image/QtIFW/ifw-add-components-introduction.png)

当最终用户选择`包管理器(Package Manager)`，然后单击`继续`时，将打开组件选择页面。

### 选择其他组件

组件选择页面列出了可用于安装的组件以及每个组件的简短描述。 已安装的组件在列表中显示为已选择。 最终用户选择要安装的其他组件。 他们可以选择重置以再次显示当前安装的组件。
![添加组件选择](Image/QtIFW/ifw-add-components-selection.png)

当最终用户选择继续时，将打开准备更新页面。

### 安装所选组件

准备更新页面通知最终用户，在用户选择 `更新(Update)` 时安装组件。

![准备更新](Image/QtIFW/ifw-ready-to-update.png)

更新页面显示有关安装进度的信息。 最终用户可以选择“显示详细信息”以查看更多信息。

![更新中](Image/QtIFW/ifw-perform-update.png)

安装完成后，将打开更新完成页面。
![更新完成](Image/QtIFW/ifw-update-finished.png)

## 移除组件
下图说明了删除全部或部分已安装组件的默认工作流程：
![用户删除组件流程](Image/QtIFW/ifw-user-flow-removing.png)

本节使用在OS X上运行的Qt 5维护工具，作为最终用户如何删除选定或所有组件实现的示例。

### 删除所有组件

当最终用户启动维护工具时，将打开介绍页面：
![添加组件介绍](Image/QtIFW/ifw-add-components-introduction.png)

最终用户可以选择`删除所有组件(Remove all components)`，然后单击`继续`以删除所有已安装的组件。

准备卸载页面通知最终用户，当用户选择卸载时，可以开始 **卸载** 。

![准备卸载](Image/QtIFW/ifw-ready-to-uninstall.png)

### 删除所选组件

最终用户可以选择`包管理器(Package Manager)`，然后选择 **继续** ，以在 **组件选择页面** 上选择要删除的组件：

![添加组件选择](Image/QtIFW/ifw-add-components-selection.png)

当最终用户取消要删除的组件的选中状态，然后选择继续时，将打开即将更新的页面。 它通知最终用户在用户选择 `更新(Update)`时删除组件。

![准备更新](Image/QtIFW/ifw-ready-to-update.png)

更新页面显示有关安装进度的信息。 最终用户可以选择“显示详细信息”以查看更多信息。

![更新中](Image/QtIFW/ifw-perform-update.png)

安装完成后，将打开更新完成页面。
![更新完成](Image/QtIFW/ifw-update-finished.png)


## 更新组件
下图说明了用于更新已安装组件的默认工作流程：
![更新组件流程](Image/QtIFW/ifw-user-flow-updating.png)

本节使用在OS X上运行的Qt 5维护工具，作为最终用户如何删除选定或所有组件实现的示例。

### 启动更新程序

当最终用户启动维护工具时，将打开介绍页面：

![更新组件介绍](Image/QtIFW/ifw-updating-introduction.png)

当最终用户选择`更新组件(Update components)`，然后单击`继续`时，将打开组件选择页面。

### 选择要更新的组件

更新选择列表显示最终用户 **可以选择** 的 **可用更新列表** 。
![可更新组件](Image/QtIFW/ifw-updating-components.png)

当用户选择`继续`，将打开准备更新页面。

### 更新所选组件

准备更新页面通知最终用户，选择`更新(Update)`进行组件更新。
![准备更新](Image/QtIFW/ifw-ready-to-update.png)

更新页面显示有关安装进度的信息。 最终用户可以选择“显示详细信息”以查看更多信息。

![更新中](Image/QtIFW/ifw-perform-update.png)

安装完成后，将打开更新完成页面。
![更新完成](Image/QtIFW/ifw-update-finished.png)


## 特定设置

设置页面允许最终用户指定代理设置或安装附加组件。 最终用户在介绍页面上选择以进行指定设置。

### 指定代理设置

默认情况下，安装程序不使用任何代理设置。 最终用户可以选择使用系统代理设置或手动指定代理设置。

![设置网络代理](Image/QtIFW/ifw-settings-network.png)

### 安装附加组件

要安装附加组件，最终用户需选择`存储库(Repositories)`选项卡。

![设置存储库](Image/QtIFW/ifw-settings-repositories.png)

如果Web服务器需要身份验证，则最终用户可以添加其用户名和密码。 如要显示密码，最终用户可以选择显示密码。

要将其自己的存储库添加到安装程序，最终用户可以选择添加并指定指向存储库的URL。

临时存储库只能在初次安装时使用一次。 安装后，只有默认和用户定义的存储库可用。




# <span id="4">4、教程: 创建一个安装程序</span>
本教程介绍如何为小项目创建简单的安装程序：

![安装起始页面](Image/QtIFW/ifw-introduction-page.png)

本节介绍以下创建安装程序 **必须** 完成的任务：

1. 创建一个包(package)目录，其中将包含所有配置文件和可安装的包。
2. 创建包含有关如何构建安装程序二进制文件和联机存储库的信息的配置文件。
3. 创建包信息文件，其中包含有关可安装组件的信息。
4. 创建安装程序内容并将其复制到软件包(package)目录中。
5. 使用`binarycreator`工具创建安装程序。<br>安装程序页面是使用您在配置和程序包信息文件中提供的信息创建的。

示例文件位于Qt Installer Framework资源库中的`examples\tutorial`目录中。

## 创建软件包目录

创建一个反映安装程序设计的目录结构，并允许将来扩展安装程序。 该目录必须包含`config`和`packages`的子目录。

![包目录结构](Image/QtIFW/ifw-tutorial-files-package-subdir.png)

有关软件包目录的详细信息，请参阅[Package Directory](http://doc.qt.io/qtinstallerframework/ifw-component-description.html)。

## 创建配置文件

在`config`目录中，创建一个名为`config.xml`的文件，其中包含以下内容：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Installer>
    <Name>Your application</Name>
    <Version>1.0.0</Version>
    <Title>Your application Installer</Title>
    <Publisher>Your vendor</Publisher>
    <StartMenuDir>Super App</StartMenuDir>
    <TargetDir>@HomeDir@/InstallationDirectory</TargetDir>
</Installer>
```
该配置文件指定介绍页面上显示以下信息：

> <Title>元素指定显示在标题栏 *(下图[1])* 上的安装程序名称。
> <Name>元素指定添加到页面名称和简介文本 *(下图[2])* 的应用程序名称。

![介绍页面](Image/QtIFW/ifw-tutorial-introduction-page.png)

其他元素用于自定义安装程序的行为：

> <Version>元素指定应用程序版本号。
> <Publisher>元素指定软件的发布者（例如，在Windows控制面板中所示）。
> <StartMenuDir>元素指定Windows开始菜单中产品的默认程序组的名称。
> <TargetDir>元素指定显示给用户的默认目标目录是当前用户的主目录中的InstallationDirectory（因为预定义变量@HomeDir@用作值的一部分）。 有关详细信息，请参阅预定义变量。

有关配置文件格式和可用元素的详细信息，请参阅[Configuration File](http://doc.qt.io/qtinstallerframework/ifw-globalconfig.html)。

## 创建程序包信息文件

在这种简单的情况下，安装程序只处理一个名为`com.vendor.product`的组件。 要向安装程序提供有关组件的信息，请创建一个名为`package.xml`的文件，其中包含以下内容，并将其放在`meta`目录中：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Package>
    <DisplayName>The root component</DisplayName>
    <Description>Install this example.</Description>
    <Version>0.1.0-1</Version>
    <ReleaseDate>2010-09-21</ReleaseDate>
    <Licenses>
        <License name="Beer Public License Agreement" file="license.txt" />
    </Licenses>
    <Default>script</Default>
    <Script>installscript.qs</Script>
    <UserInterfaces>
        <UserInterface>page.ui</UserInterface>
    </UserInterfaces>
</Package>
```
下面更详细地描述示例文件中的元素。
有关软件包信息文件的详细信息，请参阅[Package Information File Syntax](http://doc.qt.io/qtinstallerframework/ifw-component-description.html#package-information-file-syntax)

## 指定组件信息

来自以下元素的信息显示在组件选择页面上：

> <DisplayName>元素指定组件列表中组件的名称 *(下图[1])* 。
> <Description>元素指定在选择组件时显示的文本 *(下图[2])* 。

![组件选择](Image/QtIFW/ifw-tutorial-select-components.png)

## 指定安装程序版本

`<Versionss>`元素使您能够在用户有可用更新时提醒用户进行更新。

## 添加许可证

`<License>`元素指定包含许可协议文本 *(下图[1])* 的文件名称，该文本显示在许可检查页上：

![许可证检查](Image/QtIFW/ifw-tutorial-license-check.png)

## 选择默认内容

`<Default>`元素指定默认情况下是否选择组件。值为`true`的组件将设置为选定。 在此示例中，我们使用脚本在运行时解析值。 JavaScript脚本文件`installscript.qs`的名称在`<Script>`元素中指定。

## 创建安装程序内容

要安装的内容存储在组件的`data`目录中。 因为只有一个组件，请将数据放在`packages/com.vendor.product/data`目录中。 该示例已包含用于测试目的的文件，但您基本上可以将任何文件放在目录中。

有关打包规则和选项的详细信息，请参阅数据目录。

## 创建安装程序二进制文件

您现在可以创建您的第一个安装程序。 在命令行上切换到`examples\tutorial`目录。 要创建一个名为`YourInstaller.exe`一个安装程序，其中包含`com.vendor.product`标识的包，输入以下命令：

**在Windows上:**

```shell
..\..\bin\binarycreator.exe -c config\config.xml -p packages YourInstaller.exe
```

**在Linux 或 OS 上:**

```shell
../../bin/binarycreator -c config/config.xml -p packages YourInstaller
```

安装程序会在当前目录中创建，你可以提供给最终用户。

有关使用binarycreator工具的更多信息，请参阅[binarycreator](http://doc.qt.io/qtinstallerframework/ifw-tools.html#binarycreator)。

注意：如果在运行教程安装程序时显示错误消息，请检查是否使用静态构建的Qt创建安装程序。 有关更多信息，请参阅[Configuring Qt](http://doc.qt.io/qtinstallerframework/ifw-getting-started.html#configuring-qt)。


# 5、创建安装程序 Creating Installers

创建离线和联机安装程序需要以下步骤：

1. 为可安装组件创建包目录。相关相信信息，请参阅[Package Directory](http://doc.qt.io/qtinstallerframework/ifw-component-description.html)
2. 在config目录中创建一个名为`config.xml`的配置文件。 它包含有关如何构建安装程序二进制文件和联机存储库的信息。 有关文件格式和可用设置的详细信息，请参阅[Configuration File](http://doc.qt.io/qtinstallerframework/ifw-globalconfig.html)
3. 在`config\meta`目录中创建一个名为`package.xml`的包信息文件。 它包含部署和安装过程的设置。 有关详细信息，请参阅[Meta Directory](http://doc.qt.io/qtinstallerframework/ifw-component-description.html#meta-directory)
4. 创建安装程序内容并将其复制到软件包目录中。 有关详细信息，请参阅[Data Directory](http://doc.qt.io/qtinstallerframework/ifw-component-description.html#data-directory)
5. 对于联机安装程序，请使用`repogen`工具创建包含可安装内容的存储库，并将存储库上载到Web服务器。
6. 使用binarycreator工具创建安装程序。 有关详细信息，请参阅[Tools](http://doc.qt.io/qtinstallerframework/ifw-tools.html)

有关如何创建使用预定义安装程序页面的简单安装程序的示例，请参阅[Tutorial: Creating an Installer](#4)

以下部分介绍如何创建不同类型的安装程序：

## 创建离线安装程序 Creating Offline Installers

在安装期间， **离线安装程序** 根本不尝试连接到联机存储库。 但是，元数据配置（config.xml）中可以设置允许用户在线添加和更新组件。

离线安装程序在公司防火墙不允许最终用户连接到Web服务器的情况下尤其有用。 网络管理员可以在网络中设置本地更新服务。

要创建离线安装程序，请使用`binarycreator`工具的`--offline-only`选项。

要在Windows中创建离线安装程序，请输入以下命令：

```shell
<location-of-ifw>\binarycreator.exe --offline-only -t <location-of-ifw>\installerbase.exe -p <package_directory> -c <config_directory>\<config_file> <installer_name>
```
译注:`<location-of-ifw>`表示`binarycreator`程序所在目录。

某些选项具有默认值，所以，可以忽略它们。例如，输入以下命令来创建名为SDKInstaller.exe的安装程序二进制文件，其中包含由org.qt-project.sdk标识的软件包及其依赖关系：

```shell
binarycreator.exe --offline-only -c installer-config -p installer-packes SDKInstaller.exe
```

## 创建在线安装程序 Creating Online Installers

联机安装程序获取一个不在二进制文件中的存储库描述`(Update.xml)`。创建存储库并将其上传到Web服务器。 然后在用于创建安装程序的config.xml文件中指定存储库的位置。

### <span id="5_2_1">创建存储库 Creating Repositories</span>

使用`repogen`工具创建一个软件包目录的所有软件包的联机存储库：

```shell
repogen.exe -p <package_directory> <repository_directory>
```
例如，要创建只包含`org.qt-project.sdk.qt`和`org.qt-project.sdk.qtcreator`的存储库，请输入以下命令：
`
```shell
repogen.exe -p packages -i org.qt-project.sdk.qt,org.qt-project.sdk.qtcreator repository
```
创建存储库后，将其上传到Web服务器。 您必须在安装程序配置文件(config.xml)中指定存储库的位置。

### <span id="5_2_2">配置存储库 Configuring Repositories</span>

安装程序配置文件`(config.xml)`中的`<RemoteRepositories>`元素可以包含多个存储库列表。 每个存储库都可以有以下设置项：

- <Url>, 指向可用组件列表。
- <Enabled>, 0表示禁用此存储库。
- <Username>, 用于受保护存储库验证的用户名。
- <Password>, 用于受保护存储库验证的密码。
- <DisplayName>, 可选，设置要显示的字符串，而不是URL。

URL需要指向列出可用组件的Updates.xml文件。 例如：

```xml
<RemoteRepositories>
     <Repository>
             <Url>http://www.example.com/packages</Url>
             <Enabled>1</Enabled>
             <Username>user</Username>
             <Password>password</Password>
             <DisplayName>Example repository</DisplayName>
     </Repository>
</RemoteRepositories>
```
仅当安装程序 **可以访问存储库** 时，安装程序才会继续。 如果在安装后访问存储库，维护工具将拒绝安装。 但是，卸载后可再次安装。 默认情况下可以启用或禁用存储库。 对于需要验证的存储库，也可以在此处设置详细信息，但在此输入密码通常不可取，因为它以纯文本保存。 若此处未设置身份验证详细信息，将在 **运行时通过对话框获取** 。 用户可以在运行时解决这些设置。


### 创建安装程序二进制文件 Creating Installer Binaries

要使用binarycreator工具创建联机安装程序，请输入以下命令：

```shell
<location-of-ifw>\binarycreator.exe -t <location-of-ifw>\installerbase.exe -p <package_directory> -c <config_directory>\<config_file> -e <packages> <installer_name>
```

例如，输入以下命令以创建名为`SDKInstaller.exe`的安装程序二进制文件，其中 **不包含** `org.qt-project.sdk.qt`和`org.qt-project.qtcreator`的数据，因为这些软件包是 **从远程存储库下载** 的：

```shell
binarycreator.exe -p installer-packages -c installer-config\config.xml -e org.qt-project.sdk.qt,org.qt-project.qtcreator SDKInstaller.exe
```

### 减少安装程序文件大小 Reducing Installer Size

即使组件从Web服务器获取，`binarycreator` **仍然会将它们添加到安装程序二进制文件** 中。 但是，当安装程序检查Web服务器上的更新时，如果新版本不可用，则最终用户可以免于下载。

或者，您可以创建不包含任何数据仅从Web服务器获取所有数据的联机安装程序。 使用`binarycreator`工具的`-n`参数，并且只将`根(root)组件`添加到安装程序。 通常根(root)组件是空的，因此只添加根(root)的XML描述。

更多相关选项的详细信息，请参阅[Summary of binarycreator Parameters](http://doc.qt.io/qtinstallerframework/ifw-tools.html#summary-of-binarycreator-parameters)

## 促进更新 Promoting Updates

促进更新这是一个很别扭的翻译，这里的意思是在运行联机安装程序的时候，会去读取存储库中的`Update.xml`文件来判断是否需要下载更新。

创建在联机程序，以便能够向最终安装产品用户促进更新。

需要以下步骤来促进更新：

1. 复制要更新的内容到软件包目录。
2. 增加`package.xml`文件中已更新组件的`<Version>`元素的值。
3. 使用`repogen`工具重新创建具有更新内容的联机存储库，并在存储库的根目录中生成`Updates.xml`文件。
4. 将存储库上传到Web服务器。
5. 使用`binarycreator`工具创建安装程序。


### 配置更新 Configuring Updates

安装程序在启动时下载`Updates.xml`文件，并将安装的版本与文件中的版本进行比较。 如果联机版本号大于本地版本号，安装程序将在可用更新列表中显示它。

增加`package.xml`文件中组件的`<Version>`元素的值。


### 重新创建存储库 Recreating Repositories

提供更新的最简单方法是重新创建资源库，并上传到Web服务器。 有关更多信息，请参阅[创建存储库 Creating Repositories](#5_2_1)


### 部分更新存储库

整个库完全更新可能不是最优的，如果：
- 存储库非常大，因为上传需要很长时间。
- 你想只提供更新的组件。

**注意：** `repogen`会在每次调用时重新创建7zip存档。 由于7zip存储所包含的文件的时间戳（在此过程中被移动或复制时改变），每个归档的SHA值将改变。 SHA校验和值用于验证下载的存档，因此需要匹配7zip的SHA值。 由于SHA值存储在Updates.xml文件中，因此将强制您上传完整存储库。 这可以通过使用`repogen`的`--update`选项来规避。

### 创建部分更新 Creating Partial Updates

在重新创建联机存储库时，请使用`--update`参数。 它将现有存储库作为输入，并仅更改附加参数指定的组件。 那些SHA值在全局配置中也被改变。


### 上传部分更新 Uploading Partial Updates

上传下列项目到Web服务器：

- 组件目录（通常类似于com.vendor.product.updatedpart）。
- 全局Updates.xml存储在联机存储库的根目录中。

**注意：** 上传项目的顺序非常重要。 如果在实时服务器上更新存储库，请首先更新组件，然后更新`Updates.xml`。 软件包名称包括版本号，因此，在新软件包完全上传前，最终用户会收到旧软件包。

### 更改存储库 Changing Repositories

要使当前更新存储库指向其他存储库，请编辑当前存储库中的`Updates.xml`文件。 您可以`添加`，`替换`或`删除`存储库。

```xml
<RepositoryUpdate>
  <Repository action="..." OPTIONS />
  <Repository action="..." OPTIONS />
</RepositoryUpdate>
```

#### 添加存储库 Adding Repositories

要更新(原文是Update，猜测应该是添加)存储库，请将 **具有以下选项** `<Repository>`子元素添加到`<RepositoryUpdate>`元素中：

```xml
<Repository action="add" url="http://www.example.com/repository" name="user" password="password"
             displayname="Example Repository" />
```

#### 删除存储库 Removing Repositories

要删除存储库，请将 **具有以下选项** `<Repository>`子元素添加到`<RepositoryUpdate>`元素中：

```xml
<Repository action="remove" url="http://www.example.com/repository" />
```

#### 替换存储库 Replacing Repositories

要用一个存储库替换另一个，请将 **具有以下选项** `<Repository>`子元素添加到`<RepositoryUpdate>`元素中：

```xml
<Repository action="replace" oldurl="http://www.example.com/repository"
            newurl="http://www.example.com/newrepository" name="user" password="password"
            displayname="New Example Repository" />
```


## 自定义安装程序 Customizing Installers

您可以使用脚本通过以下方式自定义安装程序：

- 添加由脚本编译的Qt Installer Framework 操作，并由安装程序来执行。
- 在`package.xml`文件中指定添加的新页面，并存放到`package`目录中。
- 通过修改现有页面，将自定义的用户界面元素作为单个widgets小部件插入其中。
- 添加语言(variants)

您可以使用`组件脚本(component scripts)`和`控制脚本(control script)`来自定义安装程序。组件脚本在组件的`package.xml`文件的`<Script>`元素中指定，与特定组件相关联。在 **获取组件的元数据时加载脚本** 。有关组件脚本的更多信息，请参阅[组件脚本Controller Scripting](http://doc.qt.io/qtinstallerframework/noninteractive.html)。

有关可在组件和控制脚本中使用的全局JavaScript对象的更多信息，请参阅[Scripting API](http://doc.qt.io/qtinstallerframework/scripting-qmlmodule.html)。

### <span id="5_4_1">添加操作 Adding Operations</span>

您可以使用组件脚本在安装过程中执行Qt Installer Framework操作。 通常，通过`移动`，`复制`或`修补`来操作文件。 使用[`QInstaller::Component::addOperation`](http://doc.qt.io/qtinstallerframework/qinstaller-component.html#addOperation)或[`QInstaller::Component::addElevatedOperation`](http://doc.qt.io/qtinstallerframework/qinstaller-component.html#addElevatedOperation)函数添加操作。 有关更多信息，请参阅[向组件添加操作 Adding Operations to Components](http://doc.qt.io/qtinstallerframework/scripting.html#adding-operations-to-components)。

此外，您可以通过派生[`KDUpdater::UpdateOperation`](http://doc.qt.io/qtinstallerframework/kdupdater-updateoperation.html)来实现在安装程序中注册自定义安装操作的方法。 有关详细信息，请参阅[注册自定义操作 Registering Custom Operations](http://doc.qt.io/qtinstallerframework/scripting.html#registering-custom-operations)。

有关可用操作的信息，请参阅[操作Operations](http://doc.qt.io/qtinstallerframework/operations.html)。


### <span id="5_4_2">添加页面 Adding Pages</span>

组件可以包含一个或多个用户界面文件，这些文件由组件或控制脚本放置到安装程序中(在安装程序执行时加载)。 安装程序将自动加载`package.xml`文件的`UserInterfaces`元素中列出的所有用户界面文件。

#### 使用组件脚本添加页面 Using Component Scripts to Add Pages

要向安装程序添加新页面，请使用[`installer::addWizardPage()`](http://doc.qt.io/qtinstallerframework/scripting-installer.html#addWizardPage-method)方法并指定新页面的位置。 例如，以下代码在 **准备安装页面** 之前添加了一个`MyPage`的实例：

```js
installer.addWizardPage( component, "MyPage", QInstaller.ReadyForInstallation );
```
您可以使用组件脚本通过调用[`component::userInterface()`](http://doc.qt.io/qtinstallerframework/scripting-component.html#userInterface-method)方法访问加载的小部件的类名，如下面的代码段所示：

```js
component.userInterface( "MyPage" ).checkbox.checked = true;
```

#### 使用控制脚本添加页面 Using Control Scripts to Add Pages

要注册自定义页面，请使用[`installer::addWizardPage()`]()方法和U并文件中设置的对象名称（例如“MyPage”）。 然后调用`Dynamic${ObjectName}Callback()`函数（例如，`DynamicMyPageCallback()`）：

```js
function Controller()
{
    installer.addWizardPage(component, "MyPage", QInstaller.TargetDirectory)
}

Controller.prototype.DynamicMyPageCallback()
{
    var page = gui.pageWidgetByObjectName("DynamicMyPage");
    page.myButton.click,
    page.myWidget.subWidget.setText("hello")
}
```
您可以通过使用在UI文件中设置的对象名称来访问窗口小部件。 例如，在上面的代码中`myButton`和`myWidget`就是控件对象名称。


### 添加小部件 Adding Widgets

您可以使用组件或控制脚本将自定义用户界面元素作为单个窗口小部件（例如复选框）插入到安装程序中。

要插入单个窗口小部件，请使用[`installer::addWizardPageItem`](http://doc.qt.io/qtinstallerframework/scripting-installer.html#addWizardPageItem-method)方法。 例如，以下代码片段从脚本中将`MyWidget`的实例添加到 **组件选择页面** ：
```js
installer.addWizardPageItem( component, "MyWidget", QInstaller.ComponentSelection );
```

### 与安装程序功能交互

例如，您可以使用控制脚本在测试中自动执行安装程序功能。 以下代码段说明如何自动单击目标目录选择页上的 **下一步(Next)** 按钮：

```js
Controller.prototype.TargetDirectoryPageCallback = function()
{
    gui.clickButton(buttons.NextButton);
}
```

### <span id="5_4_5">翻译页面 Translating Pages</span>

安装程序使用`Qt Translation`系统支持将用户可读输出转换为多种语言。 要向最终用户提供组件脚本和用户界面中包含的字符串的 **本地化版本** ，请创建随组件一起加载的`QTranslator`文件。 安装程序加载与当前系统区域设置匹配的翻译文件。 例如，如果系统语言环境是德语，则会加载de.qm文件。 此外，如果找到，将显示本地化license_de.txt而不是默认的license.txt。

需要将翻译添加到激活组件的`package.xml`文件中：

```xml
<Translations>
    <Translation>de.qm</Translation>
</Translations>
```
对脚本中的文本文本使用`qsTr()`函数。 此外，您可以将`Component.prototype.retranslateUi`方法添加到脚本。 当安装程序的语言更改并加载翻译文件时调用。

当在翻译用户界面时使用`qsTr`或`UI文件的类名`,用于翻译的上下文是脚本文件的基本名称。例如，如果脚本文件被称为`installscript.qs`，上下文将是`installscript`。

注意：翻译系统也可用于自定义UI。 使用例如一个`en.ts`文件作为自定义英语版本替换安装程序中的任何文本。



# 6、Qt Installer Framework 示例

这些示例说明如何使用组件脚本来定制安装程序。

|示例|说明|
|:---|:---|
|[更改安装程序UI示例 Change Installer UI Example](http://doc.qt.io/qtinstallerframework/qt-installer-framework-changeuserinterface-example.html)|使用组件脚本修改安装程序UI|
|[组件错误示例 Component Error Example](http://doc.qt.io/qtinstallerframework/qt-installer-framework-componenterror-example.html)|如果无法安装组件，请使用组件脚本停止安装|
|[依赖性解决示例 Dependency Solving Example](http://doc.qt.io/qtinstallerframework/qt-installer-framework-dependencies-example.html)|使用组件的package.xml文件来定义组件之间的依赖性和自动依赖性|
|[动态页面安装程序示例 Dynamic Page Installer Example](http://doc.qt.io/qtinstallerframework/qt-installer-framework-dynamicpage-example.html)|使用组件脚本和动态页来构建安装程序|
|[修改提取安装程序示例 Modify Extract Installer Example](http://doc.qt.io/qtinstallerframework/qt-installer-framework-modifyextract-example.html)|在组件脚本中使用归档提取钩子来修改目标路径|
|[在线安装程序示例 Online Installer Example](http://doc.qt.io/qtinstallerframework/qt-installer-framework-online-example.html)|使用repogen工具和配置文件设置在线安装程序|
|[打开ReadMe示例 Open ReadMe Example](http://doc.qt.io/qtinstallerframework/qt-installer-framework-openreadme-example.html)|使用组件脚本添加用于打开自述文件到最终安装程序页面的复选框|
|[退出安装程序示例 Quit Installer Example](http://doc.qt.io/qtinstallerframework/qt-installer-framework-quitinstaller-example.html)|使用组件脚本退出安装程序|
|[注册文件扩展示例 Register File Extension Example](http://doc.qt.io/qtinstallerframework/qt-installer-framework-registerfileextension-example.html)|使用组件脚本在Windows上注册文件扩展名|
|[开始菜单快捷方式示例 Start Menu Shortcut Example](http://doc.qt.io/qtinstallerframework/qt-installer-framework-startmenu-example.html)|使用组件脚本将条目添加到Windows“开始”菜单|
|[系统信息示例 System Information Example](http://doc.qt.io/qtinstallerframework/qt-installer-framework-systeminfo-example.html)|在组件脚本中使用systemInfo API来检查操作系统版本和位数|
|[翻译示例 Translation Example](http://doc.qt.io/qtinstallerframework/qt-installer-framework-translations-example.html)|使用翻译本地化安装程序页面和许可证|



# 7、参考 Reference
以下部分包含有关Qt Installer Framework的详细信息：

## 配置文件 Configuration File

配置文件定制安装程序的用户界面和行为。 该文件通常名为config.xml并位于config目录中。

最小配置文件由`<Installer>`根元素和`<Name>`和`<Version>`元素组成。 其他元素都是可选的，且无顺序要求。

以下是典型的配置文件示例：

```xml
<?xml version="1.0"?>
<Installer>
    <Name>Some Application</Name>
    <Version>1.0.0</Version>
    <Title>Some Application Setup</Title>
    <Publisher>发行公司</Publisher>
    <ProductUrl>http://www.your-fantastic-company.com</ProductUrl>
    <InstallerWindowIcon>installericon</InstallerWindowIcon>
    <InstallerApplicationIcon>installericon</InstallerApplicationIcon>
    <Logo>logo.png</Logo>
    <Watermark>watermark.png</Watermark>
    <RunProgram>@TargetDir@/YourAppToRun</RunProgram>
    <RunProgramArguments>
        <Argument>Argument 1</Argument>
        <Argument>Argument 2</Argument>
    </RunProgramArguments>
    <RunProgramDescription>程序描述文本</RunProgramDescription>
    <StartMenuDir>Some Application Entry Dir</StartMenuDir>
    <MaintenanceToolName>SDKMaintenanceTool</MaintenanceToolName>
    <AllowNonAsciiCharacters>true</AllowNonAsciiCharacters>
    <Background>background.png</Background>

    <TargetDir>@HomeDir@/testinstall</TargetDir>
    <AdminTargetDir>@RootDir@/testinstall</AdminTargetDir>
    <RemoteRepositories>
        <Repository>
            <Url>http://www.your-repo-location/packages/</Url>
        </Repository>
    </RemoteRepositories>
</Installer>
```
### 配置文件元素的简要说明 Summary of Configuration File Elements

下表总结了配置文件中的元素。

**注意：** 我们建议您将配置文件中引用的所有文件放在config目录中。 但是，您也可以使用相对路径，工具相对于config.xml文件的位置进行解析。

您可以使用预定义变量 **(嵌入@字符)** 作为元素的值。 有关详细信息，请参阅[预定义变量 Predefined Variables](http://doc.qt.io/qtinstallerframework/scripting.html#predefined-variables)。


|元素|是否必需|描述|
|:---|:---|:---|
|Name|必需|正在安装的产品名称|
|Version|必需|安装软件的版本，格式要求:`[0-9]+((\.|-)[0-9]+)*`例如:`1-1`;`1.2-2`;`3.4.7.`|
|Title|可选|标题栏上显示的安装程序的名称|
|Publisher|可选|软件发行商(如Windows控制面板中所示)|
|ProductUrl|可选|指向包含您网站上产品信息的网页的网址|
|Icon|可选|自定义安装程序图标的文件名。通过附加'.icns'(OS X)，'.ico'(Windows)或'.png'(Unix)后缀来查找实际文件。 **已弃用** ，请改用`<InstallerApplicationIcon>`或`<InstallerWindowIcon>`|
|InstallerApplicationIcon|可选|自定义安装程序图标的文件名。 通过附加'.icns'(OS X)，'.ico'(Windows)后缀来查找实际文件。 在Unix上没有此功能|
|InstallerWindowIcon|可选|安装程序应用程序的自定义窗口图标的文件名(PNG格式)|
|Logo|可选|用作`QWizard::LogoPixmap`的徽标的文件名|
|Watermark|可选|用作`QWizard::WatermarkPixmap`的水印的文件名|
|Banner|可选|用作`QWizard::BannerPixmap`(横幅)的文件名(仅由`ModernStyle`使用)|
|Background|可选|用作`QWizard::BackgroundPixmap`的图像的文件名(仅由`MacStyle`使用)|
|WizardStyle|可选|设置要使用的向导样式 ("Modern"(现代), "Mac", "Aero"或"Classic"(经典))|
|WizardDefaultWidth|可选|像素单位的默认向导(Wizard)宽度，设置的横幅图像(Banner Image)将覆盖在此|
|WizardDefaultHeight|可选|以像素为单位设置向导的默认高度,设置水印图像将覆盖在此|
|TitleColor|可选|设置标题和字幕的颜色(采用HTML颜色代码，例如“＃88FF33”)|
|RunProgram|可选|如果用户接受操作，则在安装程序完成后执行命令。要提供应用程序的完整路径|
|RunProgramArguments|可选|传递给`<RunProgram>`中指定的程序的参数。您可以添加多个`<Argument>`子元素，每个子元素为`<RunProgram>`指定一个参数|
|RunProgramDescription|可选|安装后运行程序的复选框旁边显示的文本。如果`<RunProgram>`设置，但没有提供说明，用户界面将显示运行程序`<Name>`替代|
|StartMenuDir|可选|Windows开始菜单中产品的默认程序组名称|
|TargetDir|可选|安装的默认目标目录。在Linux上，这通常是用户的主目录(/home/username/)|
|AdminTargetDir|可选|具有管理员权限的安装的默认目标目录。仅在Linux上可用，通常不建议在管理员用户的主目录中安装|
|RemoteRepositories|可选|远程存储库列表。此元素可以包含多个`<Repository>`子元素，每个子元素包含指定用于访问存储库的URL的`<Url>`子元素。有关更多信息，请参阅[配置存储库](#5_2_2)|
|MaintenanceToolName|可选|生成的维护工具的文件名。默认为`maintenancetool`。将附加平台特定的可执行文件扩展名|
|MaintenanceToolIniFile|可选|用于生成维护工具配置的文件名。默认为`MaintenanceToolName.ini`|
|RemoveTargetDir|可选|如果卸载时不删除目标目录(TargetDir)，请设置为`false`|
|AllowNonAsciiCharacters|可选|如果安装路径可以包含 **非ASCII字符** ，请设置为`true`|
|RepositorySettingsPageVisible|可选|设置为`false`来 **隐藏** 设置对话框中的 **存储库设置页面** |
|AllowSpaceInPath|可选|如果安装路径 **不能包含空格字符** ，请设置为`false`|
|DependsOnLocalInstallerBinary|可选|如果要禁止从外部资源(如网络驱动器)安装，请设置为true。 这可能有意义，例如，安装程序非常大。该选项仅在Windows上使用|
|TargetConfigurationFile|可选|目标的配置文件名。默认是`components.xml`|
|Translations|可选|用于翻译用户界面的语言代码列表。要添加多个语言变体，请指定多个`<Translation>`子元素，每个元素指定语言变体的名称。该项可选。有关详细信息，请参阅[翻译页](#5_4_5)|
|UrlQueryString|可选|此字符串必须采用`key = value`形式，并且将附加到存档下载请求。这可以用于向托管存储库的web服务器传输信息|
|ControlScript|可选|自定义安装程序控制脚本的文件名。请参阅[控制脚本](#7_3)|
|CreateLocalRepository|可选|如果要在安装目录中 **创建本地存储库** ，请设置为`true`。此选项对在线安装程序没有影响。存储库将自动添加到默认存储库列表|


## 包目录 Package Directory
安装程序包含嵌入到安装程序或从远程存储库加载的组件。在这两种情况下，你需要提供组件使用的文件格式和结构，以便安装程序读取。

### 包目录结构 Package Directory Structure

将所有组件放在同一根目录中，这称为包目录。目录名称用作 **域标识符** ，其标识所有组件。例如 `com.vendor.root`。

在根目录中，创建称`data`和`meta`子目录。

一个包目录可以如下所示：
```shell
-packages
    - com.vendor.root
        - data
        - meta
    - com.vendor.root.component1
        - data
        - meta
    - com.vendor.root.component1.subcomponent1
        - data
        - meta
    - com.vendor.root.component2
        - data
        - meta
```

### 元信息目录 Meta Directory

meta目录包含指定部署和安装过程设置的文件。安装程序不会提取文件(这些文件不会嵌入到安装程序中)。该目录必须至少包含程序 **包信息文件** 和您在程序包信息文件中所有 **引用的文件** ，例如脚本，用户界面文件和翻译。

#### 包信息文件语法 Package Information File Syntax

`package.xml`文件是关于组件信息的主要来源。 以下是package文件的示例：

```xml
<?xml version="1.0"?>
<Package>
    <DisplayName>QtGui</DisplayName>
    <Description>Qt gui libraries</Description>
    <Description xml:lang="de_de">Qt GUI Bibliotheken</Description>
    <Version>1.2.3</Version>
    <ReleaseDate>2009-04-23</ReleaseDate>
    <Name>com.vendor.root.component2</Name>
    <Dependencies>com.vendor.root.component1</Dependencies>
    <Virtual>false</Virtual>
    <Licenses>
        <License name="License Agreement" file="license.txt" />
    </Licenses>
    <Script>installscript.qs</Script>
    <UserInterfaces>
        <UserInterface>specialpage.ui</UserInterface>
        <UserInterface>errorpage.ui</UserInterface>
    </UserInterfaces>
    <Translations>
        <Translation>sv_se.qm</Translation>
        <Translation>de_de.qm</Translation>
    </Translations>
    <DownloadableArchives>component2.7z, component2a.7z</DownloadableArchives>
    <AutoDependOn>com.vendor.root.component3</AutoDependOn>
    <SortingPriority>123</SortingPriority>
    <UpdateText>This changed compared to the last release</UpdateText>
    <Default>false</Default>
    <ForcedInstallation>false</ForcedInstallation>
    <Essential>false</Essential>
    <Replaces>com.vendor.root.component2old</Replaces>
</Package>
```

#### 软件包信息文件元素简介 Summary of Package Information File Elements

|元素|是否必需|描述|
|:---|:---|:---|
|DisplayName|必需|组件的可读名称|
|Description|必需|组件的可读描述。<br>将描述的翻译指定为附加说明标记的值，并将`xml:lang`属性设置为正确的语言环境。如果找不到与语言环境匹配的翻译文件，且存在未翻译的版本，则使用未翻译版本。否则，将不显示该描述|
|Version|必需|组件的版本号格式为：`[0-9]+((\.|-)[0-9]+)* `如`1-1`; `1.2-2`; `3.4.7`。<br> 如果包需要显示来自子进程的版本号，而不是它自己的版本号（由于子包的分组），可以指定属性`inheritVersionFrom`，包含版本需要继承的包名|
|ReleaseDate|必需|此组件版本发布的日期|
|Name|必需|此组件的域标识|
|Dependencies|可选|此组件依赖的组件的标识符列表(使用逗号分割)。<br>您可以指定版本号，以破折号(`-`)分隔。您可以使用比较运算符`(=, >, <, >= or <=)`为版本号添加前缀。<br>请记住，您必须使用字符引用`＆lt;`来避开左尖括号(使用`&lt;`来插入`<`，使用`＆lt; =`来插入`<=`)。更多相关信息，请参阅[组件依赖关系](#7_2_2_3)|
|AutoDependOn|可选|此组件具有自动依赖性的组件的标识符列表(逗号分隔)。<br>当且仅当 **满足所有指定的依赖关系** 时，才安装组件。如果组件对其他组件有自动依赖性，则组件树中组件旁的复选框将隐藏，并自动进行选择。如果组件以前未安装，则只有当选择此列表中的所有组件进行安装时，才会选择安装组件。如果组件已安装，则当选择此列表中的 **至少一个组件** 进行卸载时，将选择卸载组件。有关更多信息，请参阅[组件依赖关系](#7_2_2_3)|
|Virtual|可选|设置为true可从安装程序中隐藏组件。<br>请注意，在根组件上设置此选项不起作用|
|SortingPriority|可选|组件在组件树中的优先级。 树从最高优先级到最低优先级排序，在顶部具有最高优先级|
|Licenses|可选|安装用户接受的许可协议列表。<br>要添加几个许可证，请添加几个`<License>`子元素，每个子元素指定许可证名称和文件。 如果有针对此组件列出的翻译，安装程序还将查找翻译的许可证。 它们需要具有与原始许可证文件相同的名称，但是具有添加的区域设置标识符。 例如，如果许可证文件被称为`license.txt`并且指定了德语翻译，且安装程序还包含一个`license_de_de.txt`文件（将在德语系统上安装时显示）|
|Script|可选|被加载的脚本文件名。<br>有关详细信息，请参阅[添加操作](#5_4_1)|
|UserInterfaces|可选|要加载的页面列表。 要添加多个页面，请添加多个`<UserInterface>`子元素，每个子元素指定页面的文件名。有关详细信息，请参阅[添加页面](#5_4_2)|
|Translations|可选|要加载的翻译文件的列表。<br>要添加多个语言变体，请指定多个`<Translation>`子元素，每个子元素指定语言变体的文件名。有关详细信息，请参阅[翻译页](#5_4_5)|
|UpdateText|可选|如果这是一个更新组件，则将说明添加到组件描述中|
|Default|可选|可以设置为`true`、`false`、`脚本名`。<br>true：在安装程序中预先选中组件。此操作仅适用于没有可见子组件的组件。<br>脚本名：在运行时解析执行脚本，计算出布尔值(boolean)。将脚本文件名称添加为此文件中`<Script>`元素的值。有关脚本示例，请参阅[Selecting Default Contents](http://doc.qt.io/qtinstallerframework/ifw-tutorial.html#selecting-default-contents)|
|Essential|可选|将包标记为必须去强制重启`MaintenanceTool`程序的。<br>如果有 **可用的必要组件更新** ，则程序包管理器保持禁用状态直到该组件更新。当运行`updater`时，新引入的必要组件将自动安装|
|ForcedInstallation|可选|确定必须安装的包。<br>最终用户无法在安装程序中取消选择它|
|Replaces|可选|要替换的组件列表(逗号分隔)|
|DownloadableArchives|可选|列出数据文件(逗号分隔)，供在线安装程序下载。<br>如果组件中有一些数据，并且`package.xml`和(或)脚本没有`DownloadableArchives`值，`repogen`工具将自动注册找到的数据|
|RequiresAdminRights|可选|如果程序包需要提升权限进行安装，请设置为true|

#### <span id="7_2_2_3">组件依赖关系 Component Dependencies</span>

组件可以依赖于一个或多个 **真实** 或 **虚拟** 组件。 通过使用 **组件标识符** 和可选的 **组件版本** 来定义依赖性。 使用破折号`-`将版本号与标识符分隔开。

您可以使用比较运算符 `(=, >, &lt; (<), >=  或 &lt;= (<=)) `为版本号添加前缀，以指示包的版本号与所需版本进行比较，并且必须 **等于，大于，小于，大于等于 或 小于等于** 依赖关系中指定的版本号。如果未给出比较运算符，它将默认为`=`。


### 数据目录 Data Directory

数据(data)目录包含安装程序在安装期间提取的内容。您必须将数据打包为7zip存档(`.7z`)。 您可以使用随Qt Installer Framework提供的archivegen工具或其它生成7zip存档的工具。
译注：实际上并不直接打包成7z压缩包也可以，可以使用`binarycreator`工具直接创建安装程序。


## <span id="7_3">控制脚本 Controller Scripting</span>

对于每个安装程序，您可以指定一个控制脚本与安装程序的用户界面或功能进行某些交互。控制脚本可以对向导(Wizard)添加和删除页面，更改现有页面，做额外的检查，并通过模拟用户点击UI交互。 这允许例如 **无人值守的安装** 。

脚本格式必须兼容[`QJSEngine`](http://doc.qt.io/qt-5/qjsengine.html)。

本节介绍为实现此类控制脚本而调用的函数。 它还概述了安装程序页面和每个页面上可用的小部件，例如按钮，单选按钮和行编辑。

### 编写控制脚本 Writing Control Scripts

最小有效脚本需要至少包含一个构造函数，它可能如下所示：

```js
function Controller()
{
}
```

以下示例介绍了一个更高级的脚本，它使用[`gui`](http://doc.qt.io/qtinstallerframework/scripting-gui.html) JavaScript全局对象的方法在介绍页面上设置新的页面标题和欢迎消息，并自动单击 **目标目录选择页面** 上的 **下一步(Next)** 按钮：

```js
function Controller()
{
}

// 介绍页面回调函数
Controller.prototype.IntroductionPageCallback = function()
{
    var widget = gui.currentPageWidget(); // 获取当前向导页面 wizard page
    if (widget != null) {
        widget.title = "New title."; // 设置页面标题
        widget.MessageLabel.setText("New Message."); // 设置欢迎信息 welcome text
    }
}

// 目标目录选择页面回调函数
Controller.prototype.TargetDirectoryPageCallback = function()
{
    gui.clickButton(buttons.NextButton); // 自动单击 Next 按钮(下一步)
}
```
有关可以在控制脚本中使用的JavaScript全局对象的更多信息，请参阅[Scripting API](#7_7)。

### 预定义安装页 Predefined Installer Pages

`QInstaller` JavaScript对象提供对以下预定义安装程序页面的访问：

- [Introduction](#7_3_2_1) 介绍
- [TargetDirectory](#7_3_2_3) 目标目录
- [ComponentSelection](#7_3_2_4) 组件选择
- [LicenseCheck](#7_3_2_2) 许可证检查
- [StartMenuSelection](#7_3_2_5) 开始菜单选择
- [ReadyForInstallation](#7_3_2_6) 准备安装
- [PerformInstallation](#7_3_2_7) 执行安装
- [InstallationFinished](#7_3_2_8) 安装完成

[`button`](http://doc.qt.io/qtinstallerframework/scripting-buttons.html) JavaScript对象提供了一组可在安装程序页面上使用的按钮。

以下部分描述可用于与安装程序页面和 **每个页面** 上 **可用的窗口小部件** 交互的功能。(就是每个页面上具有的控件)


#### <span id="7_3_2_1">介绍页 Introduction Page</span>

实现`Controller.prototype.IntroductionPageCallback()`函数与介绍页上的小部件交互。

向导按钮 Wizard Button:
- NextButton    下一步按钮
- CanceButton   取消按钮

|Widgets|简介|
|:---|:---|
|ErrorLabel|显示错误消息|
|MessageLabel|显示消息。默认情况下，它显示`Welcome to the <Name> Setup Wizard`消息。(可能会是翻译后的版本)|
|InformationLabel|显示进度信息|

|Radio Buttons|简介|
|:---|:---|
|PackageManagerRadioButton|维护工具运行时，显示在页面上的包管理器(Package manager)单选按钮|
|UpdaterRadioButton|维护工具运行时，页面上显示更新程序(Update components)单选按钮|
|UninstallerRadioButton|维护工具运行时，页面上显示的卸载程序(Remove all Components)单选按钮。 默认选择|

|Progress Bar|简介|
|:---|:---|
|InformationProgressBar|获取远程包时显示的进度条|

|Qt Core Feature|简介|
|:---|:---|
|packageManagerCoreTypeChanged()|如果希望在维护工具类型更改时收到通知，请连接到此信号。<br> **注意：** 该信号仅在用户已经启动二进制文件时发出(就是启动的maintenancetool)，且仅在单选按钮之间切换时|


示例代码:

```js
function Controller()
{
    var widget = gui.pageById(QInstaller.Introduction); // get the introduction wizard page
    if (widget != null)
        widget.packageManagerCoreTypeChanged.connect(onPackageManagerCoreTypeChanged);
}

onPackageManagerCoreTypeChanged = function()
{
    console.log("Is Updater: " + installer.isUpdater());
    console.log("Is Uninstaller: " + installer.isUninstaller());
    console.log("Is Package Manager: " + installer.isPackageManager());
}
```


#### <span id="7_3_2_2">许可协议页 License Agreement Page</span>
实现`Controller.prototype.LicenseAgreementPageCallback()`函数与许可协议页面上的窗口小部件交互。

向导按钮 Wizard Button:
- NextButton    下一步按钮
- CanceButton   取消按钮
- BackButton    后退按钮

|Widgets|简介|
|:---|:---|
|LicenseListWidget|列出可用的许可证|
|LicenseTextBrowser|显示所选许可证文件的内容|
|AcceptLicenseLabel|显示接受许可证单选按钮旁的文本|
|RejectLicenseLabel|显示拒绝许可证单选按钮旁的文本|

|Radio Buttons|简介|
|:---|:---|
|AcceptLicenseRadioButton|接受许可协议单选按钮|
|RejectLicenseRadioButton|拒绝许可协议单选按钮。默认选择|


#### <span id="7_3_2_1"> 目标目录选择页 Target Directory Page</span>

实现`Controller.prototype.TargetDirectoryPageCallback()`函数与目标目录选择页面上的窗口小部件交互。

向导按钮 Wizard Button:
- NextButton    下一步按钮
- CanceButton   取消按钮
- BackButton    后退按钮

|Widgets|简介|
|:---|:---|
|MessageLabel|显示消息|
|TargetDirectoryLineEdit|显示安装的目标目录路径|
|WarningLabel|显示警告|


#### <span id="7_3_2_1">组件选择页 Component Selection Page</span>

实现`Controller.prototype.ComponentS actionPage Callback()`函数与组件选择页面上的窗口小部件交互。

向导按钮 Wizard Button:
- NextButton    下一步按钮
- CanceButton   取消按钮
- BackButton    后退按钮

|Methods|简介|
|:---|:---|
|selectAll()|如果可以，选择所有可用的软件包|
|deselectAll()|如果可以，取消选择所有可用的软件包|
|selectDefault()|检查可用软件包状态，重置为其初始状态|
|selectComponent(id)|选择带有ID(string)的包|
|deselectComponent(id)|取消选择带有ID(string)的包|

|Push Buttons|简介|
|:---|:---|
|SelectAllComponentsButton|如果可以，选择所有可用的软件包|
|DeselectAllComponentsButton|如果可以，取消选择所有可用的软件包|
|SelectDefaultComponentsButton|检查可用软件包状态，重置为其初始状态|
|ResetComponentsButton|重置为已安装的组件|


#### <span id="7_3_2_1">开始菜单目录页 Start Menu Directory Page</span>

实现`Controller.prototype.StartMenuDirectoryPageCallback()`函数与准备安装页面上的小部件交互。

向导按钮 Wizard Button:
- NextButton    下一步按钮
- CanceButton   取消按钮
- BackButton    后退按钮

|Widgets|简介|
|:---|:---|
|StartMenuPathLineEdit|显示创建程序快捷方式的目录|

#### <span id="7_3_2_1">准备安装页 Ready for Installation Page</span>

实现`Controller.prototype.StartMenuDirectoryPageCallback()`函数与准备安装页面上的小部件交互。


向导按钮 Wizard Button:
- NextButton    下一步按钮
- CanceButton   取消按钮
- BackButton    后退按钮

|Widgets|简介|
|:---|:---|
|MessageLabel|显示消息|
|TaskDetailsBrowser|显示有关安装的一些更详细的信息|


#### <span id="7_3_2_1">执行安装页 Perform Installation Page</span>

实现`Controller.prototype.PerformInstallationPageCallback()`函数来与执行安装页面上的窗口小部件交互。

向导按钮 Wizard Button:
- CommitButton  提交按钮
- CancelButton  取消按钮

#### <span id="7_3_2_1">介绍页 Finished Page</span>

实现`Controller.prototype.Finished Page Callback()`函数与安装完成页面上的小部件进行交互。

向导按钮 Wizard Button:
- CommitButton  提交按钮
- CancelButton  取消按钮
- FinishButton  完成按钮

|Widgets|简介|
|:---|:---|
|MessageLabel|显示消息|
|RunItCheckBox|文本字段，通知用户他们可以在安装过程完成后启动应用程序|

#### <span id="7_3_2_1">介绍页 Custom Pages</span>

自定义页面注册为`Dynamic${ObjectName}`，其中`${ObjectName}`是在UI文件中设置的对象名。因此`Dynamic${ObjectName}Callback()`的调用，可以使用对象名(从UI文件)来寻址小部件。

示例代码：
```js
function Controller()
{
    // add page with widget \c SomePageWidget before the target directory page
    // 在目标目录页之前添加具有部件\ C SomePageWidget的页
    installer.addWizardPage(component, "SomePageWidget", QInstaller.TargetDirectory)
}

Controller.prototype.DynamicSomePageWidgetCallback()
{
    var page = gui.pageWidgetByObjectName("DynamicSomePageWidget");
    page.myButton.click, //指向子对象或UI文件中的小部件 direct child of the UI file's widget
    page.someFancyWidget.subWidget.setText("foobar") // 嵌套小部件 nested widget
}
```
#### 消息框 Message Boxes

在执行安装程序应用程序时，例如，应用程序可能会(弹出)显示一些关于发生的错误的消息框。 这在最终用户的系统上运行应用程序是很棒的，但它可能会破坏自动测试套件。为了克服这个问题，Qt安装程序框架显示的所有消息框，都可以通过特定的标识符寻址。


|Identifier标识符|可能的Answers|描述|
|:---|:---|:---|
|OverwriteTargetDirectory|Yes, No|确认将已存在的目录用作安装的目标目录|
|installationError|OK, Retry, Ignore|执行安装时出现致命错误|
|installationErrorWithRetry|Retry, Ignore, Cancel|执行安装时出错。最终用户可以选择重试来再次安装|
|AuthorizationError|Abort, OK|无法提升权限|
|OperationDoesNotExistError|Abort, Ignore|尝试执行操作时发生错误，但操作并不存在|
|isAutoDependOnError|OK|调用软件包脚本时出错。 无法评估程序包是否具有对其他程序包的自动依赖性|
|isDefaultError|OK|调用软件包脚本时出错。 无法评估默认情况下是否安装软件包|
|DownloadError|Retry, Cancel|而从远程仓库下载的存档散列(hash)时发生错误。最终用户可以选择重试来重试|
|archiveDownloadError|Retry, Cancel|从远程存储库下载归档时出错。最终用户可以选择重试来重试|
|WriteError|OK|写入维护工具(maintenance tool)时出错|
|ElevationError|OK|无法提升权限|
|unknown|OK|删除某个包时发生未知错误|
|Error|OK|一般错误|
|stopProcessesForUpdates|Retry, Ignore, Cancel|更新软件包时出错。 某些正在运行的应用程序或进程需要退出才能执行更新。 最终用户可以选择重试，来在停止这些程序后重试|
|Installer_Needs_To_Be_Local_Error|OK|安装程序二进制文件是从网络位置启动的，但不支持通过网络安装|
|TargetDirectoryInUse|No|安装的目标目录已包含安装的文件|
|WrongTargetDirectory|OK|安装的目标目录是一个文件或符号链接。|
|AlreadyRunning|OK|另一个应用程序实例已在运行|

示例代码：
```js
function Controller()
{
    installer.autoRejectMessageBoxes;
    installer.setMessageBoxAutomaticAnswer("OverwriteTargetDirectory", QMessageBox.Yes);
    installer.setMessageBoxAutomaticAnswer("stopProcessesForUpdates", QMessageBox.Ignore);
}
```

## 组件脚本 Component Scripting

对于每个组件，您可以准备一个脚本来指定由安装程序执行的操作。 脚本格式必须兼容[QJSEngine](http://doc.qt.io/qt-5/qjsengine.html)

### Construction

脚本必须包含安装程序在 **加载脚本时** 创建的`Component`对象。 因此，脚本必须至少包含`Component()`函数，该函数执行初始化，例如将页面放在正确的位置或连接信号和槽。

以下代码片段将`ErrorPage`（它是从`errorpage.ui`加载的用户界面文件的类名）放在 **准备安装页面** 的前面，并将其完整性设置为`false`。

```js
function Component()
{
    // Add a user interface file called ErrorPage, which should not be complete
    // 添加UI文件调用ErrorPage，这应该是不完整的
    installer.addWizardPage( component, "ErrorPage", QInstaller.ReadyForInstallation );
    component.userInterface( "ErrorPage" ).complete = false;
}
```
有关更多信息，请参阅[`installer::addWizardPage()`](http://doc.qt.io/qtinstallerframework/scripting-installer.html#addWizardPage-method)和[`component::userInterface()`](http://doc.qt.io/qtinstallerframework/scripting-component.html#userInterface-method)的文档

### 安装钩子 Installer Hooks

您可以在脚本中添加以下钩子方法：

|Method|描述|
|:---|:---|
|Component.prototype.retranslateUi|当安装程序的语言更改时调用|
|Component.prototype.createOperations|参见[component::createOperations()](http://doc.qt.io/qtinstallerframework/scripting-component.html#createOperations-method)|
|Component.prototype.createOperationsForArchive|参见[component::createOperationsForArchive()](http://doc.qt.io/qtinstallerframework/scripting-component.html#createOperationsForArchive-method)|
|Component.prototype.createOperationsForPath|S参见[component::createOperationsForPath()](http://doc.qt.io/qtinstallerframework/scripting-component.html#createOperationsForPath-method)|


### 全局变量 Global Variables

安装程序将以下符号放入脚本空间：

|Symbol|描述|
|installer|引用组件的[QInstaller](http://doc.qt.io/qtinstallerframework/scripting-qinstaller.html)|
|component|引用组件的[Component](http://doc.qt.io/qt-5/qml-qtqml-component.html)|

### 消息框 Message Boxes

您可以通过使用以下静态成员从脚本中显示[QMessageBox](http://doc.qt.io/qtinstallerframework/scripting-qmessagebox.html)：

- [QMessageBox::critical()](http://doc.qt.io/qtinstallerframework/scripting-qmessagebox.html#critical-method)
- [QMessageBox::information()](http://doc.qt.io/qtinstallerframework/scripting-qmessagebox.html#information-method)
- [QMessageBox::question()](http://doc.qt.io/qtinstallerframework/scripting-qmessagebox.html#question-method)
- [QMessageBox::warning()](http://doc.qt.io/qtinstallerframework/scripting-qmessagebox.html#warning-method)

为了方便起见，[`QMessageBox::StandardButton`](http://doc.qt.io/qt-5/qmessagebox.html#StandardButton-enum)的值通过使用`QMessageBox.Ok`，`QMessageBox.Open`等提供。


### 添加操作到组件 Adding Operations to Components

例如，您可能需要在解压缩内容后，在复制文件或修补文件内容时添加自定义操作。 您可以使用[component::addOperation()](http://doc.qt.io/qtinstallerframework/scripting-component.html#addOperation-method)从脚本中创建和添加更新操作到安装。 如果需要运行需要管理权限的操作，请改用[component::addElevatedOperation()](http://doc.qt.io/qtinstallerframework/scripting-component.html#addElevatedOperation-method)。

操作需要在实际安装步骤之前添加。 重写[component::createOperations()](http://doc.qt.io/qtinstallerframework/scripting-component.html#createOperations-method)来注册组件的自定义操作。

每个操作都有一个用于识别的唯一键，最多可以占用五个参数。在参数值中，可以使用在[installer::setValue()](http://doc.qt.io/qtinstallerframework/scripting-installer.html#setValue-method)中设置的变量。 有关详细信息，请参阅[预定义变量](#7_4_7)。

有关所有可用操作的摘要信息，请参阅[操作](#7_5)。

### 注册自定义组件 Registering Custom Operations

您可以通过继承[`KDUpdater::UpdateOperation`](http://doc.qt.io/qtinstallerframework/kdupdater-updateoperation.html)类在安装程序中注册自定义安装操作。
下面的代码显示您必须实现的方法：

```c++
#include <KDUpdater/UpdateOperation>

class CustomOperation : public KDUpdater::UpdateOperation
{
public:
  CustomOperation()
  {
      setName( "CustomOperation" );
  }

  void backup()
  {
      // do whatever is needed to restore the state in undoOperation()
      // 恢复所有需要在undoOperation()中需要恢复的状态
  }

  bool performOperation()
  {
      const QStringList args = arguments();
      // do whatever is needed to do for the given arguments
      // 获取所有需要给定的参数

      bool success = ...;
      return success;
  }

  void undoOperation()
  {
      // restore the previous state, as saved in backup()
      // 恢复以前的状态，保存在backup()中
  }

  bool testOperation()
  {
      // currently unused
      // 当前未使用
      return true;
  }

  CustomOperation* clone() const
  {
      return new CustomOperation;
  }

  QDomDocument toXml()
  {
      // automatically adds the operation's arguments and everything set via setValue
      // 自动添加操作的参数，并通过setValue设置所有
      QDomDocument doc = KDUpdater::UpdateOperation::toXml();

      // if you need any information to undo the operation you did,
      // add them to the doc here
      // 如果您需要任何信息来撤消您所做的操作，请在此处将其添加到doc

      return doc;
  }

  bool fromXml( const QDomDocument& doc )
  {
      // automatically loads the operation's arguments and everything set via setValue
      // 自动加载操作参数并通过setValue设置所有
      if( !KDUpdater::UpdateOperation::fromXml( doc ) )
          return false;

      // if you need any information to undo the operation you did,
      // read them from the doc here
      // 如果您需要任何信息来撤消您所做的操作，请在此处将其添加到doc

      return true;
  }
};
```
最后，需要注册您的自定义操作类，如下所示：

```c++
#include <KDupdater/UpdateOperationFactory>

KDUpdater::UpdateOperationFactory::instance().registerUpdateOperation< CustomOperation >( "CustomOperation" );
```
现在，您可以用与预定义操作相同的方式在安装程序中使用您自定义的操作。


### <span id="7_4_7">预定义变量 Predefined Variables</span>

您可以在脚本中使用以下预定义变量，以便直接访问：

|Symbol|描述|
|:---|:---|
|ProductName|要安装的产品的名称，在config.xml中定义的|
|ProductVersion|要安装的产品的版本号，在config.xml中所定义|
|Title|安装程序的标题，在config.xml中定义|
|Publisher|安装程序的发布者，如config.xml中所定义|
|Url|产品URL，如config.xml中所定义|
|StartMenuDir|开始菜单组，如config.xml中所定义。 仅在Windows上可用|
|TargetDir|目标安装目录，由用户选择|
|DesktopDir|用户桌面目录名(路径)。<br>仅在Windows上可用|
|os|当前平台:"X11","Win"或"Mac"。<br>不建议使用此变量：请改用systemInfo|
|RootDir|文件系统根目录|
|HomeDir|当前用户的home目录|
|ApplicationsDir|应用程序目录。<br>例如,Windows上的`C:\Program Files`,Linux上`/opt`以及OS X上`/Applications`|
|InstallerDirPath|包含安装程序可执行文件的目录|
|InstallerFilePath|安装程序可执行文件的文件路径|
|UserStartMenuProgramsPath|包含在(当前)用户的开始菜单中的项目文件夹的路径。<br>例如：`C:\Users\USERNAME\AppData\Roaming\Microsoft\Windows\Start Menu\Programs`。仅在Windows上可用|
|AllUsersStartMenuProgramsPath|包含在所有用户的开始菜单中的项目文件夹的路径。<br>例如：`C:\ProgramData\Microsoft\Windows\Start Menu\Programs`。仅在Windows上可用|

变量可以通过调用[`installer::value(string key, string defaultValue = "")`](http://doc.qt.io/qtinstallerframework/scripting-installer.html#value-method)来解析。如果嵌入在'@'中，它们也可以作为安装操作的参数传递字符串的一部分：

```js
if (installer.value("os") === "win") {
    component.addOperation("CreateShortcut", "@TargetDir@/MyApp.exe", "@StartMenuDir@/MyApp.lnk");
}
```

## <span id="7_5">操作 Operations</span>


操作由组件和控制器脚本提供并由安装程序执行。

注意：操作是线性执行的。

在内部，每个操作具有包含用于安装程序的指令的`DO步骤`和包含用于卸载程序的指令的`UNDO步骤`。

### 操作概要 Summary of Operations

下表总结了可用的操作及其语法。

|Operation|语法|用法|
|:---|:---|:---|
|Copy|"Copy" source target|从source复制一个文件到target|
|Move|"Move" source target|从source移动一个文件到target|
|SimpleMoveFile|"SimpleMoveFile" source target|从source移动一个文件到target|
|Delete|"Delete" filename|删除filename.指定的文件|
|Mkdir|"Mkdir" path|创建path目录|
|Rmdir|"Rmdir" path|删除path目录|
|CopyDirectory|"CopyDirectory" sourcePath targetPath|复制sourcePath目录到targetPath|
|AppendFile|"AppendFile" filename text|将text追加到filename指定的文件。text被视为ASCII文本|
|PrependFile|"PrependFile" filename text|将text添加到filename指定的文件。text被视为ASCII文本|
|Replace|"Replace" file search replace|打开文件去查找search字符串，并替换为replace字符串|
|LineReplace|"LineReplace" file search replace|打开文件以查找以search字符串开头的行，并用replace字符串替换它。Lines are trimmed before the search.|
|Execute|"Execute" [{exitcodes}] command [parameter1 [parameter... [parameter10]]]|执行命令指定的命令。 最多可以传递10个参数。 如果这还不够，可以使用JavaScript字符串数组。<br>可选，您可以在大括号`{}`中传递以逗号分隔的退出代码列表作为第一个参数，以指定成功执行的退出代码。 默认为`{0}`。<br>其它可选的命名参数是：`"workingdirectory=<your_working_dir>"`; `"errormessage=<your_custom_errormessage>"`<br>此外，一个特殊的参数`UNDOEXECUTE`将操作的DO步骤与UNDO步骤分开。<br>示例:`component.addOperation("Execute", "touch", "test.txt", "UNDOEXECUTE", "rm", "test.txt")`|
|CreateShortcut|"CreateShortcut" filename linkname [arguments]|为filename指定的文件创建一个名为linkname的快捷方式。<br>在Windows上，将创建一个可带参数的 **.lik** 文件。<br>在Unix上，将创建一个符号链接|
|CreateDesktopEntry|"CreateDesktopEntry" filename "key=value[ key2=value2[ key3=value3]]]"|创建一个`.desktop`初始化文件，如[freedesktop.org](https://www.freedesktop.org)指定的。<br>如果filename是绝对路径，则桌面条目存储在那里。 否则，它存储在`$XDG_DATA_DIRS/applications`或`$XDG_DATA_HOME/applications`中指定的位置，包括由freedesktop.org定义的两者的默认路径。<br>键值对将写入文件。<br>该文件设置为使用UTF-8编码|
|InstallIcons|"InstallIcons" directory [Vendorprefix]|将目录的内容安装到由freedesktop.org指定的位置。<br>也就是`$XDG_DATA_DIRS/icons`，`/usr/share/icons`或`$HOME/.icons`。 文件将从其初始位置删除。请确保在从归档中提取文件的操作之后添加此操作。<br>如果您提供`Vendorprefix`，它会替换所有字符，直到遇到 **带有此前缀** 的图标文件名中的第一个破折号`- `|
|Extract|"Extract" archive targetdirectory|提前archive内容到targetdirectory|
|GlobalConfig|"GlobalConfig" company application key value <br>或<br>"GlobalConfig" scope company application key value<br>或<br>"GlobalConfig" filename key value|在配置文件中存储键的值。 <br>配置文件由文件名(使用[QSettings::NativeFormat](http://doc.qt.io/qt-5/qsettings.html#Format-enum)，可能是Windows注册表)或应用程序和公司名称指定。将范围设置为`SystemScope`以在系统范围中创建条目。<br> **注意：** 该操作当前使用[`QSettings`](http://doc.qt.io/qt-5/qsettings.html)来存储键值对。`QSettings`始终将反斜杠视为特殊字符，并且不提供用于读取或写入此类条目的API。<br>不要在段落或键名称中使用斜杠（`/`和`''`）; 反斜杠字符用于分隔子键。 在Windows中，`''`被QSettings转换为`/`，这使它们相同。 因为QSettings使用反斜杠字符来分隔子键，所以 **无法读取或写入包含斜杠或反斜杠的Windows注册表项** 。 如果需要，您应该使用本机Windows API。|
|EnvironmentVariable|"EnvironmentVariable" key value [persistent [system]]|设置环境变量key为value<br>如果persistent设置为true，那么将永久设置该变量。目前仅支持Windows。<br>如果system设置为true，则该变量设置在系统范围内有效，而不是仅对当前用户|
|RegisterFileType|"RegisterFileType" extension command [description [contentType [icon]]]|注册要通过命令打开的扩展名的文件类型。 可选，你可以指定描述(description)、内容类型(contentType)和图标(icon)。当前仅支持Windows|
|ConsumeOutput|"ConsumeOutput" installerKeyName executablePath processArguments|保存运行带有参数`processArguments`的`executablePath`的可执行文件的输出到`installerKeyName`指定的安装程序键。 可以传递其他参数|
|CreateLink|"CreateLink" linkPath targetPath|创建从linkPath指定的位置到由targetPath指定的位置的链接|
|CreateLocalRepository|"CreateLocalRepository" binaryPath repoPath|创建一个本地存储库在repoPath指定的目录。<br>对于离线安装程序，将二进制数据存储在binaryPath指定的目录|
|FakeStopProcessForUpdate|"FakeStopProcessForUpdate" processlist|在卸载期间匹配运行进程与`processlist`中的条目(逗号分隔)。 如果找到匹配，则显示一个消息框，要求用户在继续之前停止这些进程|
|License|"License" licenses|将licenses指定的许可证文件复制到目标目录中名为Licenses的子文件夹。<br>对于在包描述文件中声明<Licenses>的包，将自动添加此操作|
|MinimumProgress|"MinimumProgress"|将进度值增加1|
|SelfRestart|"SelfRestart" core|重新启动由core指定的updater或软件包管理器|
|Settings|"Settings" path method key aValue|根据method的值：`set`、`remove`、`add_array_value`和`remove_array_value`，来设置或删除位于path的settings文件或注册表中`key`的值`aValue`|

对于匹配的组建，`Extract`、`License`和`MinimumProgress`操作是自动添加非重写的[component::createOperations()](http://doc.qt.io/qtinstallerframework/scripting-component.html#createOperations-method)方法。另请参见[component::autoCreateOperations](http://doc.qt.io/qtinstallerframework/scripting-component.html#autoCreateOperations-prop)

如果发生错误，您可以使用`devtool`手动测试操作。 但是，变量不会解析，因此您需要使用绝对值。

例如，要测试复制文件：
```shell
devtool --operation DO,Copy,<source>,<target>
```


## 工具 Tools


## <span id="7_7">脚本API Scripting API</span>


## C++ API


Known Issues