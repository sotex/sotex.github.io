---
layout:  post
title:  "Windows下 VS2015编译RocksDB"
date:  2017-01-20
categories:  编程
tags:  编程
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2017/01/20/6323124.html](http://www.cnblogs.com/oloroso/archive/2017/01/20/6323124.html)
VS2015编译RocksDB

RocksDB 是一个来自 facebook 的可嵌入式的支持持久化的 key-value 存储系统，也可作为 C/S 模式下的存储数据库，但主要目的还是嵌入式。RocksDB 基于 LevelDB 构建。

## 1、下载rocksdb源码

```shell
git clone https://github.com/facebook/rocksdb.git
```

## 2、使用CMAKE生成VS工程

打开cmd窗口(最好使用`VS2015开发人员命令提示`)，进入源码目录，执行下面命令
```shell
mkdir msvc14	# 创建构建目录
cd msvc14		# 进入构建目录
# 生成VS项目(D:\Libs\rocksdb是编译后安装路径)
cmake -DCMAKE_INSTALL_PREFIX=D:\Libs\rocksdb -G "Visual Studio 14 Win64" ..
```
**实际上rocksdb源码目录中的`CMakeList.txt`文件中并没有写`INSTALL`段的内容，所以这里指定安装路径实际是无效的。**


## 3、编译RocksDB
打开`VS2015 x64 本机工具命令提示`(或VS2015开发人员命令提示)工具，执行下面命令进行编译64位release版本的RocksDB库
```shell
msbuild ALL_BUILD.vcproject /p:configuration=release /maxcpucount:8
```
因为test项目也都编译了，文件较多，所以这里使用了`/maxcpucount:8`指定使用的CPU核心数。
如果只编译rocksdb(静态)库，可以使用`msbuild rocksdblib.vcproject /p:configuration=release`命令
编译动态库使用`msbuild rocksdb.vcproject /p:configuration=release`命令

编译完成后，通过的做法是使用下面命令进行安装
```shell
msbuild INSTALL.vcproject /p:configuration=release
```
**但是这里没有生成INSTALL.vcproject，所以这里的安装需要手动进行。**
直接拷贝`msvc14/Release`目录下的`rocksdblib.lib`文件和源码目录的`include`目录即可。

## 4、编译后续
因为编译出来的是动态库，但是实际上`RocksDB`的代码中比没有使用`__declspec(dllexport)`导出函数和类接口，所以在使用的时候会遇到类似如下的问题
```bash
rocksdb_test.obj : error LNK2019: 无法解析的外部符号 "public: __cdecl rocksdb::ColumnFamilyOptions::ColumnFamilyOptions(void)" (??0ColumnFamilyOptions@rocksdb@@QEAA@XZ)，该符号在函数 "public: __cdecl rocksdb::Options::Options(void)" (??0Options@rocksdb@@QEAA@XZ) 中被引用
rocksdb_test.obj : error LNK2019: 无法解析的外部符号 "public: __cdecl rocksdb::DBOptions::DBOptions(void)" (??0DBOptions@rocksdb@@QEAA@XZ)，该符号在函数 "public: __cdecl rocksdb::Options::Options(void)" (??0Options@rocksdb@@QEAA@XZ) 中被引用
rocksdb_test.obj : error LNK2019: 无法解析的外部符号 "public: __cdecl rocksdb::ReadOptions::ReadOptions(void)" (??0ReadOptions@rocksdb@@QEAA@XZ)，该符号在函数 main 中被引用
rocksdb_test.obj : error LNK2019: 无法解析的外部符号 "public: static class rocksdb::Status __cdecl rocksdb::DB::Open(struct rocksdb::Options const &,class std::basic_string<char,struct std::char_traits<char>,class std::allocator<char> > const &,class rocksdb::DB * *)" (?Open@DB@rocksdb@@SA?AVStatus@2@AEBUOptions@2@AEBV?$basic_string@DU?$char_traits@D@std@@V?$allocator@D@2@@std@@PEAPEAV12@@Z)，该符号在函数 main 中被引用
rocksdb_test.exe : fatal error LNK1120: 4 个无法解析的外部命令
```
这里可以打开生成的VS工程，将配置类型改为静态库，然后再生成静态库即可。
![](http://images2015.cnblogs.com/blog/693958/201703/693958-20170322114233690-788973321.png)

生成后即可使用了。但是还会遇到下面的问题
```bash
rocksdb.lib(env_win.obj) : error LNK2019: 无法解析的外部符号 __imp_RpcStringFreeA，该符号在函数 "public: virtual class std::basic_string<char,struct std::char_traits<char>,class std::allocator<char> > __cdecl rocksdb::Env::GenerateUniqueId(void)" (?GenerateUniqueId@Env@rocksdb@@UEAA?AV?$basic_string@DU?$char_traits@D@std@@V?$allocator@D@2@@std@@XZ) 中被引用
rocksdb.lib(env_win.obj) : error LNK2019: 无法解析的外部符号 __imp_UuidCreateSequential，该符号在函数 "public: virtual class std::basic_string<char,struct std::char_traits<char>,class std::allocator<char> > __cdecl rocksdb::Env::GenerateUniqueId(void)" (?GenerateUniqueId@Env@rocksdb@@UEAA?AV?$basic_string@DU?$char_traits@D@std@@V?$allocator@D@2@@std@@XZ) 中被引用
rocksdb.lib(env_win.obj) : error LNK2019: 无法解析的外部符号 __imp_UuidToStringA，该符号在函数 "public: virtual class std::basic_string<char,struct std::char_traits<char>,class std::allocator<char> > __cdecl rocksdb::Env::GenerateUniqueId(void)" (?GenerateUniqueId@Env@rocksdb@@UEAA?AV?$basic_string@DU?$char_traits@D@std@@V?$allocator@D@2@@std@@XZ) 中被引用
rocksdb_test.exe : fatal error LNK1120: 3 个无法解析的外部命令
```
这些函数实际上是在`Rpcrt4.lib`这个库里面，所以需要链接上才行。可以在使用的时候在代码中添加`#pragma comment(lib,"Rpcrt4.lib")`来引入。

相关测试代码如下
```cpp
#include <cassert>  
#include <string>  
#include <iostream>
#include <chrono>

#include "rocksdb/db.h"  

#pragma comment(lib,"Rpcrt4.lib")

#define TEST_FREQUENCY	(10000)

char* randomstr()
{
	static char buf[1024];
	int len = rand() % 768 + 255;
	for (int i = 0; i < len; ++i) {
		buf[i] = 'A' + rand() % 26;
	}
	buf[len] = '\0';
	return buf;
}

int main() 
{
	rocksdb::DB* db;
	rocksdb::Options options;
	options.create_if_missing = true;

	// 打开数据库
	rocksdb::Status status = rocksdb::DB::Open(options, "./testdb", &db);
	assert(status.ok());

	srand(2017);
	std::string k[TEST_FREQUENCY];
	for (int i = 0; i < TEST_FREQUENCY; ++i) {
		k[i] = (randomstr());
	}
	std::string v("壹贰叁肆伍陆柒捌玖拾");
	v.append(v).append(v).append(v).append(v).append(v);

	// 测试添加
	{
		auto start = std::chrono::system_clock::now();
		for (int i = 0; i < TEST_FREQUENCY; ++i) {
			status = db->Put(rocksdb::WriteOptions(), k[i], v);
			assert(status.ok());
		}
		auto end = std::chrono::system_clock::now();
		auto duration = std::chrono::duration_cast<std::chrono::microseconds>(end - start);

		std::cout << TEST_FREQUENCY <<"次添加耗时: "
			<< double(duration.count()) * std::chrono::microseconds::period::num / std::chrono::microseconds::period::den
			<< "秒" << std::endl;
	}
	// 测试获取
	{
		auto start = std::chrono::system_clock::now();
		std::string v2[TEST_FREQUENCY];
		for (int i = 0; i < TEST_FREQUENCY; ++i) {
			status = db->Get(rocksdb::ReadOptions(), k[i], &v2[i]);
			assert(status.ok());
		}
		auto end = std::chrono::system_clock::now();
		auto duration = std::chrono::duration_cast<std::chrono::microseconds>(end - start);

		std::cout << TEST_FREQUENCY <<"次获取耗时: "
			<< double(duration.count()) * std::chrono::microseconds::period::num / std::chrono::microseconds::period::den
			<< "秒" << std::endl;
		// 验证获取结果是否正确
		std::string ss;
		for (int i = 0; i < TEST_FREQUENCY; ++i) {
			if (v2[i] != v) {
				std::cout << "第 " << i << " 个结果不正确" << std::endl;
				std::cout << v2[i] << std::endl;
			}
		}
	}
	// 测试修改
	{
		auto start = std::chrono::system_clock::now();
		v.append(v);
		for (int i = 0; i < TEST_FREQUENCY; ++i) {
			status = db->Put(rocksdb::WriteOptions(), k[i], v);
			assert(status.ok());
		}
		auto end = std::chrono::system_clock::now();
		auto duration = std::chrono::duration_cast<std::chrono::microseconds>(end - start);

		std::cout << TEST_FREQUENCY <<"次修改耗时: "
			<< double(duration.count()) * std::chrono::microseconds::period::num / std::chrono::microseconds::period::den
			<< "秒" << std::endl;
	}

	// 测试删除
	{
		auto start = std::chrono::system_clock::now();
		for (int i = 0; i < TEST_FREQUENCY; ++i) {
			status = db->Delete(rocksdb::WriteOptions(), k[i]);
			assert(status.ok());
		}
		auto end = std::chrono::system_clock::now();
		auto duration = std::chrono::duration_cast<std::chrono::microseconds>(end - start);

		std::cout << TEST_FREQUENCY <<"次删除耗时: "
			<< double(duration.count()) * std::chrono::microseconds::period::num / std::chrono::microseconds::period::den
			<< "秒" << std::endl;
	}
	delete db;
	return 0;
}
```
编译命令如下：
```bash
# Debug版本
cl rocksdb_test.cpp /I ..\..\libs\vs140-x64\rocksdb\include  ..\..\libs\vs140-x64\rocksdb\Debug\rocksdb.lib /MDd /EHsc \
/GF /TP /Zp8 /Od /Gy /W3 /D "OS_WIN" /D "WIN32" /D "_WINDOWS" /D "NDEBUG" /D "ROCKSDB_LITE" /D "_MBCS" /D "WIN64" /D "NOMINMAX" 

# Release版本
cl rocksdb_test.cpp /I ..\..\libs\vs140-x64\rocksdb\include  ..\..\libs\vs140-x64\rocksdb\Release\rocksdb.lib /MD /EHsc \
/GF /TP /Zp8 /O2 /Gy /W3 /D "OS_WIN" /D "WIN32" /D "_WINDOWS" /D "NDEBUG" /D "ROCKSDB_LITE" /D "_MBCS" /D "WIN64" /D "NOMINMAX" 
```
注意上面使用了`/Zp8`和`/TP` `/GF`等选项。**如果不设置`/Zp8(8字节对齐)`，在运行时候会抛出异常。因为VS编译64位程序的时候，默认就是8字节对齐，而我没有测试32位版本的，所以这里可能不是关键。**
## 后话
Linux下的编译都比较简单，包括`LevelDB`等，都只需要直接`make`即可。Linux下编译的时候可能有某些依赖需要安装一下，包括`gflags`、`jemalloc`、`libsnappy`、`libbz2`、`liblz4`等。