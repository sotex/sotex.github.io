---
layout:  post
title:  "由一次 symbol lookup error 引发的思考"
date:  2017-01-13
categories:  其它
tags:  其它
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2017/01/13/6283248.html](http://www.cnblogs.com/oloroso/archive/2017/01/13/6283248.html)
开发一个跨平台的项目的时候，大部分时候都是在VS下进行编码，所以也就使用了VS的解决方案来管理项目。
因为要跨平台，当时网上看`scons`这个工具不错，所以在linux下就使用了`scons`来作为编译脚本。

linux(gcc)下与windows(vs)下的对于链接这一步稍有不同。当目标文件是一个(共享)库的时候，VS会在链接的时候就去解析所有用到的符号，而gcc则不会，只有在生成最终可执行程序的时候才会去解析。
所以在VS下，一直都没有问题。linux下进行测试的时候也没有问题(因为不是所有的代码都被调用了)。

这几天在一次移植过程中出现了如下的问题
```shell
/a.out: symbol lookup error: ./uds_file_storage_module.so: undefined symbol: _ZN8unispace13us_ini_config9from_fileERKNS_10us_ustringEPS0_
```
错误很简单，就是`us_ini_config::from_file`这个函数没有找到，说明没有将其添加到动态导出符号表中。（uds_file_storage_module.so是运行时动态加载的，所以编译的时候没有提示错误）


VC中导出符号需要使用到`dllexport`，而gcc下则默认是不需要。所以这个问题很是疑惑。


考虑用的`gcc`版本比较高，是不是它将`-fvisibility`的参数默认设置为`hidden`呢？查看了gnu的wiki之后也没有发现这个。

[https://gcc.gnu.org/wiki/Visibility](https://gcc.gnu.org/wiki/Visibility)

这里还是有收获的，看到一段好的跨平台代码。

```cpp
#if defined _WIN32 || defined __CYGWIN__
  #ifdef BUILDING_DLL
    #ifdef __GNUC__
      #define DLL_PUBLIC __attribute__ ((dllexport))
    #else
      #define DLL_PUBLIC __declspec(dllexport) // 注记:实际上gcc似乎也支持这种语法。
    #endif
  #else
    #ifdef __GNUC__
      #define DLL_PUBLIC __attribute__ ((dllimport))
    #else
      #define DLL_PUBLIC __declspec(dllimport) // 注记:实际上gcc似乎也支持这种语法。
    #endif
  #endif
  #define DLL_LOCAL
#else
  #if __GNUC__ >= 4
    #define DLL_PUBLIC __attribute__ ((visibility ("default")))
    #define DLL_LOCAL  __attribute__ ((visibility ("hidden")))
  #else
    #define DLL_PUBLIC
    #define DLL_LOCAL
  #endif
#endif

extern "C" DLL_PUBLIC void function(int a);
class DLL_PUBLIC SomeClass
{
   int c;
   DLL_LOCAL void privateMethod();  // Only for use within this DSO
public:
   Person(int _c) : c(_c) { }
   static void foo(int a);
};
```

然后添加了`__attribute__((visibility("default")))`进行修饰，发现编译的结果没有改变(md5sun判断)。

然后又在链接的时候添加`-rdynamic`参数，将所有符号都添加到动态符号表，编译结果也没有变。

在仔细查看了`SConstruct`脚本之后，发现问题在于没有将对应的源文件添加到脚本中。也就是编译的时候完全就没有编译`us_ini_config::from_file`所在的源文件。
这样问题就很清晰明了了，修改脚本之后重新编译就可以了。

----
说到这里，就该反思一下了。
这个项目早期确实是直接写的`Makefile`来进行编译的，使用`SOURCES = $(shell find $(SRC_DIR)$(PROJECT)/ -name "*.cpp")`来自动发现cpp文件，这本来是很好的。对于贸然使用自己不熟悉的`scons`，又没有进行有效的学习，这是很不好的。
对于这个项目，还是直接使用`qmake`来做跨平台的编译脚本比较好。