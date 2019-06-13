---
layout:  post
title:  "给evpp支持https"
date:  2018-11-07
categories:  其它
tags:  其它
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2018/11/07/9923818.html](http://www.cnblogs.com/oloroso/archive/2018/11/07/9923818.html)
# 

**参考**
- [Libevent：8Bufferevents高级主题](https://blog.csdn.net/gqtcgq/article/details/43374387)
- [ssl_ctx_set_options(3) - Linux man page](https://linux.die.net/man/3/ssl_ctx_set_options)
- [openssl编程之服务端](https://blog.csdn.net/fly2010love/article/details/46458963)
- [SSL编程- 简单函数介绍](https://blog.csdn.net/xs574924427/article/details/17240793)
- [SSL编程简介](https://segmentfault.com/a/1190000012065369)
- [libevent添加ssl加密功能](https://blog.csdn.net/fly2010love/article/details/46459485)
- [windows c/c++使用libevent库编写http/https服务端](https://blog.csdn.net/hedubao135792468/article/details/82841713)
- [基于Libevent的HTTP Server](https://www.cnblogs.com/luxiaoxun/p/3704573.html)
- [https://github.com/ppelleti/https-example](https://github.com/ppelleti/https-example)

## 生成证书
```bash
# 创建私钥
openssl genrsa -out rsa_private.key 1024
# 生成公钥
openssl rsa -in rsa_private.key -pubout -out rsa_public.key
# 生成crs文件
openssl req -new -out cert.csr -key rsa_private.key
# 生成pem证书
openssl x509 -req -in cert.csr -out server.pem -outform pem -signkey rsa_private.key -days 3650
```