---
layout:  post
title:  "使用Træfɪk(traefik)来加速Qt在线更新"
date:  2018-10-02
categories:  其它
tags:  其它
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2018/10/02/9736845.html](http://www.cnblogs.com/oloroso/archive/2018/10/02/9736845.html)
## 简述

在使用Qt的MaintenanceTool程序进行在线更新的时候遇到一个问题，就是访问`download.qt.io`实在太慢了，老是失败。所以想使用国内的镜像站来进行更新。
使用Qt的镜像站方法也很简单，下载`Update.xml`和`Update_orig.xml`回来，然后修改里面的`url`即可，这个网上有很多教程。
但是这个方法不是很好用，还需要自己手动把一些元数据文件下载回来。

最近在研究[traefik](https://traefik.io/)，所以就用它做了一个简单的代理转发，来达到加速的目的。

## traefik 简介

Træfɪk 是一个为了让部署微服务更加便捷而诞生的现代HTTP反向代理、负载均衡工具。 它支持多种后台 (Docker, Swarm, Kubernetes, Marathon, Mesos, Consul, Etcd, Zookeeper, BoltDB, Rest API, file…) 来自动化、动态的应用它的配置文件设置。

![](https://img2018.cnblogs.com/blog/693958/201810/693958-20181002124932193-800142650.jpg)

![](https://img2018.cnblogs.com/blog/693958/201810/693958-20181002125013209-1466371485.jpg)

关于traefik的介绍，网上资料不是很多，可以看它的官网和一个国内的网站
- 官网 https://traefik.io/
- 国内站 http://traefik.cn/

## 代理设置具体过程

1、下载traefik程序
这个可以去github上下载源码回来自己编，也可以直接下载编译好的文件。因为traefik是使用go语言编译的，所有的依赖都在一个程序里面，没有乱七八糟的依赖问题。
发布版本下载地址[https://github.com/containous/traefik/releases](https://github.com/containous/traefik/releases)

2、编写配置文件，添加前后端来配置代理。
我使用的是清华大学的镜像站，速度还比较快。地址:[https://mirrors.tuna.tsinghua.edu.cn/qt/](https://mirrors.tuna.tsinghua.edu.cn/qt/)

写好配置文件之后，直接运行起来即可
```
./traefik --c config.toml
```
配置文件如何写，可以看官网上的文档。需要中文的也可以看这里[http://docs.traefik.cn/basics](http://docs.traefik.cn/basics)
配置文件如下：
```ini
# 入口点
[entryPoints]
  # HTTP 入口点，只需要HTTP的就够了
  [entryPoints.http]
  address = ":80"    # 使用80端口，这样后面有用


# 管理界面监听端口
[web]
  address = ":8012"
  [web.statistics]
  ReccentError = 10

# 配置文件监测(有改变的时候无需重启服务程序，会自动更新)
[file]
filename = "./config.toml"
watch = true
# 后端服务器定义
  [backends]
    # 定义后端，这里我直接使用的tuna的名称
    [backends.tuna]
      # 设置最大连接数，其实可以不设置
      [backends.tuna.maxconn]
  	  amount = 10
  	  extractorfunc = "request.host"
          # 后端的服务器，可以添加多个
	  [backends.tuna.servers.server1]
	  url = "https://mirrors-i.tuna.tsinghua.edu.cn"  # 这里使用清华镜像站的URL
	  weight = 10

  # 前端转发规则定义  
  [frontends]
   # 定义一个前端，前端就是你访问treafik入口点的时候，用来确定如何转发的规则
    [frontends.tuna]
    # 这个前端转发到的后端
    backend = "tuna"
    passHostHeader = false # 这里不能为true，否则转发的时候会是一个不正常的重定向，导致服务器返回错误
      # 路由规则，也可以有多个
      [frontends.tuna.routes.test_1]
      rule = "AddPrefix:/qt"   # 添加前缀，也就是访问入口点的时候，给URL里面的路径添加前缀/qt再去访问后端
                                          # 比如访问 http://127.0.0.1/online/qtsdkrepository/windows_x86/android/
                                          # 那么就会转发到后端 https://mirrors-i.tuna.tsinghua.edu.cn/qt/online/qtsdkrepository/windows_x86/android/
```

3、修改hosts文件，把`download.qt.io`解析到`127.0.0.1。
![](https://img2018.cnblogs.com/blog/693958/201810/693958-20181002131204626-83152882.jpg)

4、做完上面几步，可以直接在浏览器访问`download.qt.io`，看看是否正常。
![](https://img2018.cnblogs.com/blog/693958/201810/693958-20181002131308809-921240112.jpg)

然后直接使用Qt安装目录下的MaintenanceTool程序进行升级更新即可。

![](https://img2018.cnblogs.com/blog/693958/201810/693958-20181002130927924-1639732715.jpg)

![](https://img2018.cnblogs.com/blog/693958/201810/693958-20181002130939976-1205239452.jpg)

**注意，使用中科大的源，因为文件不全，可能出现下面的问题，北交大的源不能用，它里面是空的，就一个假的。使用清华的源没问题。**
![](https://img2018.cnblogs.com/blog/693958/201810/693958-20181002131538152-1334519025.jpg)