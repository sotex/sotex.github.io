---
layout:  post
title:  "CString与UTF8互转代码"
date:  2017-12-25
categories:  其它
tags:  其它
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2017/12/25/8108477.html](http://www.cnblogs.com/oloroso/archive/2017/12/25/8108477.html)
这个代码网上很多，留在这里做个备份。
```cpp
static std::string ConvertCStringToUTF8( CString strValue )
{
	std::wstring wbuffer;
#ifdef _UNICODE
	wbuffer.assign( strValue.GetString(), strValue.GetLength() );
#else
	/*
	 * 转换ANSI到UNICODE
	 * 获取转换后长度
	 */
	int length = ::MultiByteToWideChar( CP_ACP, MB_ERR_INVALID_CHARS, (LPCTSTR) strValue, -1, NULL, 0 );
	wbuffer.resize( length );
	/* 转换 */
	MultiByteToWideChar( CP_ACP, 0, (LPCTSTR) strValue, -1, (LPWSTR) (wbuffer.data() ), wbuffer.length() );
#endif

	/* 获取转换后长度 */
	int length = WideCharToMultiByte( CP_UTF8, 0, wbuffer.data(), wbuffer.size(), NULL, 0, NULL, NULL );
	/* 获取转换后内容 */
	std::string buffer;
	buffer.resize( length );

	WideCharToMultiByte( CP_UTF8, 0, strValue, -1, (LPSTR) (buffer.data() ), length, NULL, NULL );
	return(buffer);
}


static CString ConvertUTF8ToCString( std::string utf8str )
{
	/* 预转换，得到所需空间的大小 */
	int nLen = ::MultiByteToWideChar( CP_UTF8, NULL,
					  utf8str.data(), utf8str.size(), NULL, 0 );
	/* 转换为Unicode */
	std::wstring wbuffer;
	wbuffer.resize( nLen );
	::MultiByteToWideChar( CP_UTF8, NULL, utf8str.data(), utf8str.size(),
			       (LPWSTR) (wbuffer.data() ), wbuffer.length() );

#ifdef UNICODE
	return(CString( wbuffer.data(), wbuffer.length() ) );
#else
	/*
	 * 转换为ANSI
	 * 得到转换后长度
	 */
	nLen = WideCharToMultiByte( CP_ACP, 0,
				    wbuffer.data(), wbuffer.length(), NULL, 0, NULL, NULL );

	std::string ansistr;
	ansistr.resize( nLen );

	/* 把unicode转成ansi */
	WideCharToMultiByte( CP_ACP, 0, (LPWSTR) (wbuffer.data() ), wbuffer.length(),
			     (LPSTR) (ansistr.data() ), ansistr.size(), NULL, NULL );
	return(CString( ansistr.data(), ansistr.length() ) );
#endif
}
```