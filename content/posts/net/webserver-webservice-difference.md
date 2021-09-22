---
title: "WebServer 与 WebService 有什么区别"
date: 2021-09-19T00:13:05+08:00
draft: false
toc: true
images:
tags: 
  - web server
  - web service
---

## 什么是 Web Server
Web Server 也称为 Web 服务器，Web 服务器有两种定义，分别是硬件和软件。
提供 Http 服务的 Server。

### 硬件
物理形式或者“云”形式的机器，在大多数的情况下可能不是一台服务器，而是利用负载均衡、反向代理等技术组成庞大的集群。
对外部来说可能是一台机器，但是形象是虚拟的。

### 软件
提供 Web 服务的应用程序，一般来说都是运行在硬件服务器上。利用强大的硬件处理能力，能够响应海量来自客户端 Http 请求。
比如磁盘上的网页、图片等静态文件，或者做转发处理，把请求转发给 Tomcat、Node.js 等业务应用，返回动态的信息。

常见的 Web 服务器以下几个：
1. Apache：功能相对完善，学习资料多，上手快。
2. Nginx：提供高性能、高稳定性、易扩展特性
3. Tomcat：使用 Java 开发，但由于性能不是很高，导致应用的比较少。

## 什么是 Web Service
Web Service 基于 Web（Http）的服务器架构技术（client-server 主从架构），可以理解为我们实际用到的项目/服务。
运行在 Http 协议上的服务接口规范，提供 web 应用的软件服务。