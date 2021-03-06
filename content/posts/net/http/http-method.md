---
title: "Http 有哪些请求方法"
date: 2021-10-13T16:08:32+08:00
draft: false
toc: true
images:
tags: 
  - http
---

我们在编写 API 接口时，会要求客户端通过什么请求方式来访问这个 API 接口，不同请求方式访问到的结果也不相同。请求方式更像是一种动作指令，告诉服务端我接下来通过这个这个动作来执行这个操作了。

## 请求方式
在 Http 0.9 协议中只有 GET 一种请求方式。

在 Http 1.0 协议中定义了三种请求方式，分别是 GET、POST 和 HEAD。

在 Http 1.1 协议中定义了8种请求方式，分别对应 GET、HEAD、POST、PUT、DELETE、OPTIONS、CONNECT、TRACE 。

### GET
从服务器获取资源，这个资源可以是文本、图片、音视频等文件。

GET 请求 从 Http 0.9 诞生以来一直沿用至今，应该是用的最多的请求方式。

### HEAD
HEAD 请求与 GET 请求类似，都是从服务器获取资源，不过 HEAD 不同的是，服务器不会返回请求的实体数据，只会传回响应头，也就是资源的元信息，避免传输 body 数据的浪费。

比如判断文件是否存在，就可以使用 HEAD 请求方式，不需要使用 GET 把整个资源拉取下来，但是在实际开发过程中，我们通常使用 GET 请求方式，将结果传递到 body 中。

### POST
向指定资源提交数据进行处理，数据放在请求头的 body 里面。

POST 请求也是比较常用的一种请求方式，比如我们在网页进行表单提交、文件上传等操作，都是使用 POST 请求。

### PUT
PUT 请求与 POST 请求类似，向指定资源上传最新的数据进行处理。

PUT 更像是 update，POST 是 insert。

### DELETE
请求服务器删除指定的资源。

### CONNECT
要求服务器为客户端与另一台远程服务器建立特殊的连接隧道，此时的 Web 服务器类似一个代理服务器。

### OPTIONS
请求服务器返回资源所支持的所有 Http 请求方法。

### TRACE
显示出服务器收到的请求信息，一般用于诊断和测试。

## 安全与幂等
### 什么是安全
在 Http 协议中，安全指的是请求方式不会对服务器资源照成任何影响。

因为 GET/HEAD 请求是获取资源信息，是只读操作，不会对服务器资源照成影响，所以是安全的。

而 POST/PUT/DELETE 请求会修改服务器上资源信息，所以是不安全的。

### 什么是幂等
幂等指的是多次执行相同的操作，结果也相同，多次幂结果也相等。

对于 GET/HEAD 请求操作多次获取资源，得到的结果也是相同的，它们是幂等的。

对于 DELETE 请求操作，多次删除同一资源，得到的结果是资源不存在，是幂等的。

对于 POST/PUT 请求操作，POST 可以理解为新增，PUT 为更新，多次新增会创建多个文件，多次更新结果还是第一次更新的状态，所以 POST 是不幂等，PUT 是幂等的。
