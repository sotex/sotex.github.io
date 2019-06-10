---
layout: post
title:  "MinFilter(MaxFilter)快速算法C++实现"
date:   2019-04-23
categories: 算法
tags: MinFilter
comments: 1
---
# MinFilter(MaxFilter)快速算法C++实现

[TOC]

[博客园文章地址 https://www.cnblogs.com/oloroso/p/10758029.html](https://www.cnblogs.com/oloroso/p/10758029.html)

**参考资料：**

-   [MinFilter - Wolfram 语言与系统参考资料中心](https://reference.wolfram.com/language/ref/MinFilter.html)
-   [ImageFilter - Wolfram 语言与系统参考资料中心](https://reference.wolfram.com/language/ref/ImageFilter.html)
-   [Streaming Maximum-Minimum Filter Using No More than Three Comparisons per Element](https://lemire.me/en/publication/arxiv0610046/)
-   [[SSE图像算法优化系列七：基于SSE实现的极速的矩形核腐蚀和膨胀（最大值和最小值）算法。](https://www.cnblogs.com/Imageshop/p/7018510.html)]

## 1、算法简述

### 1.1、MinFilter(MaxFilter) 算法简述

MinFilter（MaxFilter）算法是用于对一维或多维数据进行滤波的算法，滤波的结果为原数据中对应位置领域`r`内的最小（最大）值。在数据的边界处，使用较小（较大）的邻域.。

![img](Image/MinFilter快速算法Cpp实现/Image_5.gif)



### 1.2、MinFilter(MaxFilter) 快速算法简述

对于MinFilter(MaxFilter)的快速算法，思想来自于这篇论文[Streaming Maximum-Minimum Filter Using No More than Three Comparisons per Element](https://lemire.me/en/publication/arxiv0610046/)。在网上找到了这张图，但这个图也没有什么文字说明，并不是很清楚。

![img](Image/MinFilter快速算法Cpp实现/349293-20170615162729853-38699322.png)

下面按照我实现的时候的思路，来说一下我的理解。

首先，对于一个多维的数据，都可以逐个维度进行处理。比如说一个图片，也就是二维数据，可以先对每一行进行处理，然后再对每一列进行处理，这样**得到的结果与行列同时处理是一样的**。

```bash
假设r=1
原始数据     -->   逐行处理    -->  逐列处理
5 2 1 3 4       2 1 1 1 3      2 1 1 1 3
6 9 8 4 7       6 6 4 4 4      2 1 1 0 0
7 3 8 2 0       3 3 2 0 0      0 0 0 0 0
9 0 1 5 6       0 0 0 1 5      0 0 0 0 0

原始数据     -->  逐行列处理
5 2 1 3 4       2 1 1 1 3
6 9 8 4 7       2 1 1 0 0
7 3 8 2 0       0 0 0 0 0
9 0 1 5 6       0 0 0 0 0
```

因此算法的关键在于提高一行数据处理的效率。

这个算法的过程大概是这样的：

1、首先遍历一行数据中最左边的`r*2+1`个数据，获取最小值和最小值的位置。然后对左边边界部分的处理，直接赋最小值。

2、从`r+1`位置开始向后遍历，一直到右边界部分。

3、遍历的时候，判断上一次获取的最小值索引`minIndex`，是否在当前位置的领域`r`以内。

如果不在，则遍历当前位置的领域`r`范围，找出最小值的位置。也可以先与当前位置领域`r`内最右边的比较，如果最右边的小于`minIndex`位置的值，则`minIndex`就是这最右边的这个，否则就需要遍历当前位置领域`r`范围内。

如果在，则说明当前位置领域`r`内，除了最右边的元素，肯定都小于`minIndex`处的值。因为`minIndex`是当前位置上一个的领域`r`内的最小值，而上一个位置的领域`r`范围与当前位置的领域`r`范围只偏移了一个位置。

## 2、实现代码

我这里实现了对一行数据的过滤，然后在一行数据过滤的基础上实现对二维矩阵进行过滤。

对于**MaxFilter**的相关实现，只需要将下面对应的`>=`改为`<=`即可。

### 2.1、MinFilterOneRow 单行滤波代码

```cpp
/**********************************************************************//**
 * @brief	对一行数据进行滤波，每个值用邻域 r 内的最小值替换.
 * @author	solym@sohu.com/ymwh@foxmail.com
 * @date	2019/4/23
 * @param	srcData             待滤波数据地址.
 * @param	srcChanelCount      待滤波数据每个像素的通道数.
 * @param	srcChanelIndex      待滤波数据要进行滤波的通道[0,srcChanelCount).
 * @param	dstData             滤波后输出数据地址.
 * @param	dstChanelCount      滤波后输出数据每个像素的通道数.
 * @param	dstChanelIndex      滤波后数据要输出的通道索引[0,srcChanelCount).
 * @param	colnumCount         该行数据要滤波的像素数.
 * @param	radius              滤波的半径大小.
 *************************************************************************/
template<typename PixelDataType>
void MinFilterOneRow(
        PixelDataType* srcData, const size_t srcChanelCount, const size_t srcChanelIndex,
        PixelDataType* dstData, const size_t dstChanelCount, const size_t dstChanelIndex,
        const size_t colnumCount, const size_t radius)
{
    PixelDataType* pSrc = srcData;
    PixelDataType* pDst = dstData;

    size_t minIndex = 0;   // 记录最小值的下标
    size_t blockSize = radius * 2 + 1; // 块大小，以当前点为中心，左右各radius的宽度
    PixelDataType minValue = pSrc[srcChanelIndex];  // 比较中获取最小值进行记录

    // 对第一个块进行处理（i从1开始，比较(i-1,i)位置像素值）
    // 找出最小值(第一个块内的最小值，就是r位置(块中心)处的输出值)
    for(size_t iPixel=1; iPixel < blockSize; ++iPixel){
        PixelDataType value = pSrc[iPixel*srcChanelCount + srcChanelIndex];
        // 使用 >= 比 > 更快推进minIndex向前走
        if(minValue >= value){
            minValue = value;
            minIndex = iPixel;
        }
    }
    // 输出到第一个块中心(r)位置处的值。
    // 它已经是第一个块内的最小值，也就是该块左边都只能是这个值
    for(size_t i=0;i<=radius;++i){
        pDst[i*dstChanelCount + dstChanelIndex] = minValue;
    }
    // 开始处理r+1位置之后的值
    for (size_t iPixel = radius + 1; iPixel < colnumCount - radius; ++iPixel) {
        /*  i-r           i          i+r
         *   |____________|___________|_
         *       └min
         * 当前最小的索引在当前位置为中心的块的内(一定位于当前块内或前一个)
         * iPixel是当前块的中心，下面说的当前位置都指iPixel
         */
        if(minIndex >= (iPixel - radius)) {
            // 当前最小索引位置值与当前位置为中心的块的最后一个值比较
            // 根据下面的代码可知，如果mIndex在块的内部，它所在位置的值一定是最小的
            // 进入本次循环时，minIndex是上次比较的值，而上一个块与当前块等长，位置差一位
            // 所以可以直接和当前块最后一个像素值进行比较了，当前块也就完全比较完了
            size_t nextBlockFirstIndex = iPixel + radius;
            if(pSrc[minIndex*srcChanelCount + srcChanelIndex] >
                    pSrc[nextBlockFirstIndex*srcChanelCount + srcChanelIndex]){
                // 赋值当前最小值索引和值
                minIndex = nextBlockFirstIndex;
                minValue = pSrc[nextBlockFirstIndex*srcChanelCount + srcChanelIndex];
            }
        }else{
            // 如果不在当前位置为中心的块内，则对当前块进行查找最小值
            // 则将minIndex设置该块的最左边位置
            minIndex = iPixel - radius;
            // 获取当前位置为中心的块的最小值和索引
            minValue = pSrc[minIndex*srcChanelCount + srcChanelIndex];
            size_t blockEnd = minIndex + blockSize;
            for (size_t iBPixel = minIndex; iBPixel < blockEnd; ++iBPixel) {
                PixelDataType value = pSrc[iBPixel*srcChanelCount + srcChanelIndex];
                if(minValue >= value){
                    minIndex = iBPixel;
                    minValue = value;
                }
            }
        } // end if minIndex > ...
        pDst[iPixel*dstChanelCount + dstChanelIndex] = minValue;
    } // end for iPixel

    // 最后一个块中心位置的右边，一定都是和它中心位置的值是一样的
    for (size_t i = colnumCount-radius; i < colnumCount; ++i) {
        pDst[i*dstChanelCount + dstChanelIndex] = minValue;
    }
}
```

### 2.2、MinFilterOneMatrix 单个二维矩阵滤波代码

对于二维矩阵进行滤波，实际上是先进行行滤波，然后结果进行行列转置，对转置的结果再次进行行滤波，然后再行列转置输出。

```cpp
/**********************************************************************//**
 * @brief	对一个矩阵数据进行滤波，每个值用邻域 r 内的最小值替换.
 * @author	solym@sohu.com/ymwh@foxmail.com
 * @date	2019/4/23
 * @param	srcData             待滤波数据地址.
 * @param	srcBytePerRow       待滤波数据每行的字节数.
 * @param	srcChanelCount      待滤波数据每个像素的通道数.
 * @param	srcChanelIndex      待滤波数据要进行滤波的通道[0,srcChanelCount).
 * @param	dstData             滤波后输出数据地址.
 * @param	dstBytePerRow       滤波后输出数据每行的字节数.
 * @param	dstChanelCount      滤波后输出数据每个像素的通道数.
 * @param	dstChanelIndex      滤波后数据要输出的通道索引[0,srcChanelCount).
 * @param	rowCount            矩阵的行数.
 * @param	colCount            矩阵的列数.
 * @param	radius              滤波的半径大小.
 *************************************************************************/
template<typename PixelDataType>
void MinFilterOneMatrix(
        PixelDataType* srcData, const size_t srcBytePerRow,
        const size_t srcChanelCount, const size_t srcChanelIndex,
        PixelDataType* dstData, const size_t dstBytePerRow,
        const size_t dstChanelCount, const size_t dstChanelIndex,
        const size_t rowCount, const size_t colCount,
        const size_t radius)
{
    unsigned char* pSrc = reinterpret_cast<unsigned char*>(srcData);
    unsigned char* pDst = reinterpret_cast<unsigned char*>(dstData);
    // 保存中间结果
    std::vector<PixelDataType> tmpData(rowCount * colCount);
    // 逐行进行滤波
    for (size_t row = 0; row < rowCount; ++row) {
        // 获取输入和输出每行的行首位置
        PixelDataType* pSrcRowFirst = (PixelDataType*)(pSrc + row * srcBytePerRow);
        PixelDataType* pDstRowFirst = tmpData.data() + row * colCount;
        // 对当前行进行滤波
        MinFilterOneRow<PixelDataType>(pSrcRowFirst,srcChanelCount,srcChanelIndex,
                                       pDstRowFirst,1,0,
                                       colCount,radius);
    }
    // 将行滤波后的结果进行 行列转置（进行列滤波）
    std::vector<PixelDataType> tmpDataT(rowCount * colCount);
    for (size_t row = 0; row < rowCount; ++row) {
        for(size_t col = 0; col < colCount; ++col){
            tmpDataT[col*rowCount + row] = tmpData[row*colCount + col];
        }
    }
    // 对转置后的矩阵进行 逐行滤波（就是原行滤波后结果进行列滤波）
    for (size_t col = 0; col < colCount; ++col) {
        PixelDataType* pSrcColFirst = tmpDataT.data() + col * rowCount;
        PixelDataType* pDstColFirst = tmpData.data() + col * rowCount;
        // 对当前行进行滤波
        MinFilterOneRow<PixelDataType>(pSrcColFirst,1,0,
                                       pDstColFirst,1,0,
                                       rowCount,radius);
    }
    // 将行列滤波后的结果输出
    for (size_t row = 0; row < rowCount; ++row) {
        PixelDataType* pDstRowFirst = (PixelDataType*)(pDst + row * dstBytePerRow);
        for(size_t col = 0; col < colCount; ++col){
            pDstRowFirst[col*dstChanelCount+dstChanelIndex] = tmpData[col*rowCount + row];
        }
    }
}
```

## 3、测试

### 3.1 测试截图

使用Qt写了一个简单的测试程序进行测试，测试结果如下：

![1556014361058](Image/MinFilter快速算法Cpp实现/1556014361058.png)

### 3.2 测试代码

```cpp
#include "filter.hpp"
#include <QApplication>
#include <QWidget>
#include <QLineEdit>
#include <QPushButton>
#include <QComboBox>
#include <QVBoxLayout>
#include <QHBoxLayout>
#include <QFileDialog>
#include <QWebEngineView>
#include <QXmlStreamWriter>
#include <QBuffer>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    QImage srcImage,dstImage;
    // 创建窗口
    QWidget widget;
    // 添加控件
    QWebEngineView *wevView = new QWebEngineView(&widget);
    QLineEdit* leSrcImagePath = new QLineEdit(&widget);
    QPushButton* pbSelectSrcFile = new QPushButton(QStringLiteral("选择图片"),&widget);
    QComboBox* cbSelectFilterAlg = new QComboBox(&widget);
    cbSelectFilterAlg->addItems(
    { QStringLiteral("MinFilter"),QStringLiteral("MaxFilter")});
    QPushButton* pbRunFilter = new QPushButton(QStringLiteral("执行滤波"),&widget);
    //pbRunDetect->setEnabled(false);
    QHBoxLayout* hbLayout = new QHBoxLayout;
    // 设置布局
    hbLayout->addWidget(leSrcImagePath);
    hbLayout->addWidget(pbSelectSrcFile);
    hbLayout->addWidget(cbSelectFilterAlg);
    hbLayout->addWidget(pbRunFilter);
    QVBoxLayout* vbLayout = new QVBoxLayout(&widget);
    vbLayout->addLayout(hbLayout);
    vbLayout->addWidget(wevView);
    // 添加处理操作
    std::function<void(QString,const QImage&,const QImage&)>
            updateHtmlView =
            [wevView,leSrcImagePath](QString filterName,const QImage& srcImage,const QImage& resultimage)
    {
        QString tmpPath;
        QByteArray html;
        {
            QXmlStreamWriter writer(&html);
            writer.setAutoFormatting(true);
            writer.writeStartDocument();
            writer.writeStartElement("html");
            writer.writeStartElement("body");
            writer.writeAttribute("bgcolor","gray");
            if(!srcImage.isNull()){
                writer.writeTextElement("h2",QStringLiteral("原图"));
                writer.writeStartElement("img");
                QBuffer buffer;
                srcImage.save(&buffer,"PNG");
                writer.writeAttribute("src","data:image/png;base64," + buffer.data().toBase64());
                // writer.writeAttribute("src",QUrl(leSrcImagePath->text()).toString());
                writer.writeEndElement();
            }
            if(!resultimage.isNull()){
                writer.writeTextElement("h2",filterName + QStringLiteral("滤波结果图"));
                writer.writeStartElement("img");
                QBuffer buffer;
                resultimage.save(&buffer,"PNG");
                writer.writeAttribute("src","data:image/png;base64," + buffer.data().toBase64());
                // tmpPath = QDir::tempPath() + QString::fromUtf8(".png");
                // resultimage.save(tmpPath,"PNG");
                // writer.writeAttribute("src",QUrl(tmpPath).toString());
                writer.writeEndElement();
            }
            writer.writeEndElement();
            writer.writeEndElement();
        }
        wevView->setHtml(QString::fromUtf8(html));
        // if(!resultimage.isNull()){
        //     qDebug()<<tmpPath;
        //     QFile::remove(tmpPath);
        // }
    };
    
    // 文件选择按钮单击信号处理
    QObject::connect(pbSelectSrcFile,&QPushButton::clicked,
                     [leSrcImagePath,&srcImage,&widget,&updateHtmlView]()
    {
        static QString path(".");
        path = QFileDialog::getOpenFileName(&widget,
                                            QStringLiteral("选择待滤波图片"),
                                            path,
                                            QString("Images (*.png *.jpg *.jpeg *.jfif)"));
        if(path.isEmpty()){return;}
        QImage image;
        if(!image.load(path)){return;}

        srcImage = image.convertToFormat(QImage::Format_RGBA8888);
        leSrcImagePath->setText(path);
        updateHtmlView(QString(),srcImage,QImage());
    });

    // 执行滤波按钮单击信号处理
    QObject::connect(pbRunFilter,&QPushButton::clicked,
                     [&cbSelectFilterAlg,&srcImage,&dstImage,&updateHtmlView]
    {
        // 获取滤波算法名称
        QString filterName = cbSelectFilterAlg->currentText();
        // 最小值滤波 https://reference.wolfram.com/language/ref/MinFilter.html
        if(filterName == QStringLiteral("MinFilter")){
            dstImage = srcImage;
            unsigned int raduis = 2;
            uchar* pSrc = srcImage.bits();
            uchar* pDst = dstImage.bits();

            unsigned int colCount = srcImage.width();
            unsigned int rowCount = srcImage.height();
            unsigned int chanel = 4;
            // 分别对RGB通道进行滤波
            MinFilterOneMatrix<uchar>(pSrc,srcImage.bytesPerLine(),chanel,0,
                                      pDst,dstImage.bytesPerLine(),chanel,0,
                                      rowCount,colCount,raduis);
            MinFilterOneMatrix<uchar>(pSrc,srcImage.bytesPerLine(),chanel,1,
                                      pDst,dstImage.bytesPerLine(),chanel,1,
                                      rowCount,colCount,raduis);
            MinFilterOneMatrix<uchar>(pSrc,srcImage.bytesPerLine(),chanel,2,
                                      pDst,dstImage.bytesPerLine(),chanel,2,
                                      rowCount,colCount,raduis);
        }
        else if(filterName == QStringLiteral("MaxFilter")){
            dstImage = srcImage;
            unsigned int raduis = 2;
            uchar* pSrc = srcImage.bits();
            uchar* pDst = dstImage.bits();

            unsigned int colCount = srcImage.width();
            unsigned int rowCount = srcImage.height();
            unsigned int chanel = 4;
            // 分别对RGB通道进行滤波
            MaxFilterOneMatrix<uchar>(pSrc,srcImage.bytesPerLine(),chanel,0,
                                      pDst,dstImage.bytesPerLine(),chanel,0,
                                      rowCount,colCount,raduis);
            MaxFilterOneMatrix<uchar>(pSrc,srcImage.bytesPerLine(),chanel,1,
                                      pDst,dstImage.bytesPerLine(),chanel,1,
                                      rowCount,colCount,raduis);
            MaxFilterOneMatrix<uchar>(pSrc,srcImage.bytesPerLine(),chanel,2,
                                      pDst,dstImage.bytesPerLine(),chanel,2,
                                      rowCount,colCount,raduis);
        }
        updateHtmlView(filterName,srcImage,dstImage);

    });

    widget.resize(1024,768);
    widget.show();
    return a.exec();
}

```



