---
layout:  post
title:  "XML文件生成C++代码(基于rapidxml)"
date:  2018-03-09
categories:  编程
tags:  编程
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2018/03/09/8532624.html](http://www.cnblogs.com/oloroso/archive/2018/03/09/8532624.html)
## 简述

与[XML文件生成C++代码(基于pugixml)](http://www.cnblogs.com/oloroso/p/8515466.html)中的功能一致，只是这里改用的rapidxml来实现。就不多说了，直接放代码。

## 代码

```cpp
#include "rapidxml-1.13/rapidxml.hpp"
#include "rapidxml-1.13/rapidxml_utils.hpp"
#include "rapidxml-1.13/rapidxml_iterators.hpp"
#include "rapidxml-1.13/rapidxml_print.hpp"


#include <algorithm>
#include <cstdio>

// XML节点名称中可以包含C++变量名不支持的字符
// 需要将其规范化，使之能够正常作为C++变量名
std::string normalVarName(std::string s)
{
    //    XML 元素必须遵循以下命名规则：
    //        名称可以含字母、数字以及其他的字符
    //        名称不能以数字或者标点符号开始
    //        名称不能以字符 “xml”（或者 XML、Xml）开始
    //        名称不能包含空格
    //    可使用任何名称，没有保留的字词。
    // 如果只需要C++编译通过，完全可以将s转换为base16表示的字符串即可
    // 这里我只对 : . , ; 做一下处理，其它的就先不管了
    for(std::size_t pos = s.find_first_of(":.,;");
        pos != std::string::npos; pos = s.find_first_of(":.,;",pos+1)){
        s[pos] = '_';
    }
    return s;
}

// 对于节点的值，它有可能包含需要转义的字符，需要进行转义(未考虑C++原始字面常量)
std::string toEscape(const std::string& s)
{
    std::string s2;
    s2.reserve(s.size());
    for(std::size_t i=0;i<s.size();++i){
        switch (s[i]) {
        case '"':
        case '\\':
            s2.push_back('\\');
            break;
        default:
            break;
        }
        s2.push_back(s[i]);
    }
    return s2;
}

bool TraversalXml(rapidxml::xml_node<char>& node,int level = 0)
{
    std::string s(level*4,32);
    if(level == 0){ /*表明是root节点*/
        puts("rapidxml::xml_document<char> xmldocObject;\n"
             "rapidxml::xml_document<char>* xmldoc = &xmldocObject;\n"
             "{\n"
             "    rapidxml::xml_node<char>* decl = \n"
             "        xmldoc->allocate_node(rapidxml::node_pi,\n"
             "             xmldoc->allocate_string(\"xml version='1.0' encoding='utf-8'\"));\n"
             "    xmldoc->append_node(decl);\n"
             "}");
        puts("{");
    }
    // 获取节点名称
    std::string nodeName(node.name(),node.name_size());
    std::string nodeValue(node.value(),node.value_size());
    std::string parentName;
    if(node.parent() == NULL || node.parent()->name_size() == 0){
        parentName = "xmldoc";
    } else {
        parentName = std::string(node.parent()->name(),node.parent()->name_size());;
    }

    // pugixml这里有点问题，不知道是不是bug(我对XML规范和pugixml都不是很熟)
    // 一个节点A没有子节点的时候，它也能获取到它的一个子节点，这个子节点的name是
    // 空的,value与节点A相同，其它的属性信息也是没有的。
    // 所以在这里一些地方写的会比较别扭
    // 这里使用rapidxml的时候也有这个问题，所以我觉得可能是对于没有子节点的的节点
    // 它的值就是它的子节点吧。这只是我的猜想，并未查阅相关规范。


    if(!nodeName.empty()){
        printf("%s{/*add %s*/\n",s.c_str(),nodeName.c_str());

        // 输出添加子节点代码
        if(nodeValue.empty()){
            printf("%s    rapidxml::xml_node<char>* %s = xmldoc->allocate_node(\n"
                   "%s        rapidxml::node_element,\"%s\",NULL);\n",
                   s.c_str(),normalVarName(nodeName).c_str(),
                   s.c_str(),nodeName.c_str());
        }else{
            printf("%s    rapidxml::xml_node<char>* %s = xmldoc->allocate_node(\n"
                   "%s        rapidxml::node_element,\"%s\",\"%s\");\n",
                   s.c_str(),normalVarName(nodeName).c_str(),
                   s.c_str(),nodeName.c_str(),toEscape(nodeValue).c_str());
        }

        // 输出添加子节点到父节点代码
        printf("%s    %s->append_node(%s);\n",
               s.c_str(),normalVarName(parentName).c_str(),
               normalVarName(nodeName).c_str());

        // 逐个输出属性信息添加代码
        for(rapidxml::xml_attribute<char>* attr = node.first_attribute();
            attr != NULL; attr = attr->next_attribute()){
            // 输出属性信息添加代码
            printf("%s    %s->append_attribute(\n"
                   "%s        xmldoc->allocate_attribute(\"%s\",\"%s\"));\n",
                   s.c_str(),normalVarName(nodeName).c_str(),
                   s.c_str(),attr->name(),attr->value());
        }
    }
    // 遍历子节点
    for(rapidxml::xml_node<char>* subnode = node.first_node();
        subnode != NULL;subnode = subnode->next_sibling()){
        TraversalXml(*subnode,level+1);
    }
    // 与前面匹配
    if(!nodeName.empty()){
        printf("%s}/*end %s*/\n\n",s.c_str(),nodeName.c_str());
    }
    // 与前面匹配
    if(level == 0){
        puts("}");
    }
    return true;
}


int main(int argc,char** argv)
{
    if(argc != 2){
        puts("Usage:....");
        return 0;
    }
    try{
        rapidxml::file<char> xmlfile(argv[1]);
        rapidxml::xml_document<char> doc;
        doc.parse<0>(xmlfile.data());
        TraversalXml(doc);
    }
    catch(std::exception& e){
        puts(e.what());
    }

    return 0;
}

```