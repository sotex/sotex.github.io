---
layout:  post
title:  "OpenCV学习代码记录—— Snake轮廓"
date:  2018-04-05
categories:  其它
tags:  其它
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2018/04/05/8722010.html](http://www.cnblogs.com/oloroso/archive/2018/04/05/8722010.html)
很久之前学习过一段时间的OpenCV，当时没有做什么笔记，但是代码都还在，这里把它贴出来做个记录。
代码放在码云上，地址在这里[https://gitee.com/solym/OpenCVTest/tree/master/OpenCVTest](https://gitee.com/solym/OpenCVTest/tree/master/OpenCVTest)

## 效果

![](https://images2018.cnblogs.com/blog/693958/201808/693958-20180813205628231-1509747319.jpg)

## 代码

```cpp
#include <opencv2/core.hpp>
#include <opencv2/highgui.hpp>
#include <opencv2/imgproc.hpp>
#include <opencv2/legacy/legacy.hpp>	// cvSnakeImage

// http://blog.csdn.net/hongxingabc/article/details/51606520
//http://wiki.opencv.org.cn/index.php/Snake%E8%BD%AE%E5%BB%93%E4%BE%8B%E5%AD%90

static cv::Mat	src, src_bak;	// 原始图像及其备份

static int	thresholdness = 128;	// 阈值
// 下面的参数用于滑块的值传出
static int ialpha = 20;	// alpha代表点相互靠拢的权值(0-1.0)
static int ibeta  = 20;	// beta表示弯曲能量（越小越容易弯曲）(0-1.0)
static int igamma = 20;	// gamma表示整体能量(0-1.0)

void on_change(int pos,void*);

int snakeContour()
{
	// 创建窗口
	cv::namedWindow("win", 0);
	// 添加四个进度条(滑块条)
	cv::createTrackbar("Thd", "win", &thresholdness, 255, on_change);
	cv::createTrackbar("alpha", "win", &ialpha, 100, on_change);
	cv::createTrackbar("beta", "win", &ibeta, 100, on_change);
	cv::createTrackbar("gamma", "win", &igamma, 100, on_change);
	
	cv::resizeWindow("win", 640, 480);
	// 调用一次on_change，以便显示出窗口
	on_change(0,NULL);

	while (true){
		if (cv::waitKey(40) == 27) {
			cv::destroyWindow("win");
			break;	// 按下ESC键就退出
		}
		// 显示图像到窗口
		cv::imshow("win", src_bak);
	}

	return 0;
}

void on_change(int,void*)
{
	// 读取原图像
	src_bak = cv::imread("../Image/sisy.jpg", 1);
	// 转换为灰度图像
	cv::cvtColor(src_bak, src, CV_BGR2GRAY);

	// 对单通道数组应用固定阈值操作。该函数的典型应用是对灰度图像进行阈值操作得到二值图像。
	// 二值化操作
	cv::threshold(src/*原始图像(单通道)*/,
		src/*目标图像(与原始图像类型一致)*/,
		thresholdness/*二值化控制阈值*/,
		255/*使用 CV_THRESH_BINARY 和 CV_THRESH_BINARY_INV 的最大值*/,
		CV_THRESH_BINARY/*阈值类型*/);
	// CV_THRESH_BINARY:如果 src(x, y)>threshold, dst(x, y) = max_value; 否则, dst（x, y） = 0;
	// CV_THRESH_BINARY_INV:如果 src(x, y)>threshold, dst(x, y) = 0; 否则, dst(x, y) = max_value.
	// CV_THRESH_TRUNC:如果 src(x, y)>threshold，dst(x, y) = max_value; 否则dst(x, y) = src(x, y).
	// CV_THRESH_TOZERO:如果src(x, y)>threshold，dst(x, y) = src(x, y); 否则 dst(x, y) = 0。
	// CV_THRESH_TOZERO_INV:如果 src(x, y)>threshold，dst(x, y) = 0; 否则dst(x, y) = src(x, y).
	
	std::vector<std::vector<cv::Point> > contours;

	// http://www.360doc.com/content/12/1028/14/6408986_244247418.shtml
	// 寻找初始化轮廓
	cv::findContours(src/*输入图像(2值单通道图像)*/,
		contours/*传出检测的轮廓点数组*/,
		CV_RETR_EXTERNAL/*轮廓检测模式*/,
		CV_CHAIN_APPROX_SIMPLE/*轮廓近似方式*/,
		cv::Point(0, 0)/*轮廓点偏移量*/);

	if (contours.empty()) {
		return;	//没有检测到轮廓
	}

	// 绘制轮廓线(绘制到备份的图像上)
	cv::drawContours(src_bak, contours, -1, cv::Scalar(0, 0, 255), 2, 8);

	{
	// 获取alpha、beta、gamma三个值
		float alpha = ialpha / 100.0f;
		float beta = ibeta / 100.0f;
		float gamma = igamma / 100.0f;

		// 改变轮廓位置使得它的能量最小
		// 因为没有C++的接口，所以这里使用C的接口
		IplImage ipl_src = IplImage(src);
		
		// 每个点用于搜索最小值的邻域尺寸，两个 win.width 和 win.height 都必须是奇数
		CvSize size;
		size.width = 3; size.height = 3;
		// 终止条件
		CvTermCriteria criteria;
		criteria.type = CV_TERMCRIT_ITER;
		criteria.max_iter = 1000;
		criteria.epsilon = 0.1;
		// 梯度符号。如果非零，函数为每一个图像象素计算梯度幅值，且把它当成能量场，否则考虑输入图像本身。
		int calc_gradient = 1;

		std::vector<CvPoint> point_arr;	// 用于获取一个轮廓环的特征点
		for (int nline = 0; nline < contours.size(); ++nline) {
			// 当前轮廓线点数
			int point_count = contours[nline].size();
			if (point_count > 2) {	// 低于3个点的时候，会出错
				point_arr.resize(point_count);
			} else {
				continue;
			}
			// 拷贝特征点
			for (int npt = 0; npt < contours[nline].size(); ++npt) {
				point_arr[npt] = contours[nline][npt];
			}
			// 执行改变轮廓位置使得它的能量最小
			cvSnakeImage(&ipl_src/*输入图像或外部能量域*/,
				point_arr.data()/*传入传出轮廓点*/,
				point_count/*传入轮廓点数组大小*/,
				&alpha/*连续性能量的权 Weight[s]，单个浮点数或长度为 length 的浮点数数组，每个轮廓点有一个权*/,
				&beta/*曲率能量的权 Weight[s]，与 alpha 类似*/,
				&gamma/*图像能量的权 Weight[s]，与 alpha 类似*/,
				CV_VALUE/*前面三个参数的不同使用方法*/,
				/*CV_VALUE 表示每个 alpha, beta, gamma 都是指向为所有点所用的一个单独数值;
				CV_ARRAY 表示每个 alpha, beta, gamma 是一个指向系数数组的指针，snake 上面各点的系数都不相同。
				因此，各个系数数组必须与轮廓具有同样的大小。所有数组必须与轮廓具有同样大小*/
				size, criteria, calc_gradient);

			// 显示曲线(蓝色)
			for (int prev = 0; prev < point_count; ++prev) {
				int next = (prev + 1) % point_count;
				// 绘线，蓝色
				cv::line(src_bak, point_arr[prev], point_arr[next], cv::Scalar(255, 0, 0), 2, 8, 0);
			}
		}
	}
}
```