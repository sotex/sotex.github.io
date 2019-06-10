---
layout: post
title:  "RGBA alpha 透明度混合算法实现和测试"
date:   2019-04-17
categories: 算法
tags: RGBA AlphaBlend
comments: 1
---
# RGBA alpha 透明度混合算法实现和测试

[TOC]

[博客园文章地址 https://www.cnblogs.com/oloroso/p/10724803.html](https://www.cnblogs.com/oloroso/p/10724803.html)

## 1、算法叙述

算法参考自：[【RGBA alpha 透明度混合算法】](https://blog.csdn.net/xhhjin/article/details/6445460) ，下面的叙述和实现中有一些个人修改在里面。

### 1.1、透明度混合算法1

**R1、G1、B1、Alpha1 **为前景颜色值，**R2、G2、B2、Alpha2** 为背景颜色值，则：

```bash
Alpha = 1 - (1 - Alpha1) * ( 1 - Alpha2)
R = (R1 * Alpha1 + R2 * Alpha2 * (1-Alpha1))/Alpha
G = (G1 * Alpha1 + G2 * Alpha2 * (1-Alpha1))/Alpha
B = (B1 * Alpha1 + B2 * Alpha2 * (1-Alpha1))/Alpha
```

这里的Alpha取值范围是[0,1]，需要使用到浮点计算（实数计算）。对于我们常见的8位图像，我们可以将其值域改为[0,255]进行计算，具体的见下面测试代码。

1.2、AlphaBlend算法介绍

混合算法目前在常用到的算法是**AlphaBlend**。
计算公式如下:假设一幅图像是A，另一幅透明的图像是B，那么**透过B去看A**，看上去的图像C就是B和A的混合图像，
      设B图像的透明度为`alpha`(取值为0-1，1为完全透明，0为完全不透明).
      `Alpha`混合公式如下：

```bash
R(C)=(1-alpha)*R(B) + alpha*R(A)
G(C)=(1-alpha)*G(B) + alpha*G(A)
B(C)=(1-alpha)*B(B) + alpha*B(A)
```

R(x)、G(x)、B(x)分别指颜色x的RGB分量原色值。从上面的公式可以知道，**Alpha其实是一个决定混合透明度的数值**。

这里只对B图的Alpha进行了处理，但A图本身如果也有透明通道的，也需要进行一样的处理，即

```bash
A(C)=(1-alpha)*A(B) + alpha*A(A)
```

### 1.3、简易Alpha混合算法

首先，要能取得上层与下层颜色的 RGB三基色，然后用r,g,b 为最后取得的颜色值；**r1、g1、b1**是上层的颜色值；**r2、g2、b2**是下层颜色值，若Alpha=上层透明度，则：

-   当Alpha=50%时

    ```bash
    r = r1/2 + r2/2;
    g = g1/2 + g2/2;
    b = b1/2 + b2/2;
    ```

-   当Alpha<50%时

    ```bash
    r = r1 - r1/Alpha + r2/Alpha;
    g = g1 - g1/Alpha + g2/Alpha;
    b = b1 - b1/Alpha + b2/Alpha;
    ```

-   当Alpha>50%时

    ```bash
    r = r1/Alpha + r2 - r2/Alpha;
    g = g1/Alpha + g2 - g2/Alpha;
    b = b1/Alpha + b2 - b2/Alpha;
    ```

这个算法比较简单，我也没有去做实现。简单来说这里就是看透明度高是否超过50%，来决定是上下层图像谁占主导地位。

## 2、算法实现代码和测试

实现其实是很简单的，只是注意下面没有实数的计算，都是使用的整数计算，要注意移位与抹零的操作别出错。

下面的`rgba1`表示前景图（我的代码里是水印图）的一个像素值，是RGBA8888格式的。`rgba2`表示背景图的一个像素值。`a1`表示`rgba1`中的`Alpha`分量值，`a2`表示`rgba2`中的`Alpha`分量值。

### 2.1、透明度混合算法1实现代码

```cpp
// 如果alpha的值域是[0,1]，这里相当于将其拉伸为[0,255]
// 所以相当于是 Alpha = 1 - (1 - Alpha1) * ( 1 - Alpha2)乘以了两次255
uint32_t A = (0xffff - (0xff - a1)*(0xff - a2));
// 下面左边部分少左移8位，相当于乘以了255
uint32_t R = (((rgba1 << 8 &0xff00U) * a1 + (rgba2 >> 0 &0xffU) * a2 *(0xff - a1))/A)&0xffU;
uint32_t G = (((rgba1 >> 0 &0xff00U) * a1 + (rgba2 >> 8 &0xffU) * a2 *(0xff - a1))/A)&0xffU;
uint32_t B = (((rgba1 >> 8 &0xff00U) * a1 + (rgba2 >> 16&0xffU) * a2 *(0xff - a1))/A)&0xffU;
```

### 2.1、AlphaBlend算法实现代码

```cpp
uint32_t A = a1;
uint32_t R = (((rgba1 >> 0 &0xffU) * A + (rgba2 >> 0 &0xffU) *(0xff - A)) >> 8)&0xffU;
uint32_t G = (((rgba1 >> 8 &0xffU) * A + (rgba2 >> 8 &0xffU) *(0xff - A)) >> 8)&0xffU;
uint32_t B = (((rgba1 >> 16&0xffU) * A + (rgba2 >> 16&0xffU) *(0xff - A)) >> 8)&0xffU;
A = ((a1 * A + a2 *(0xff - A)) >> 8)&0xffU; // 必须对Alpha波段也处理
```

### 2.3、测试截图

![1555492062975](Image/RGBA_alpha透明度混合算法实现和测试/1555492062975.png)

### 2.4、完整测试程序代码

```cpp
#include <QApplication>
#include <QWidget>
#include <QLineEdit>
#include <QPushButton>
#include <QVBoxLayout>
#include <QHBoxLayout>
#include <QFileDialog>
#include <QWebEngineView>
#include <QXmlStreamWriter>
#include <QBuffer>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    QImage bkImage,wmImage;
    QImage mixImage1,mixImage2;
    // 创建窗口
    QWidget widget;
    // 添加控件
    QWebEngineView *wevView = new QWebEngineView(&widget);
    QLineEdit* leBkImagePath = new QLineEdit(&widget);
    QLineEdit* leWmImagePath = new QLineEdit(&widget);
    QPushButton* pbSelectBkFile = new QPushButton(QStringLiteral("选择背景图"),&widget);
    QPushButton* pbSelectWmFile = new QPushButton(QStringLiteral("选择水印图"),&widget);
    QPushButton* pbRunMix = new QPushButton(QStringLiteral("执行叠加"),&widget);
    //pbRunDetect->setEnabled(false);
    QHBoxLayout* hbLayout = new QHBoxLayout;
    // 设置布局
    hbLayout->addWidget(leBkImagePath);
    hbLayout->addWidget(pbSelectBkFile);
    hbLayout->addWidget(leWmImagePath);
    hbLayout->addWidget(pbSelectWmFile);
    hbLayout->addWidget(pbRunMix);
    QVBoxLayout* vbLayout = new QVBoxLayout(&widget);
    vbLayout->addLayout(hbLayout);
    vbLayout->addWidget(wevView);
    // 添加处理操作
    std::function<void()> updateHtmlView =
            [wevView,&bkImage,&wmImage,&mixImage1,&mixImage2]()
    {
        QByteArray html;
        {
            QXmlStreamWriter writer(&html);
            writer.setAutoFormatting(true);
            writer.writeStartDocument();
            writer.writeStartElement("html");
            writer.writeStartElement("body");
            writer.writeAttribute("bgcolor","gray");
            QStringList imgName =
            {
                QStringLiteral("背景图"),
                QStringLiteral("水印图"),
                QStringLiteral("算法1结果图"),
                QStringLiteral("算法2结果图")
            };
            QList<QImage*> imgRef =
            {
                &bkImage,&wmImage,&mixImage1,&mixImage2
            };
            for(int i=0;i<imgName.size();++i){
                if(imgRef[i]->isNull()){continue;}
                writer.writeTextElement("h2",imgName[i]);
                writer.writeStartElement("img");
                QBuffer buffer;
                imgRef[i]->save(&buffer,"PNG");
                writer.writeAttribute("src","data:image/png;base64," + buffer.data().toBase64());
                writer.writeEndElement();
            }
            writer.writeEndElement();
            writer.writeEndElement();
        }
        wevView->setHtml(QString::fromUtf8(html));

    };
    QObject::connect(pbSelectBkFile,&QPushButton::clicked,
                     [leBkImagePath,&bkImage,&widget,&updateHtmlView]()
    {
        static QString path(".");
        path = QFileDialog::getOpenFileName(&widget,
                                            QStringLiteral("选择背景图"),
                                            path,
                                            QString("Images (*.png *.jpg *.jpeg *.jfif)"));
        if(path.isEmpty()){return;}
        QImage image;
        if(!image.load(path)){return;}

        bkImage = image.convertToFormat(QImage::Format_RGBA8888);
        leBkImagePath->setText(path);
        updateHtmlView();
    });
    QObject::connect(pbSelectWmFile,&QPushButton::clicked,
                     [leWmImagePath,&bkImage,&wmImage,&widget,&updateHtmlView]()
    {
        static QString path(".");
        path = QFileDialog::getOpenFileName(&widget,
                                            QStringLiteral("选择水印图"),
                                            path,
                                            QString("Images (*.png)"));
        if(path.isEmpty()){return;}
        QImage image;
        if(!image.load(path)){return;}
        // 水印图不能比背景图大
        int w = image.width() * 2 > bkImage.width() ? bkImage.width()/2:image.width();
        int h = image.height() * w / image.width();
        h = h > bkImage.height()?bkImage.height():h;

        wmImage = image.scaledToHeight(h).convertToFormat(QImage::Format_RGBA8888);
        leWmImagePath->setText(path);
        updateHtmlView();
    });

    QObject::connect(pbRunMix,&QPushButton::clicked,
                     [&bkImage,&wmImage,&mixImage1,&mixImage2,&updateHtmlView]
    {
        mixImage1 = mixImage2 = bkImage;
        for(int r = 0;r < wmImage.height();++r){
            uint32_t* pBgLine = reinterpret_cast<uint32_t*>(bkImage.bits() + bkImage.bytesPerLine()*r);
            uint32_t* pWmLine = reinterpret_cast<uint32_t*>(wmImage.bits() + wmImage.bytesPerLine()*r);
            uint32_t* pM1Line = reinterpret_cast<uint32_t*>(mixImage1.bits() + mixImage1.bytesPerLine()*r);
            uint32_t* pM2Line = reinterpret_cast<uint32_t*>(mixImage2.bits() + mixImage2.bytesPerLine()*r);
            for(int c=0;c<wmImage.width();++c){
                uint32_t rgba1 = pWmLine[c];
                uint32_t rgba2 = pBgLine[c];
                uint32_t a1 = rgba1 >> 24;
                uint32_t a2 = rgba2 >> 24;

                {
                    // 如果alpha的值域是[0,1]，这里相当于将其拉伸为[0,255]
                    // 所以相当于是 Alpha = 1 - (1 - Alpha1) * ( 1 - Alpha2)乘以了两次255
                    uint32_t A = (0xffff - (0xff - a1)*(0xff - a2));
                    // 下面左边部分少左移8位，相当于乘以了255
                    uint32_t R = (((rgba1 << 8 &0xff00U) * a1 + (rgba2 >> 0 &0xffU) * a2 *(0xff - a1))/A)&0xffU;
                    uint32_t G = (((rgba1 >> 0 &0xff00U) * a1 + (rgba2 >> 8 &0xffU) * a2 *(0xff - a1))/A)&0xffU;
                    uint32_t B = (((rgba1 >> 8 &0xff00U) * a1 + (rgba2 >> 16&0xffU) * a2 *(0xff - a1))/A)&0xffU;
                    pM1Line[c] = R|(G<<8)|(B<<16)|((A&0xff)<<24);
                }
                {
                    uint32_t A = a1;
                    uint32_t R = (((rgba1 >> 0 &0xffU) * A + (rgba2 >> 0 &0xffU) *(0xff - A)) >> 8)&0xffU;
                    uint32_t G = (((rgba1 >> 8 &0xffU) * A + (rgba2 >> 8 &0xffU) *(0xff - A)) >> 8)&0xffU;
                    uint32_t B = (((rgba1 >> 16&0xffU) * A + (rgba2 >> 16&0xffU) *(0xff - A)) >> 8)&0xffU;
                    A = ((a1 * A + a2 *(0xff - A)) >> 8)&0xffU; // 必须对Alpha波段也处理
                    pM2Line[c] = R|(G<<8)|(B<<16)|(A<<24);
                }
            }
        }
        updateHtmlView();
    });

    widget.resize(1024,768);
    widget.show();
    return a.exec();
}
```