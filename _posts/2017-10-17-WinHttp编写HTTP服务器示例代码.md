---
layout:  post
title:  "WinHttp编写HTTP服务器示例代码"
date:  2017-10-17
categories:  其它
tags:  其它
comments: 1
---

[TOC]
[博客园文章地址 http://www.cnblogs.com/oloroso/archive/2017/10/17/7684672.html](http://www.cnblogs.com/oloroso/archive/2017/10/17/7684672.html)
这是微软提供的示例程序，原文地址在此[https://msdn.microsoft.com/en-us/library/windows/desktop/aa364640(v=vs.85).aspx](https://msdn.microsoft.com/en-us/library/windows/desktop/aa364640(v=vs.85).aspx)

# HTTP Server示例程序

以下示例应用程序展示如何使用`HTTP Server API`处理HTTP请求任务。第一个示例中包含的`precomp.h`文件包含示例所需的所有头文件，如下：
```c++
#ifndef UNICODE
#define UNICODE
#endif

#ifndef _WIN32_WINNT
#define _WIN32_WINNT 0x0600
#endif

#ifndef WIN32_LEAN_AND_MEAN
#define WIN32_LEAN_AND_MEAN
#endif

#include <windows.h>
#include <http.h>
#include <stdio.h>

#pragma comment(lib, "httpapi.lib")
```

## Main and Preliminaries(main和准备工作)
```c++
#include "precomp.h"

//
// Macros.初始化HTTP响应体宏
//
#define INITIALIZE_HTTP_RESPONSE( resp, status, reason )    \
    do                                                      \
    {                                                       \
        RtlZeroMemory( (resp), sizeof(*(resp)) );           \
        (resp)->StatusCode = (status);                      \
        (resp)->pReason = (reason);                         \
        (resp)->ReasonLength = (USHORT) strlen(reason);     \
    } while (FALSE)

#define ADD_KNOWN_HEADER(Response, HeaderId, RawValue)               \
    do                                                               \
    {                                                                \
        (Response).Headers.KnownHeaders[(HeaderId)].pRawValue =      \
                                                          (RawValue);\
        (Response).Headers.KnownHeaders[(HeaderId)].RawValueLength = \
            (USHORT) strlen(RawValue);                               \
    } while(FALSE)

#define ALLOC_MEM(cb) HeapAlloc(GetProcessHeap(), 0, (cb))

#define FREE_MEM(ptr) HeapFree(GetProcessHeap(), 0, (ptr))

//
// Prototypes.原型
//
DWORD DoReceiveRequests(HANDLE hReqQueue);

DWORD
SendHttpResponse(
    IN HANDLE        hReqQueue,
    IN PHTTP_REQUEST pRequest,
    IN USHORT        StatusCode,
    IN PSTR          pReason,
    IN PSTR          pEntity
    );

DWORD
SendHttpPostResponse(
    IN HANDLE        hReqQueue,
    IN PHTTP_REQUEST pRequest
    );


/*******************************************************************++

函数说明:
    main函数

参数:
    argc - 命令行参数个数.
    argv - 命令行参数.

返回值:
    Success/Failure

--*******************************************************************/
int __cdecl wmain(
        int argc, 
        wchar_t * argv[]
        )
{
    ULONG           retCode;
    HANDLE          hReqQueue      = NULL;
    int             UrlAdded       = 0;
    HTTPAPI_VERSION HttpApiVersion = HTTPAPI_VERSION_1;
    
    if (argc < 2)
    {
        wprintf(L"%ws: <Url1> [Url2] ... \n", argv[0]);
        return -1;
    }
```

## 初始化HTTP Service
```c++
    //
    // 初始化HTTP Server APIs
    //
    retCode = HttpInitialize( 
                HttpApiVersion,
                HTTP_INITIALIZE_SERVER,    // Flags
                NULL                       // Reserved
                );

    if (retCode != NO_ERROR)
    {
        wprintf(L"HttpInitialize failed with %lu \n", retCode);
        return retCode;
    }

    //
    // 创建请求队列句柄
    //
    retCode = HttpCreateHttpHandle(
                &hReqQueue,        // Req Queue
                0                  // Reserved
                );

    if (retCode != NO_ERROR)
    {    
        wprintf(L"HttpCreateHttpHandle failed with %lu \n", retCode);
        goto CleanUp;
    }
```

## 注册URLs进行监听

```c++
    //
    // 命令行参数指定要监听的URI。为每个URI调用HttpAddUrl。
    //
    // URI是一个完全合格的URI，必须包含终止字符(/)
    //
    for (int i = 1; i < argc; i++)
    {
        wprintf(L"listening for requests on the following url: %s\n", argv[i]);

        retCode = HttpAddUrl(
                    hReqQueue,    // Req Queue
                    argv[i],      // Fully qualified URL
                    NULL          // Reserved
                    );

        if (retCode != NO_ERROR)
        {
            wprintf(L"HttpAddUrl failed with %lu \n", retCode);
            goto CleanUp;
        }
        else
        {
            //
            // Track the currently added URLs.
            //
            UrlAdded ++;
        }
    }
```

## 调用程序以接收请求
```c++
    DoReceiveRequests(hReqQueue);
```

## 清理HTTP Server API
```c++
CleanUp:

    //
    // 对所有添加的URI调用HttpRemoveUrl.
    //
    for(int i=1; i<=UrlAdded; i++)
    {
        HttpRemoveUrl(
              hReqQueue,     // Req Queue
              argv[i]        // Fully qualified URL
              );
    }

    //
    // 关闭请求队列句柄.
    //
    if(hReqQueue)
    {
        CloseHandle(hReqQueue);
    }

    // 
    // 调用HttpTerminate.
    //
    HttpTerminate(HTTP_INITIALIZE_SERVER, NULL);

    return retCode;
```

## 接收请求

```c++
/*******************************************************************++

函数说明:
    他的功能是接收一个请求。
	该函数调用相应的函数来处理响应。

参数:
    hReqQueue - 请求队列句柄

返回值:
    Success/Failure.

--*******************************************************************/
DWORD DoReceiveRequests(
    IN HANDLE hReqQueue
    )
{
    ULONG              result;
    HTTP_REQUEST_ID    requestId;
    DWORD              bytesRead;
    PHTTP_REQUEST      pRequest;
    PCHAR              pRequestBuffer;
    ULONG              RequestBufferLength;

    //
    // 分配一个2 KB缓冲区。 这个大小应该适用于大多数请求。 如果需要，
	// 可以增加缓冲区大小。HTTP_REQUEST结构也需要空间。
    //
    RequestBufferLength = sizeof(HTTP_REQUEST) + 2048;
    pRequestBuffer      = (PCHAR) ALLOC_MEM( RequestBufferLength );

    if (pRequestBuffer == NULL)
    {
        return ERROR_NOT_ENOUGH_MEMORY;
    }

    pRequest = (PHTTP_REQUEST)pRequestBuffer;

    //
    // 等待一个新请求. 标记为一个NULL请求ID
    //

    HTTP_SET_NULL_ID( &requestId );

    for(;;)
    {
        RtlZeroMemory(pRequest, RequestBufferLength);

        result = HttpReceiveHttpRequest(
                    hReqQueue,          // Req Queue
                    requestId,          // Req ID
                    0,                  // Flags
                    pRequest,           // HTTP request buffer
                    RequestBufferLength,// req buffer length
                    &bytesRead,         // bytes received
                    NULL                // LPOVERLAPPED
                    );
```

## 处理HTTP请求
```c++
        if(NO_ERROR == result)
        {
            //
            // Worked! 
            // 
            switch(pRequest->Verb)
            {
			    /* GET 请求处理 */
                case HttpVerbGET:
                    wprintf(L"Got a GET request for %ws \n", 
                            pRequest->CookedUrl.pFullUrl);

                    result = SendHttpResponse(
                                hReqQueue, 
                                pRequest, 
                                200,
                                "OK",
                                "Hey! You hit the server \r\n"
                                );
                    break;

			    /* POST 请求处理 */
                case HttpVerbPOST:

                    wprintf(L"Got a POST request for %ws \n", 
                            pRequest->CookedUrl.pFullUrl);

                    result= SendHttpPostResponse(hReqQueue, pRequest);
                    break;

                default:
                    wprintf(L"Got a unknown request for %ws \n", 
                            pRequest->CookedUrl.pFullUrl);

                    result = SendHttpResponse(
                                hReqQueue, 
                                pRequest,
                                503,
                                "Not Implemented",
                                NULL
                                );
                    break;
            }

            if(result != NO_ERROR)
            {
                break;
            }

            //
            // 重置请求ID用于处理下一个请求.
            //
            HTTP_SET_NULL_ID( &requestId );
        }
        else if(result == ERROR_MORE_DATA)
        {
            //
            // 输入缓冲区太小，无法容纳请求标头。增加缓冲区大小，再次调用API。
            //
            // 再次调用API时，通过传递RequestID来处理失败的请求。
            //
            // 该RequestID从旧缓冲区读取。
            //
            requestId = pRequest->RequestId;

            //
            // 释放旧的缓冲区并分配一个新的缓冲区。
            //
            RequestBufferLength = bytesRead;
            FREE_MEM( pRequestBuffer );
            pRequestBuffer = (PCHAR) ALLOC_MEM( RequestBufferLength );

            if (pRequestBuffer == NULL)
            {
                result = ERROR_NOT_ENOUGH_MEMORY;
                break;
            }

            pRequest = (PHTTP_REQUEST)pRequestBuffer;

        }
        else if(ERROR_CONNECTION_INVALID == result && 
                !HTTP_IS_NULL_ID(&requestId))
        {
            // 当尝试使用更多缓冲区来处理请求时,TCP连接被对方破坏
            // 继续下一个请求。
            
            HTTP_SET_NULL_ID( &requestId );
        }
        else
        {
            break;
        }

    }

    if(pRequestBuffer)
    {
        FREE_MEM( pRequestBuffer );
    }

    return result;
}
```

## 发送一个HTTP响应
```c++
/*******************************************************************++

函数说明:
    这个函数用于发送一个HTTP响应

参数:
    hReqQueue     - 请求队列句柄
    pRequest      - 解析出的HTTP请求
    StatusCode    - Response状态码
    pReason       - Response原因短语
    pEntityString - Response实体主体

返回值:
    Success/Failure.
--*******************************************************************/

DWORD SendHttpResponse(
    IN HANDLE        hReqQueue,
    IN PHTTP_REQUEST pRequest,
    IN USHORT        StatusCode,
    IN PSTR          pReason,
    IN PSTR          pEntityString
    )
{
    HTTP_RESPONSE   response;
    HTTP_DATA_CHUNK dataChunk;
    DWORD           result;
    DWORD           bytesSent;

    //
    // 初始化HTTP response结构体
    //
    INITIALIZE_HTTP_RESPONSE(&response, StatusCode, pReason);

    //
    // 添加一个known header.
    //
    ADD_KNOWN_HEADER(response, HttpHeaderContentType, "text/html");
   
    if(pEntityString)
    {
        // 
        // 添加一个entity chunk.
        //
        dataChunk.DataChunkType           = HttpDataChunkFromMemory;
        dataChunk.FromMemory.pBuffer      = pEntityString;
        dataChunk.FromMemory.BufferLength = 
                                       (ULONG) strlen(pEntityString);

        response.EntityChunkCount         = 1;
        response.pEntityChunks            = &dataChunk;
    }

    // 
    // 因为entity body在一个调用中发送，所以不需要指定Content-Length。
    //
    
    result = HttpSendHttpResponse(
                    hReqQueue,           // ReqQueueHandle
                    pRequest->RequestId, // Request ID
                    0,                   // Flags
                    &response,           // HTTP response
                    NULL,                // pReserved1
                    &bytesSent,          // bytes sent  (OPTIONAL)
                    NULL,                // pReserved2  (must be NULL)
                    0,                   // Reserved3   (must be 0)
                    NULL,                // LPOVERLAPPED(OPTIONAL)
                    NULL                 // pReserved4  (must be NULL)
                    ); 

    if(result != NO_ERROR)
    {
        wprintf(L"HttpSendHttpResponse failed with %lu \n", result);
    }

    return result;
}
```

## 发送一个HTTP POST响应

```c++
#define MAX_ULONG_STR ((ULONG) sizeof("4294967295"))

/*******************************************************************++

函数说明:
    这个函数在读取entity body后发送HTTP响应

参数:
    hReqQueue     - 请求队列句柄
    pRequest      - 解析出的HTTP request.

返回值:
    Success/Failure.
--*******************************************************************/

DWORD SendHttpPostResponse(
    IN HANDLE        hReqQueue,
    IN PHTTP_REQUEST pRequest
    )
{
    HTTP_RESPONSE   response;
    DWORD           result;
    DWORD           bytesSent;
    PUCHAR          pEntityBuffer;
    ULONG           EntityBufferLength;
    ULONG           BytesRead;
    ULONG           TempFileBytesWritten;
    HANDLE          hTempFile;
    TCHAR           szTempName[MAX_PATH + 1];
    CHAR            szContentLength[MAX_ULONG_STR];
    HTTP_DATA_CHUNK dataChunk;
    ULONG           TotalBytesRead = 0;

    BytesRead  = 0;
    hTempFile  = INVALID_HANDLE_VALUE;

    //
    // 为实体缓冲区分配空间。 缓冲区可按需增加。
    //
    EntityBufferLength = 2048;
    pEntityBuffer      = (PUCHAR) ALLOC_MEM( EntityBufferLength );

    if (pEntityBuffer == NULL)
    {
        result = ERROR_NOT_ENOUGH_MEMORY;
        wprintf(L"Insufficient resources \n");
        goto Done;
    }

    //
    // 初始化HTTP response结构体.
    //
    INITIALIZE_HTTP_RESPONSE(&response, 200, "OK");

    //
    // 对于POST，从客户端回显实体
    //
    // 注意: 如果HTTP_RECEIVE_REQUEST_FLAG_COPY_BODY标识通过HttpReceiveHttpRequest()
	//       传递，则entity将是HTTP_REQUEST的一部分(使用pEntityChunks字段).因为此标识
	//       未被传递，则entity不在HTTP_REQUEST中.
    //
   
    if(pRequest->Flags & HTTP_REQUEST_FLAG_MORE_ENTITY_BODY_EXISTS)
    {
        // 实体主体通过多个调用发送. 收集这些在一个文件并回发.创建一个临时文件
        //

        if(GetTempFileName(
                L".", 
                L"New", 
                0, 
                szTempName
                ) == 0)
        {
            result = GetLastError();
            wprintf(L"GetTempFileName failed with %lu \n", result);
            goto Done;
        }

        hTempFile = CreateFile(
                        szTempName,
                        GENERIC_READ | GENERIC_WRITE, 
                        0,                  // Do not share.
                        NULL,               // No security descriptor.
                        CREATE_ALWAYS,      // Overrwrite existing.
                        FILE_ATTRIBUTE_NORMAL,    // Normal file.
                        NULL
                        );

        if(hTempFile == INVALID_HANDLE_VALUE)
        {
            result = GetLastError();
            wprintf(L"Cannot create temporary file. Error %lu \n",
                     result);
            goto Done;
        }

        do
        {
            //
            // 从请求中读取entity chunk.
            //
            BytesRead = 0; 
            result = HttpReceiveRequestEntityBody(
                        hReqQueue,
                        pRequest->RequestId,
                        0,
                        pEntityBuffer,
                        EntityBufferLength,
                        &BytesRead,
                        NULL 
                        );

            switch(result)
            {
                case NO_ERROR:

                    if(BytesRead != 0)
                    {
                        TotalBytesRead += BytesRead;
                        WriteFile(
                                hTempFile, 
                                pEntityBuffer, 
                                BytesRead,
                                &TempFileBytesWritten,
                                NULL
                                );
                    }
                    break;

                case ERROR_HANDLE_EOF:

                    //
                    // The last request entity body has been read.
                    // Send back a response. 
                    //
                    // To illustrate entity sends via 
                    // HttpSendResponseEntityBody, the response will 
                    // be sent over multiple calls. To do this,
                    // pass the HTTP_SEND_RESPONSE_FLAG_MORE_DATA
                    // flag.
                    
                    if(BytesRead != 0)
                    {
                        TotalBytesRead += BytesRead;
                        WriteFile(
                                hTempFile, 
                                pEntityBuffer, 
                                BytesRead,
                                &TempFileBytesWritten,
                                NULL
                                );
                    }

                    //
                    // Because the response is sent over multiple
                    // API calls, add a content-length.
                    //
                    // Alternatively, the response could have been
                    // sent using chunked transfer encoding, by  
                    // passimg "Transfer-Encoding: Chunked".
                    //

                    // NOTE: Because the TotalBytesread in a ULONG
                    //       are accumulated, this will not work
                    //       for entity bodies larger than 4 GB. 
                    //       For support of large entity bodies,
                    //       use a ULONGLONG.
                    // 

                  
                    sprintf_s(szContentLength, MAX_ULONG_STR, "%lu", TotalBytesRead);

                    ADD_KNOWN_HEADER(
                            response, 
                            HttpHeaderContentLength, 
                            szContentLength
                            );

                    result = 
                        HttpSendHttpResponse(
                               hReqQueue,           // ReqQueueHandle
                               pRequest->RequestId, // Request ID
                               HTTP_SEND_RESPONSE_FLAG_MORE_DATA,
                               &response,       // HTTP response
                               NULL,            // pReserved1
                               &bytesSent,      // bytes sent-optional
                               NULL,            // pReserved2
                               0,               // Reserved3
                               NULL,            // LPOVERLAPPED
                               NULL             // pReserved4
                               );

                    if(result != NO_ERROR)
                    {
                        wprintf(
                           L"HttpSendHttpResponse failed with %lu \n", 
                           result
                           );
                        goto Done;
                    }

                    //
                    // Send entity body from a file handle.
                    //
                    dataChunk.DataChunkType = 
                        HttpDataChunkFromFileHandle;

                    dataChunk.FromFileHandle.
                        ByteRange.StartingOffset.QuadPart = 0;

                    dataChunk.FromFileHandle.
                        ByteRange.Length.QuadPart = 
                                          HTTP_BYTE_RANGE_TO_EOF;

                    dataChunk.FromFileHandle.FileHandle = hTempFile;

                    result = HttpSendResponseEntityBody(
                                hReqQueue,
                                pRequest->RequestId,
                                0,           // This is the last send.
                                1,           // Entity Chunk Count.
                                &dataChunk,
                                NULL,
                                NULL,
                                0,
                                NULL,
                                NULL
                                );

                    if(result != NO_ERROR)
                    {
                       wprintf(
                          L"HttpSendResponseEntityBody failed %lu\n", 
                          result
                          );
                    }

                    goto Done;

                    break;
                       

                default:
                  wprintf( 
                   L"HttpReceiveRequestEntityBody failed with %lu \n", 
                   result);
                  goto Done;
            }

        } while(TRUE);
    }
    else
    {
        // 此请求没有实体主体。
        //
        
        result = HttpSendHttpResponse(
                   hReqQueue,           // ReqQueueHandle
                   pRequest->RequestId, // Request ID
                   0,
                   &response,           // HTTP response
                   NULL,                // pReserved1
                   &bytesSent,          // bytes sent (optional)
                   NULL,                // pReserved2
                   0,                   // Reserved3
                   NULL,                // LPOVERLAPPED
                   NULL                 // pReserved4
                   );
        if(result != NO_ERROR)
        {
            wprintf(L"HttpSendHttpResponse failed with %lu \n",
                    result);
        }
    }

Done:

    if(pEntityBuffer)
    {
        FREE_MEM(pEntityBuffer);
    }

    if(INVALID_HANDLE_VALUE != hTempFile)
    {
        CloseHandle(hTempFile);
        DeleteFile(szTempName);
    }

    return result;
}
```