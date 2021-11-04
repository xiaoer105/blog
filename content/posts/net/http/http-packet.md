---
title: "Http 报文长什么样子"
date: 2021-10-13T14:00:19+08:00
draft: false
toc: true
images:
tags: 
  - http
---

Http 是一个`纯文本`协议，所有的头信息（Header）都是 ASCII 码，通过肉眼很容易知道表达的意思。


## 报文格式

http 协议报文分为请求报文和相应报文，其基本结构相同，都是由四部分组成：
1. 起始行：描述请求或相应的基本信息
2. 头部字段集合：使用 key-value 的形式描述信息。
3. 空行：用于告诉服务器，空行以下不是头部信息，使用 CRLF 表示，十六进制的 0D0A。
4. 消息正文：实际传输的数据，可以是一个纯文本、图片、音视频等二进制数据。

### 请求头报文
请求头由4部分组成，分别是请求行、请求头部、空行、请求数据。

请求头一般 4~8k 的传输大小。

```
POST /hello HTTP/1.1
Host: 127.0.0.1
Connection: keep-alive
Content-Length: 1
Pragma: no-cache
Cache-Control: no-cache
Accept: application/json, text/plain, */*
Content-Type: application/json
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/94.0.4606.71 Safari/537.36
sec-ch-ua-platform: "Windows"
Accept-Encoding: gzip, deflate, br
Accept-Language: en,zh;q=0.9,zh-CN;q=0.8

```

### 响应头报文
响应头由4部分组成，分别是响应行、请求头部、空行、请求数据。

```
HTTP/1.1 200 OK
Accept-Ranges: bytes
Content-Length: 12470
Content-Type: text/html; charset=utf-8
Last-Modified: Wed, 13 Oct 2021 06:15:17 GMT
Date: Wed, 13 Oct 2021 06:15:33 GMT

```

## 起始行

### 请求起始行
请求行共由三个部分组成，分别是请求方法、请求地址、Http 版本号，它们之间使用空格分割。

以上面代码块`POST /hello HTTP/1.1`为例，我们可以得出请求方式为 POST，请求地址 /hello，http 版本号 HTTP/1.1。

### 响应起始行
状态行共由三个部分组成，分别是 Http 版本号、状态码、描述信息，它们之间使用空格分割。

以上面代码块`HTTP/1.1 200 OK`为例，我们可以得出请求方式为 POST，请求地址 /hello，http 版本号 HTTP/1.1，描述信息 OK。

## 头部信息（header）
请求头报文和响应头报文的头信息存储格式是一致的，是以 key-value 的形式存储数据，key 和 value 之间使用 : 冒号进行分割， CRLF 进行结尾。

以`Host: 127.0.0.1`为例，key 为 Host，value 为 127.0.0.1。

### 自定义头信息
头部信息除了使用 Http 规定的那些字段之外，还可以添加自定义字段。

在自定义头部字段时，需要注意以下几点：
1. 字段名不区分大小写，但是我们从请求头看到首字母都进行了大写。
2. 字段名里不允许出现空格，但是可以使用连符号`-`进行分割，不可以使用下划线`_`进行分割。
3. 字段名后面必须紧挨着:，: 前面不允许出现空格，但是: 后面允许出现空格。
4. 字段名的顺序没有任何意义，可以任意排列。
5. 字段原则上不允许出现重复，除非字段语义上允许，例如 Set-Cookie。

### 常用头字段
- Host：客户端报文，告知服务器，请求由哪个服务器来处理。Http1.1 规范里必须出现的字段，如果报文没有 Host，那么这就是个错误的报文。
- User-Agent：客户端报文，描述发起请求的 Http 客户端信息。
- Date：共有报文，表示 Http 报文创建时间。
- Server：描述服务器端的软件名称和版本号。
- Accept-Encoding：客户端支持的压缩格式，常见的有 gzip、deflate、br 等，其中 gzip 的压缩率通常能够超过60%，而 br 算法是专门为 HTML 设计的，压缩效率比 gzip 好，能够在提高20%压缩密度。
- Accept：客户端可理解的 MIME type，存在多个使用;分隔，例如 text/html、image/png 等。 

## 消息正文（body）
保存需要传输给服务器的数据，可以是文本、图片等文件。

## 问题思考

### 通过 ip+port 已经可以确定服务器的地址，为什么还需要通过请求头host来找呢?
host 字段是提供给 Web 服务器使用的，比如说一台物理主机存在 a.com、b.com 和 c.com 三个虚拟地址，通过 host 可以知道具体访问哪个虚拟地址。 

### 在头字段后多加了一个 CRLF，导致出现了一个空行，会发生什么？
多出来的空格会被当作 body 来处理。

### 那为什么绝大多数情况下“:”后都只使用一个空格呢？
多一个空格就相当于多一个数据的传输，为了节约资源，去掉无用的信息，保证头部信息尽可能少。

空格可以是零个或多个，但是一个空格成为了一种约束规范。