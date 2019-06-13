---
layout:  post
title:  "LevelDB和ForestDB简单性能测试(含代码)"
date:  2017-01-19
categories:  其它
tags:  其它
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2017/01/19/6306352.html](http://www.cnblogs.com/oloroso/archive/2017/01/19/6306352.html)
# 测试环境简单说明

## Windows下测试
硬件环境如下：
    处理器：Intel(R) Core(TM) i5-4460 CPU @ 3.20GHz
    内  存：8GB
    硬  盘：希捷 ST1000DM003
    操作系统：Windows 10 企业版
编译说明：
    两个都是使用VS2015编译的64位Release版本。运行时库采用动态多线程版本(MD)

## Linux下测试
硬件环境如下：
    处理器：Intel(R) Core(TM) i7-4500U CPU @ 1.80GHz
    内  存：8GB
    硬  盘：金士顿64G SSD
    操作系统：ArchLinux (Linux version 4.8.13-1-ARCH)
编译说明：
    两个都是使用Gcc 6.2.1编译的x64版本，使用`-O2`参数优化。

# 测试结果

对`LevelDB`和`ForestDB`进行简单的性能测试。
两个都在`单线程`下进行`10000`次的`增删查改`测试，共测试5次。（这里测试的次数有点少，应该测试十万次以上的）
测试的时候可以发现（设置断点），Forest每次操作都将数据缓存在内存了，内存占用比较大。而LevelDB在添加的时候并没有缓存，但是在数据获取和修改的时候内存会变大。
总体上LevelDB占用内存小一点，但是linux下速度不及ForestDB(非常接近)。易用程度上，LevelB简单得多。磁盘占用的情况的话，Forest对磁盘使用比较少，这10000条数据占了13MB左右，而LevelDB则占了120MB左右。

## Windows下测试结果
测试结果平均值对比直方图：
![测试结果对比直方图](http://images2015.cnblogs.com/blog/693958/201701/693958-20170119133938890-1813945194.png)

LevelDB 测试结果截图
![LevelDB 测试结果截图](http://images2015.cnblogs.com/blog/693958/201701/693958-20170119134052015-1839390104.png)

ForestDB 测试结果截图
![ForestDB 测试结果截图](http://images2015.cnblogs.com/blog/693958/201701/693958-20170119134026109-2099387576.png)

## Linux下测试结果

测试结果平均值对比直方图：
![测试结果对比直方图](http://images2015.cnblogs.com/blog/693958/201701/693958-20170119135355843-2084456732.png)

LevelDB 测试结果截图
![LevelDB 测试结果截图](http://images2015.cnblogs.com/blog/693958/201701/693958-20170119135420015-758339593.png)

ForestDB 测试结果截图
![ForestDB 测试结果截图](http://images2015.cnblogs.com/blog/693958/201701/693958-20170119135437812-1984853997.png)

# 测试代码

## LevelDB测试代码
```cpp
#include <cassert>  
#include <string>  
#include <iostream>
#include <chrono>

#include "leveldb/db.h"  

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
	leveldb::DB* db;
	leveldb::Options options;
	options.create_if_missing = true;

	// 打开数据库
	leveldb::Status status = leveldb::DB::Open(options, "./testdb", &db);
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
			status = db->Put(leveldb::WriteOptions(), k[i], v);
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
			status = db->Get(leveldb::ReadOptions(), k[i], &v2[i]);
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
			status = db->Put(leveldb::WriteOptions(), k[i], v);
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
			status = db->Delete(leveldb::WriteOptions(), k[i]);
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
## Forest 测试代码
```cpp
#include <cassert>  
#include <string>  
#include <iostream>
#include <chrono>

#include "libforestdb/forestdb.h"

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
	fdb_file_handle* fdbFileHandle = nullptr;
	fdb_kvs_handle* fdbKvsHandle = nullptr;
	fdb_status status;

	// 初始化ForestDB
	// 1、文件配置设置配置
	fdb_config fileConfig = fdb_get_default_config();
	{// WAL阈值4K
		fileConfig.wal_threshold = 4096;
		// 缓存大小64MB
		fileConfig.buffercache_size = 64 * 1024 * 1024;
		// 设置使用默认的kvs
		fileConfig.multi_kv_instances = false;
		// 关闭循环块复用
		fileConfig.block_reusing_threshold = 100;
		// 使用序列树
		fileConfig.seqtree_opt = FDB_SEQTREE_USE;
	}
	// 2、使用设置的配置进行初始化
	status = fdb_init(&fileConfig);
	assert(status == FDB_RESULT_SUCCESS);

	// 打开数据库
	status = fdb_open(&fdbFileHandle, "./testdb", &fileConfig);
	assert(status == FDB_RESULT_SUCCESS);
	// 打开kvs
	fdb_kvs_config kvsConfig = fdb_get_default_kvs_config();
	status = fdb_kvs_open_default(fdbFileHandle, &fdbKvsHandle, &kvsConfig);
	assert(status == FDB_RESULT_SUCCESS);


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
			status = fdb_set_kv(fdbKvsHandle, k[i].data(), k[i].size(), v.data(), v.size());
			assert(status == FDB_RESULT_SUCCESS);
		}
		// 提交操作到磁盘(这里必须commit才能实际写入到磁盘)
		fdb_commit(fdbFileHandle, FDB_COMMIT_NORMAL);

		auto end = std::chrono::system_clock::now();
		auto duration = std::chrono::duration_cast<std::chrono::microseconds>(end - start);
	
		std::cout << TEST_FREQUENCY <<"次添加耗时: "
			<< double(duration.count()) * std::chrono::microseconds::period::num / std::chrono::microseconds::period::den
			<< "秒" << std::endl;
	}
	// 测试获取
	{
		auto start = std::chrono::system_clock::now();
		void* v2[TEST_FREQUENCY]; size_t v2len[TEST_FREQUENCY];
		for (int i = 0; i < TEST_FREQUENCY; ++i) {
			status = fdb_get_kv(fdbKvsHandle, k[i].data(), k[i].size(), &v2[i], &v2len[i]);
			assert(status == FDB_RESULT_SUCCESS);
		}
		auto end = std::chrono::system_clock::now();
		auto duration = std::chrono::duration_cast<std::chrono::microseconds>(end - start);

		std::cout << TEST_FREQUENCY <<"次获取耗时: "
			<< double(duration.count()) * std::chrono::microseconds::period::num / std::chrono::microseconds::period::den
			<< "秒" << std::endl;
		// 验证获取结果是否正确
		std::string ss;
		for (int i = 0; i < TEST_FREQUENCY; ++i) {
			ss.assign((const char*)v2[i], v2len[i]);
			if (ss != v) {
				std::cout << "第 " << i << " 个结果不正确" << std::endl;
				std::cout << ss << std::endl;
			}
			free(v2[i]);
		}
	}
	// 测试修改
	{
		auto start = std::chrono::system_clock::now();
		v.append(v);
		for (int i = 0; i < TEST_FREQUENCY; ++i) {
			status = fdb_set_kv(fdbKvsHandle, k[i].data(), k[i].size(), v.data(), v.size());
			assert(status == FDB_RESULT_SUCCESS);
		}
		// 提交操作到磁盘(这里必须commit才能实际写入到磁盘)
		fdb_commit(fdbFileHandle, FDB_COMMIT_NORMAL);

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
			status = fdb_del_kv(fdbKvsHandle, k[i].data(), k[i].size());
			assert(status == FDB_RESULT_SUCCESS);
		}
		// 提交操作到磁盘(这里必须commit才能实际写入到磁盘)
		fdb_commit(fdbFileHandle, FDB_COMMIT_NORMAL);

		auto end = std::chrono::system_clock::now();
		auto duration = std::chrono::duration_cast<std::chrono::microseconds>(end - start);

		std::cout << TEST_FREQUENCY <<"次删除耗时: "
			<< double(duration.count()) * std::chrono::microseconds::period::num / std::chrono::microseconds::period::den
			<< "秒" << std::endl;
	}
	
	// 关闭数据库
	status = fdb_kvs_close(fdbKvsHandle);
	assert(status == FDB_RESULT_SUCCESS);
	status = fdb_close(fdbFileHandle);
	assert(status == FDB_RESULT_SUCCESS);
	status = fdb_shutdown();
	assert(status == FDB_RESULT_SUCCESS);
	return 0;
}
```