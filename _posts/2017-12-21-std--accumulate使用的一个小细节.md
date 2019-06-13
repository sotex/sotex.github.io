---
layout:  post
title:  "std::accumulate使用的一个小细节"
date:  2017-12-21
categories:  其它
tags:  其它
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2017/12/21/8080121.html](http://www.cnblogs.com/oloroso/archive/2017/12/21/8080121.html)
今天使用`std::accumulate`模板函数的时候出现了一个错误，特此记录一下。
```C++
#include <iostream>
#include <numeric>

int main()
{
	int LevelColRow[][3] =
	{
		1,  2,	   1,
		2,  4,	   2,
		3,  8,	   4,
		4,  16,	   8,
		5,  32,	   16,
		6,  64,	   32,
		7,  128,   64,
		8,  256,   128,
		9,  512,   256,
		10, 1024,  512,
		11, 2048,  1024,
		12, 4096,  2048,
		13, 8192,  4096,
		14, 16384, 8192,
		15, 32768, 16384,
		16, 65536, 32768 /*,
		17, 131072, 65536/*,
		18, 262144, 131072,
		19, 524288, 262144*/
	};
	double xx = std::accumulate( std::begin( LevelColRow ), std::end( LevelColRow ),
				     0, [] (double val, int lcr[3]) { return(val + double(lcr[1]) * double(lcr[2]) ); } );
	double xx2 = 0;
	for ( auto x : LevelColRow )
	{
		xx2 += double(x[1]) * double(x[2]);
	}
	std::cout << (int64_t) xx << std::endl;
	std::cout << (int64_t) xx2 << std::endl;
	std::cout << "Hello, world!\n";
}
```
这个代码是用于求这个行列组总共有多少个格子的，但是算出的结果总是为负数。
经过排查，当格子数大于`2^31`个时候，就出现问题了，这就应该是计算的结果实际是`int32`的。
查看`accumulate`([http://en.cppreference.com/w/cpp/algorithm/accumulate](http://en.cppreference.com/w/cpp/algorithm/accumulate))的声明如下：
```C++
template< class InputIt, class T >
T accumulate( InputIt first, InputIt last, T init );

template< class InputIt, class T, class BinaryOperation >
T accumulate( InputIt first, InputIt last, T init,
              BinaryOperation op );
```
那么原因就很简单了，因为我在使用的时候，第三个参数写的是0（默认为int），改为`0.0`就可以了。