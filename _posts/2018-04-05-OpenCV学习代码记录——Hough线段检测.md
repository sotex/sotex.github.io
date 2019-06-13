---
layout:  post
title:  "OpenCV学习代码记录——Hough线段检测"
date:  2018-04-05
categories:  其它
tags:  其它
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2018/04/05/8722002.html](http://www.cnblogs.com/oloroso/archive/2018/04/05/8722002.html)
很久之前学习过一段时间的OpenCV，当时没有做什么笔记，但是代码都还在，这里把它贴出来做个记录。
代码放在码云上，地址在这里[https://gitee.com/solym/OpenCVTest/tree/master/OpenCVTest](https://gitee.com/solym/OpenCVTest/tree/master/OpenCVTest)

## 效果

![](https://images2018.cnblogs.com/blog/693958/201808/693958-20180813205814310-1719794655.jpg)


## 代码

```cpp
#include <opencv2/core.hpp>
#include <opencv2/highgui.hpp>
#include <opencv2/imgproc.hpp>


// http://blog.csdn.net/zhaocj/article/details/50281537

int houghLineDetection()
{
	// 加载原图像
	cv::Mat src = cv::imread("../Image/houghline2.jpg");
	if (src.empty()) {
		puts("原图像读取失败!!!");
		return -1;
	}
	cv::Mat	gray(src.size(), CV_8UC1);	// 原图的灰度图像
	cv::cvtColor(src, gray, CV_BGR2GRAY);

	cv::Mat	edge(src.size(), CV_8UC1);	// 边缘图像
	cv::Mat dst(src.size(), src.type());// 绘制直线的目标图像、
	cv::cvtColor(gray, dst, CV_GRAY2BGR);	// 从灰度图转过来，便于后面绘制红色的线条

	// 进行Canny边缘检测
	cv::Canny(gray, edge, 50, 200, 3);

#if 0
	// 定义输出数组，用于存储直线的角度和距离这两个变量  
	std::vector<cv::Vec2f> lines;
	// 距离分辨率为1，角度分辨率为π/180，阈值为215  
	// 阈值的选取直接影响到输出直线的数量
	cv::HoughLines(edge, lines, CV_HOUGH_PROBABILISTIC,
		CV_PI / 180.0,215,0,0);
	// 画直线
	for (size_t i = 0; i < lines.size(); ++i) {
		// 提取出距离和角度
		float rho = lines[i][0];
		float theta = lines[i][1];

		// 定义两个点，确定一条直线
		cv::Point p1, p2;
		// 计算得到两个点的坐标
		// （ρcosθ-1000sinθ，ρsinθ+1000cosθ），（ρcosθ+1000sinθ，ρsinθ-1000cosθ）  
		double a = cos(theta);
		double b = sin(theta);
		double x0 = a*rho;
		double y0 = b*rho;
		p1.x = cvRound(x0 + 1000 * (-b));
		p1.y = cvRound(y0 + 1000 * a);
		p2.x = cvRound(x0 - 1000 * (-b));
		p2.y = cvRound(y0 - 1000 * a);
		// 在灰度图上画宽度为2的红线
		cv::line(dst, p1, p2, cv::Scalar(0, 0, 255), 2, 8);
	}
#else
	std::vector<cv::Vec4i> lines;
	cv::HoughLinesP(edge, lines, CV_HOUGH_PROBABILISTIC,
		CV_PI / 180.0, 80, 30, 10);
	for (size_t i = 0; i < lines.size(); ++i) {
		cv::line(dst, cv::Point(lines[i][0], lines[i][1]),
			cv::Point(lines[i][2], lines[i][3]), cv::Scalar(0, 0, 255), 2, 8);
	}

#endif

	// 创建窗口显示原图和目标图
	cv::namedWindow("src");
	cv::namedWindow("dst");
	cv::imshow("src", src);
	cv::imshow("dst", dst);
	cv::imshow("edge", edge);

	// 等待按键退出
	cv::waitKey();
	return 0;
}
```