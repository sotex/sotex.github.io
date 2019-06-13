---
layout:  post
title:  "gcc下inline的一个问题"
date:  2018-04-25
categories:  编程
tags:  编程
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2018/04/25/8944230.html](http://www.cnblogs.com/oloroso/archive/2018/04/25/8944230.html)
今天发现一个问题，与`inline`有关,也与编译时候是不是优化有关。
大概问题可以用下面的代码来描述：

 先写一个`libtest1`，代码如下
**libtest1.h**
```cpp
#ifndef LIBTEST_H
#define LIBTEST_H

class Test{
	public:
		inline void fun1()const;
		void fun2()const;
};
#endif //!LIBTEST_H
```
**libtest1.cpp**
```cpp
#include <stdio.h>
#include "libtest.h"

void Test::fun1()const
{
	puts("fun1");
}

void Test::fun2()const
{
	fun1();
	puts("fun2 call fun1");
}
```
编译为动态库，使用命令为：`gcc -shared -fpic libtest.cpp -o libtest1.so`

然后第二个动态库`libtest2`，代码如下
```cpp
#include "libtest.h"

extern "C" void fun3()
{
	Test t;
	t.fun1();
	t.fun2();
}
```
编译命令为:`gcc -shared -fpic libtest2.cpp -o libtest2.so -Wl,-rpath=. -L. -ltest1`

然后写测试代码，运行时加载`libtest2.so`，然后调用`fun3`函数。代码如下
```cpp
#include <stdio.h>
#include <dlfcn.h>

typedef void (FuncType)();

int main()
{
	//void* p = dlopen("./libtest2.so",RTLD_NOW);
	void* p = dlopen("./libtest2.so",RTLD_LAZY);
	if(p == NULL){
		printf("dlopen libtest2.so failed:%s\n",dlerror());
		return 0;
	}
	FuncType* f1 = (FuncType*)dlsym(p,"fun3");
	if(f1 == NULL){
		printf("dlsym fun3 failed:%s\n",dlerror());
		return 0;
	}
	f1();
	dlclose(p);
	return 0;
}
```
编译执行结果如下：
```bash
/home/o/sopath [o@o-pc] [13:40]
> gcc test.cpp -o test -ldl                                            

/home/o/sopath [o@o-pc] [13:41]
> ./test 
fun1
fun1
fun2 call fun1
```
看起来好像没有问题，但是这里编译的时候都没有进行优化，使用的默认选项，如果我们编译命令修改一下，则就变了
```
/home/o/sopath [o@o-pc] [13:34]
> gcc -shared -fpic libtest.cpp -o libtest.so -O3
/home/o/sopath [o@o-pc] [13:41]
> ./test 
./test: symbol lookup error: ./libtest2.so: undefined symbol: _ZNK4Test4fun1Ev
```
这时候就找不到`fun1`这个函数了，使用`strings libtest1.so`也确实找不到。但是如果把fun1前面的`inline`去掉，就没有问题了。