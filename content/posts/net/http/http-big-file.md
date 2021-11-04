---
title: "Http 怎么处理大文件传输"
date: 2021-11-04T14:42:36+08:00
draft: false
toc: true
images:
tags: 
  - http
---

我们都知道 http 支持传输文本、图片、音视频等文件，如果我们直接传输几个G的文件，会导致传输过程中占用了大部分的宽带，影响其他服务。

为此 http 协议提供了数据压缩、分块传输、范围传输、多段范围传输等技术解决这个问题。

## 数据压缩
仔细观察 http 请求报文头字段时，会发现每次请求都会携带 `Accept-Encoding` 头字段，Accept-Encoding 表示客户端支持的压缩类型列表，常用的有 gzip、deflate、br 等，
服务器收到请求后，可以根据 Accept-Encoding 提供的压缩类型列表，选择一种压缩算法，将压缩算法放到 Content-Encoding 响应头中，告诉客户端使用的是哪一种压缩算法，
再将压缩完成的数据返回给客户端。

小知识：gzip 等算法对文本压缩效果好一点，有较好的压缩率。但是压缩图片等文件时，就没有那么好的压缩率了，因为文件本身就已经是高度压缩的。

## 分段传输
分段传输可以理解为将数据拆分成若干个小数据，分批次逐个发送，最后将数据整合成一个完整的数据。
这样的好处是不需要用内存来保存全部的数据，每次只发一部分数据，网络也不会被大数据给占用，内存和宽带都节省了。

## 范围传输
介绍范围传输之前，想象这么一个场景，我们平时使用视频平台看视频时，通常会选择快进或者跳到指定的播放进度，他们是怎么处理视频传输呢？

之前我们说过分段传输，但是分段传输不适合这个场景，而范围传输，允许客户端在请求头里使用专门字段来表示只获取文件的一部分数据。

具体一点是，客户端通过在请求头当中增加 `Accept-Range` 字段，标明需要获取的数据范围，
例如 Accept-Range:bytes=0-9，表示获取文件前10个字节。

如果服务器不支持的话，服务器可以在响应头增加 `Accept-Range:none` 或者不增加该字段，告诉客户端不支持范围传输。

如果服务器支持，那么：
首先获取客户端发送的范围，验证范围是否合法（超出文件大小），如果不合法，返回状态码 416，表示请求的范围有误。

然后根据范围取文件的片段数据，此时状态码 206，表示 body 的数据只是其中的一部分。

之后服务器需要增加 `Content-Range` 响应头字段，告诉客户端实际的偏移量和文件大小，例如 Content-Range: bytes 0-9/100，0-9表示偏移的范围，100表示文件总大小。

之后就是发送数据给客户端了。

客户端报文示例：
```
GET /16-2 HTTP/1.1
Host: www.chrono.com
Accept-Range: bytes=0-9
```

服务器报文示例：
```
HTTP/1.1 206 Partial Content
Content-Length: 10
Content-Range: bytes 0-9/9
helloworld
```

## 多段范围传输
范围请求不止能获取一个范围的数据，还可以同时获取多个范围的数据。

客户端通过在 Accept-Range 头部字段中指定多个范围，范围之间使用,进行分隔，例如 `Accept-Range:bytes=0-9,10-19`。

服务器收到客户端发送的请求报文时，需要指定 MIME type 为 multipart/byteranges，并指定分隔标识 boundary，
例如 Content-Type: multipart/byteranges; boundary=0000000000。

客户端报文示例：
```
GET /16-2 HTTP/1.1
Host: www.chrono.com
Range: bytes=0-9, 20-29
```

服务器报文示例：
```
HTTP/1.1 206 Partial Content
Content-Type: multipart/byteranges; boundary=00000000001

Content-Length: 20
Connection: keep-aliveAccept-Ranges: bytes

--00000000001
Content-Type: text/plain
Content-Range: bytes 0-9/96
helloworld
--00000000001
Content-Type: text/plain
Content-Range: bytes 20-29/96
test1test2
--00000000001
```
