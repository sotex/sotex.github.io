---
layout: post
title:  "libfacedetection简单使用记录"
date:   2019-04-16
categories: 开源学习
tags: libfacedetection 人脸识别
comments: 1
---
# libfacedetection简单使用记录

[TOC]

## 1、源码下载

直接从github上克隆项目仓库。

```bash
git clone https://github.com/ShiqiYu/libfacedetection.git
```

## 2、编译

### 2.1、linux

这个项目使用了cmake脚本，先生成makefile。（我这里是在ArchLinux x86_64环境下测试的）

```bash
cmake -DENABLE_NEON=OFF -DCMAKE_BUILD_TYPE=RELEASE .
```

执行上面的命令成功后，执行下面语句进行编译

```bash
make -j4
```

编译完成后会同时生成动态库和静态库

```bash
[ 90%] Linking CXX static library libfacedetection.a
[100%] Linking CXX shared library libfacedetection.so
```

### 2.2、Windows MINGW64

这里和上面也是一样的。先使用`cmake`生产`Makefile`文件，然后执行编译即可。这里我指定了使用`g++`作为C++的编译器，因为如果不指定会使用`gcc`去链接，导致生成动态库的时候找不到`c++`标准库的一些定义。

```bash
cmake -DENABLE_NEON=OFF -DCMAKE_BUILD_TYPE=RELEASE  -DCMAKE_CXX_COMPILER=g++ .
```

其它的与上面linux的一致。

**备注：我这里使用的是MSYS2环境，直接使用pacman命令安装的gcc/g++。**

### 2.3、VS2017 NMake编译

因为就这么几个文件，生成一个庞大的VS工程实在有点多余，所以我简单写了一个`Makefile.vc`文件，使用`nmake`脚本来生成静态库（因为源码中并没有对函数接口进行export，所以只生成静态库）。

**Makefile.vc 文件内容如下：**

```makefile
# 编译参数设置
CXX           = cl
DEFINES       = -DWIN32 -DWIN64 -DQT_NO_DEBUG -DNDEBUG -D_WINDLL
CXXFLAGS      = -nologo -Zc:wchar_t -FS -Zc:rvalueCast -Zc:inline -Zc:strictStrings -Zc:throwingNew -Zc:referenceBinding -O2 -MD -W3 -w34100 -w34189 -w44996 -w44456 -w44457 -w44458 -wd4577 -wd4467 -EHsc $(DEFINES)
INCPATH       = -I.
LIBEXE        = lib
LIBFLAGS      = /NOLOGO 
LIBS          =  kernel32.lib

# 源文件
SOURCES       = facedetectcnn.cpp			\
				facedetectcnn-floatdata.cpp	\
				facedetectcnn-int8data.cpp	\
				facedetectcnn-model.cpp
OBJECTS       = facedetectcnn.obj			\
				facedetectcnn-floatdata.obj	\
				facedetectcnn-int8data.obj	\
				facedetectcnn-model.obj

# 输出目标文件
DESTDIR_TARGET = libfacedetection.lib

# 编译规则
.cpp.obj::
    $(CXX) @<< -c $(CXXFLAGS) $(INCPATH) $<
<<

all:	$(DESTDIR_TARGET)

$(DESTDIR_TARGET):	$(OBJECTS)
	$(LIBEXE) $(LIBFLAGS) /OUT:$(DESTDIR_TARGET) $(LIBS) $(OBJECTS)

facedetectcnn.obj: facedetectcnn.cpp facedetectcnn.h
facedetectcnn-floatdata.obj: facedetectcnn-floatdata.cpp facedetectcnn.h
facedetectcnn-int8data.obj: facedetectcnn-int8data.cpp facedetectcnn.h
facedetectcnn-model.obj: facedetectcnn-model.cpp facedetectcnn.h

clean:
	del /Q $(OBJECTS)
```

在`VS2017开发者命令行工具中`直接使用`nmake`命令来生成静态库

```cmd
nmake -f Makefile.vc
```

![1555383674779](Image/libfacedetection简单使用记录/1555383674779.png)

## 3、简单测试程序

使用Qt写了一个小程序，简单的测试了一下。

测试的时候发现年龄始终是`0`，然后就看了一下源码，发现其并未对年龄和面部特征点进行输出，实际在代码中也没有进行检测，年龄和面部特征点相关的代码我没有找到(2019年4月16日)。

### 3.1、测试截图

![1555388348622](Image/libfacedetection简单使用记录/1555388348622.png)

![1555388572863](Image/libfacedetection简单使用记录/1555388572863.png)

### 3.2、测试代码如下

```cpp
/*****************************************************
 *       project:libfacedetection 测试代码            *
 *       anchor:ymwh@foxmail.com/solym@sohu.com      *
 *       date:2019-4-16                              *
 *****************************************************/

#include <facedetectcnn.h>

#include <QApplication>
#include <QWidget>
#include <QLabel>
#include <QLineEdit>
#include <QPushButton>
#include <QVBoxLayout>
#include <QHBoxLayout>
#include <QFileDialog>
#include <QPainter>
#include <QGraphicsView>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    QImage srcImage;
    // 创建窗口
    QWidget widget;
    // 添加控件
    QGraphicsScene* gsViewScene = new QGraphicsScene();
    gsViewScene->setItemIndexMethod(QGraphicsScene::NoIndex);
    QGraphicsView* gvImageView = new QGraphicsView(&widget);
    gvImageView->setScene(gsViewScene);

    QLineEdit* leImagePath = new QLineEdit(&widget);
    QPushButton* pbSelectFile = new QPushButton(QStringLiteral("选择文件"),&widget);
    QPushButton* pbRunDetect = new QPushButton(QStringLiteral("执行检测"),&widget);
    pbRunDetect->setEnabled(false);
    QHBoxLayout* hbLayout = new QHBoxLayout;
    // 设置布局
    hbLayout->addWidget(leImagePath);
    hbLayout->addWidget(pbSelectFile);
    hbLayout->addWidget(pbRunDetect);
    QVBoxLayout* vbLayout = new QVBoxLayout(&widget);
    vbLayout->addLayout(hbLayout);
    vbLayout->addWidget(gvImageView);
    // 添加处理操作
    QObject::connect(pbSelectFile,&QPushButton::clicked,
                     [leImagePath,gvImageView,gsViewScene,pbRunDetect,&srcImage,&widget]()
    {
        static QString path;
        path = QFileDialog::getOpenFileName(&widget,
                                            QStringLiteral("选择人像图"),
                                            path,
                                            QString("Images (*.png *.jpg *.jpeg *.jfif)"));
        if(path.isEmpty()){return;}
        QImage image;
        if(!image.load(path)){return;}
        // 图像太大的时候，执行太慢，先缩小一点
        if(image.width() > 2048){ image = image.scaledToWidth(2048); }
        if(image.width() > 1536){ image = image.scaledToWidth(1536); }

        srcImage.swap(image);
        leImagePath->setText(path);

        gsViewScene->clear();
        gsViewScene->setSceneRect(srcImage.rect());
        gsViewScene->addPixmap(QPixmap::fromImage(srcImage));
        gvImageView->fitInView(gvImageView->sceneRect());

        pbRunDetect->setEnabled(true);
    });

    QObject::connect(pbRunDetect,&QPushButton::clicked,
                     [gvImageView,gsViewScene,&srcImage]
    {
        // 转换为RGB24
        QImage image = srcImage.convertToFormat(QImage::Format_RGB888);
        int rows = image.height();
        int cols = image.width();
        int rowbytes = image.bytesPerLine();

        uchar* pImgData = image.bits();
        // RGB -> BGR 因为facedetect_cnn函数要求传入图像为BGR三波段图像
        for( int r = 0; r < rows; ++r ){
            uchar* pRow = pImgData + (r * rowbytes);
            for(int c = 0; c < cols; ++c ){
                qSwap(pRow[c*3],pRow[c*3+2]);
            }
        }
        // 进行检测
        QByteArray buffer(0x20000,0);
        int* pResults = facedetect_cnn((uchar*)buffer.data(),pImgData,cols,rows,rowbytes);
        // 将检测结果画到图像上
        QPixmap pixmap = QPixmap::fromImage(srcImage);
        if(pResults != NULL) {
            for(int i=0; i< *pResults; ++i) {
                short * p = ((short*)(pResults+1))+142*i;
                int x = p[0];
                int y = p[1];
                int w = p[2];
                int h = p[3];
                int confidence = p[4];
                // 查看源码可知，其并未进行人像框范围和置信率以外的数据赋值
                int angle = p[5];
                QPainter painter(&pixmap);
                // painter.setBrush(QBrush(QColor(255,0,0),));
                painter.setPen(Qt::green);
                // painter.drawEllipse(x,y,w,h);
                painter.drawRect(x,y,w,h);
                painter.drawText(x,y,w,h,Qt::AlignCenter,
                                 QStringLiteral("置信:%1%\n年龄:%2").arg(confidence).arg(angle));
            }
        }

        gsViewScene->clear();
        gsViewScene->setSceneRect(pixmap.rect());
        gsViewScene->addPixmap(pixmap);
        gvImageView->fitInView(gvImageView->sceneRect());
    });

    widget.resize(1024,768);
    widget.show();
    return a.exec();
}
```

