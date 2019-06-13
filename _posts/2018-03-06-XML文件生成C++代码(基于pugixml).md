---
layout:  post
title:  "XML文件生成C++代码(基于pugixml)"
date:  2018-03-06
categories:  编程
tags:  编程
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2018/03/06/8515466.html](http://www.cnblogs.com/oloroso/archive/2018/03/06/8515466.html)
## 简述
在一个项目中需要用到XML的解析和生成，知乎上有人推荐`rapidxml`和`pugixml`等库。`RapidXML`一看库还比较大，就先研究一下`pugixml`了。
因为对解析XML的需求不大（都是一些很小的XML文本），但是对生成XML有较大的需求，且这些XML文本都很大，所以先写了一个根据XML文件生成对应的C++代码的项目。

对XML的规范并不熟悉，所以这里只做了读取`节点属性`和`节点值`生成对应代码的操作，对于其它的部分，我也不知道还有哪里需要做的。
这里没有考虑非UTF-8编码和宽字符的情况，我这里暂时用不上，需要的可以根据自己的情况改。

**不知道是不是`pugixml`的bug，对于一个没有子节点的节点，也能获取其一个无名子节点**

## 示例
假设有如下内容的XML文件

```xml
<?xml version="1.0" encoding="utf-8" ?>
<EnvelopeN xsi:type='typens:EnvelopeN' xmlns:xsi='http://www.w3.org/2001/XMLSchema-instance' xmlns:xs='http://www.w3.org/2001/XMLSchema' xmlns:typens='http://www.esri.com/schemas/ArcGIS/10.1'>
	<XMin>-180</XMin>
	<YMin>-90</YMin>
	<XMax>180</XMax>
	<YMax>90</YMax>
	<SpatialReference xsi:type='typens:GeographicCoordinateSystem'>
		<WKT>GEOGCS[&quot;GCS_WGS_1984&quot;,DATUM[&quot;D_WGS_1984&quot;,SPHEROID[&quot;WGS_1984&quot;,6378137.0,298.257223563]],PRIMEM[&quot;Greenwich&quot;,0.0],UNIT[&quot;Degree&quot;,0.0174532925199433],AUTHORITY[&quot;EPSG&quot;,4326]]</WKT>
		<XOrigin>-400</XOrigin>
		<YOrigin>-400</YOrigin>
		<XYScale>11258999068426.238</XYScale>
		<ZOrigin>-100000</ZOrigin>
		<ZScale>10000</ZScale>
		<MOrigin>-100000</MOrigin>
		<MScale>10000</MScale>
		<XYTolerance>8.983152841195215e-009</XYTolerance>
		<ZTolerance>0.001</ZTolerance>
		<MTolerance>0.001</MTolerance>
		<HighPrecision>true</HighPrecision>
		<LeftLongitude>-180</LeftLongitude>
		<WKID>4326</WKID>
		<LatestWKID>4326</LatestWKID>
	</SpatialReference>
</EnvelopeN>
```

那么生成的代码如下

```cpp
pugi::xml_document xmldoc;
{
    pugi::xml_node decl = xmldoc.append_child(pugi::node_declaration);
    decl.append_attribute("version") = "1.0";
    decl.append_attribute("encoding") = "utf-8";
}
{
    {/*add EnvelopeN*/
        pugi::xml_node EnvelopeN = xmldoc.append_child("EnvelopeN");
        EnvelopeN.append_attribute("xsi:type")="typens:EnvelopeN";
        EnvelopeN.append_attribute("xmlns:xsi")="http://www.w3.org/2001/XMLSchema-instance";
        EnvelopeN.append_attribute("xmlns:xs")="http://www.w3.org/2001/XMLSchema";
        EnvelopeN.append_attribute("xmlns:typens")="http://www.esri.com/schemas/ArcGIS/10.1";
        {/*add XMin*/
            pugi::xml_node XMin = EnvelopeN.append_child("XMin");
            XMin.text().set("-180");
        }/*end XMin*/

        {/*add YMin*/
            pugi::xml_node YMin = EnvelopeN.append_child("YMin");
            YMin.text().set("-90");
        }/*end YMin*/

        {/*add XMax*/
            pugi::xml_node XMax = EnvelopeN.append_child("XMax");
            XMax.text().set("180");
        }/*end XMax*/

        {/*add YMax*/
            pugi::xml_node YMax = EnvelopeN.append_child("YMax");
            YMax.text().set("90");
        }/*end YMax*/

        {/*add SpatialReference*/
            pugi::xml_node SpatialReference = EnvelopeN.append_child("SpatialReference");
            SpatialReference.append_attribute("xsi:type")="typens:GeographicCoordinateSystem";
            {/*add WKT*/
                pugi::xml_node WKT = SpatialReference.append_child("WKT");
                WKT.text().set("GEOGCS[\"GCS_WGS_1984\",DATUM[\"D_WGS_1984\",SPHEROID[\"WGS_1984\",6378137.0,298.257223563]],PRIMEM[\"Greenwich\",0.0],UNIT[\"Degree\",0.0174532925199433],AUTHORITY[\"EPSG\",4326]]");
            }/*end WKT*/

            {/*add XOrigin*/
                pugi::xml_node XOrigin = SpatialReference.append_child("XOrigin");
                XOrigin.text().set("-400");
            }/*end XOrigin*/

            {/*add YOrigin*/
                pugi::xml_node YOrigin = SpatialReference.append_child("YOrigin");
                YOrigin.text().set("-400");
            }/*end YOrigin*/

            {/*add XYScale*/
                pugi::xml_node XYScale = SpatialReference.append_child("XYScale");
                XYScale.text().set("11258999068426.238");
            }/*end XYScale*/

            {/*add ZOrigin*/
                pugi::xml_node ZOrigin = SpatialReference.append_child("ZOrigin");
                ZOrigin.text().set("-100000");
            }/*end ZOrigin*/

            {/*add ZScale*/
                pugi::xml_node ZScale = SpatialReference.append_child("ZScale");
                ZScale.text().set("10000");
            }/*end ZScale*/

            {/*add MOrigin*/
                pugi::xml_node MOrigin = SpatialReference.append_child("MOrigin");
                MOrigin.text().set("-100000");
            }/*end MOrigin*/

            {/*add MScale*/
                pugi::xml_node MScale = SpatialReference.append_child("MScale");
                MScale.text().set("10000");
            }/*end MScale*/

            {/*add XYTolerance*/
                pugi::xml_node XYTolerance = SpatialReference.append_child("XYTolerance");
                XYTolerance.text().set("8.983152841195215e-009");
            }/*end XYTolerance*/

            {/*add ZTolerance*/
                pugi::xml_node ZTolerance = SpatialReference.append_child("ZTolerance");
                ZTolerance.text().set("0.001");
            }/*end ZTolerance*/

            {/*add MTolerance*/
                pugi::xml_node MTolerance = SpatialReference.append_child("MTolerance");
                MTolerance.text().set("0.001");
            }/*end MTolerance*/

            {/*add HighPrecision*/
                pugi::xml_node HighPrecision = SpatialReference.append_child("HighPrecision");
                HighPrecision.text().set("true");
            }/*end HighPrecision*/

            {/*add LeftLongitude*/
                pugi::xml_node LeftLongitude = SpatialReference.append_child("LeftLongitude");
                LeftLongitude.text().set("-180");
            }/*end LeftLongitude*/

            {/*add WKID*/
                pugi::xml_node WKID = SpatialReference.append_child("WKID");
                WKID.text().set("4326");
            }/*end WKID*/

            {/*add LatestWKID*/
                pugi::xml_node LatestWKID = SpatialReference.append_child("LatestWKID");
                LatestWKID.text().set("4326");
            }/*end LatestWKID*/

        }/*end SpatialReference*/

    }/*end EnvelopeN*/

}
```

## 关键代码
这里贴出关键部分代码
```cpp
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

bool TraversalXml(pugi::xml_node& node,int level = 0)
{
    if(!node){return true;}
    std::string s(level*4,32);

    if(node.root() == node){
        puts("pugi::xml_document xmldoc;\n"
             "{\n"
             "    pugi::xml_node decl = xmldoc.append_child(pugi::node_declaration);\n"
             "    decl.append_attribute(\"version\") = \"1.0\";\n"
             "    decl.append_attribute(\"encoding\") = \"utf-8\";\n"
             "}");
    }
    if(node == node.root()){
        puts("{");
    }
    // 获取节点名称
    std::string nodeName = node.name();
    // pugixml这里有点问题，不知道是不是bug(我对XML规范和pugixml都不是很熟)
    // 一个节点A没有子节点的时候，它也能获取到它的一个子节点，这个子节点的name是
    // 空的,value与节点A相同，其它的属性信息也是没有的。
    // 所以在这里一些地方写的会比较别扭

    if(!nodeName.empty()){
        printf("%s{/*add %s*/\n",s.c_str(),nodeName.c_str());

        // 获取父节点名称
        std::string parentName = node.parent() == node.root()?
                    "xmldoc":node.parent().name();
        // 输出添加子节点代码
        printf("%s    pugi::xml_node %s = %s.append_child(\"%s\");\n",
               s.c_str(),normalVarName(nodeName).c_str(),
               normalVarName(parentName).c_str(),nodeName.c_str());

    // 逐个输出属性信息添加代码
    for(pugi::xml_node::attribute_iterator iter = node.attributes_begin();
        iter != node.attributes_end(); ++iter){
        // 输出属性信息添加代码
        printf("%s    %s.append_attribute(\"%s\")=\"%s\";\n",
               s.c_str(),normalVarName(nodeName).c_str(),
               iter->name(),iter->value());
    }
    // 如果节点的值为空，则不输出节点的值了(例如有子节点的情况下，节点的值为空)
    if(!node.text().empty()){
        // 输出设置节点值代码
        printf("%s    %s.text().set(\"%s\");\n",
               s.c_str(),normalVarName(nodeName).c_str(),
               toEscape(node.text().as_string()).c_str());
    }
    }
    // 遍历子节点
    for(pugi::xml_node subnode = node.first_child();
        subnode;subnode = subnode.next_sibling()){
        TraversalXml(subnode,level+1);
    }
    // 与前面匹配
    if(!nodeName.empty()){
        printf("%s}/*end %s*/\n\n",s.c_str(),nodeName.c_str());
    }
    // 与前面匹配
    if(node == node.root()){
        puts("}");
    }
    return true;
}
```

完整代码可在这里下载
https://files.cnblogs.com/files/oloroso/XmlGenCppCode.zip