---
layout:  post
title:  "DLib Http Server程序示例"
date:  2017-03-27
categories:  编程
tags:  编程
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2017/03/27/6627003.html](http://www.cnblogs.com/oloroso/archive/2017/03/27/6627003.html)
```cpp
/*

    这个示例是一个使用了Dlib C++ 库的server组件的HTTP扩展

    它创建一个始终以简单的HTML表单为响应的服务器。
    要查看这个页面，你应该访问 http://localhost:5000

*/

#include <iostream>
#include <sstream>
#include <string>
#include <dlib/server.h>

using namespace dlib;
using namespace std;

class web_server : public server_http
{
    const std::string on_request (
        const incoming_things& incoming,
        outgoing_things& outgoing
    )
    {
        ostringstream sout;
        // 我们将发回一个包含HTML表单的页面，其中包含两个文本输入字段。一个字段是name(还有一个是pass)
        // HTML表单使用post方法，也可以使用get方法(只需将method='post'改为method='get').

        sout <<" <html><meta charset=\"utf-8\"><body> "
            << "<form action='/form_handler' method='post'> "
            << "用户名称: <input name='user' type='text'><br>  "
            << "用户密码: <input name='pass' type='text'> <input type='submit'> "
            << " </form>";

        // 回写这个请求传入的一些信息，以便它们显示在生成的网页上
        sout << "<br>  path = "         << incoming.path << endl;
        sout << "<br>  request_type = " << incoming.request_type << endl;
        sout << "<br>  content_type = " << incoming.content_type << endl;
        sout << "<br>  protocol = "     << incoming.protocol << endl;
        sout << "<br>  foreign_ip = "   << incoming.foreign_ip << endl;
        sout << "<br>  foreign_port = " << incoming.foreign_port << endl;
        sout << "<br>  local_ip = "     << incoming.local_ip << endl;
        sout << "<br>  local_port = "   << incoming.local_port << endl;
        sout << "<br>  body = \""       << incoming.body << "\"" << endl;


        // 如果此请求是用户提交表单的结果，则回显提交
        if (incoming.path == "/form_handler")
        {
            sout << "<h2> 来自 query string 的信息 </h2>" << endl;
            sout << "<br>  user = " << incoming.queries["user"] << endl;
            sout << "<br>  pass = " << incoming.queries["pass"] << endl;

            // 将这些表单提交作为Cookie保存
            outgoing.cookies["user"] = incoming.queries["user"];
            outgoing.cookies["pass"] = incoming.queries["pass"];
        }


        // 将所有Cookie回传到客户端浏览器
        sout << "<h2>客户web浏览器发送给服务器的 Cookies</h2>";
        for ( key_value_map::const_iterator ci = incoming.cookies.begin(); ci != incoming.cookies.end(); ++ci )
        {
            sout << "<br/>" << ci->first << " = " << ci->second << endl;
        }

        sout << "<br/><br/>";

        sout << "<h2>客户web浏览器发送给服务器的 HTTP Headers</h2>";
        // 回显从客户端web浏览器接收的所有HTTP headers
        for ( key_value_map_ci::const_iterator ci = incoming.headers.begin(); ci != incoming.headers.end(); ++ci )
        {
            sout << "<br/>" << ci->first << ": " << ci->second << endl;
        }

        sout << "</body> </html>";

        return sout.str();
    }

};

int main()
{
    try
    {
        // 创建一个 web server 实例对象
        web_server our_web_server;

        // 监听5000端口
        our_web_server.set_listening_port(5000);
        // 告诉服务器开始接受连接。
        our_web_server.start_async();

        cout << "请按Enter(回车)键去结束程序" << endl;
        cin.get();
    }
    catch (exception& e)
    {
        // 上面出错会抛出异常
        // 输出异常消息
        cout << e.what() << endl;
    }
}
```