---
layout:  post
title:  "外网IP监测上报程序(使用Poco库的SMTPClientSession发送邮件)"
date:  2018-09-04
categories:  编程
tags:  编程
comments: 1
---
[TOC]
[博客园原文地址 http://www.cnblogs.com/oloroso/archive/2018/09/04/9583854.html](http://www.cnblogs.com/oloroso/archive/2018/09/04/9583854.html)



# IPReport
代码地址[https://gitee.com/solym/IPReport](https://gitee.com/solym/IPReport)
## 项目介绍
外网IP变动自动上报工具。
写这个工具的目的是为了监测一台服务器的外网IP的变动情况。之前办理的电信宽带是有外网IP的，因此把一台小服务器上的ut管理端口开放出来，以便随时都能添加下载任务。
但是这个外网IP不是固定的，大概每周都会变动一次，变动的时间不固定，所以写了个程序来检测它，改变的时候主动发送信息给我。

## 编译说明
程序依赖于[`Poco`](https://pocoproject.org/)库，需要自己准备。
因为我的服务器装的是`ArchLinux`，所以直接使用`pacman -Syu poco`安装就好了。
如果是`Windows`可以直接使用`vcpkg`来编译安装`poco`库。

**linux**下直接使用`make`编译即可。

## 安装使用说明

无需安装，编译之后可以直接运行。
**linux**下可以使用`nohup ./IPReport 2>&1 1>/dev/null &`来放在后台运行。
**Windows**下你可以在VS工程`属性页->链接器->系统`里面选择子系统为`窗口(SUBSYSTEM:WINDOWS)`来生成一个无窗口的窗口应用，就可以无控制台运行了。

## 获取外网IP方式
外网IP可以通过访问淘宝的[https://www.taobao.com/help/getip.php](https://www.taobao.com/help/getip.php)获取。

## 邮件发送关键代码
```cpp
		// 发送邮件通知
		// 发送的消息内容
		Poco::Net::MailMessage message;
		message.setSubject("外网IP地址改变通知");
		message.setDate(Poco::DateTime().timestamp());
		message.addContent(new StringPartSource(lastResult,"text/plain"));
		message.setSender(mailuser);
		message.addRecipient(MailRecipient(MailRecipient::PRIMARY_RECIPIENT,mailrecipient));
		// 开始发送邮件
		// 第一个是非SSL连接的，第二个是SSL连接的
		//Poco::Net::SMTPClientSession smtpSession(mailhost);
		Poco::Net::SecureSMTPClientSession smtpSession(mailhost);
		smtpSession.open();
		// 下面两行是SSL连接必须的
		smtpSession.login();
		smtpSession.startTLS();
		// 登录邮件服务
		smtpSession.login(Poco::Net::SMTPClientSession::LoginMethod::AUTH_LOGIN, mailuser, mailpasswd);
		// 发送出邮件内容
	        smtpSession.sendMessage(message);
		// 发送后关闭会话
		smtpSession.close();
```