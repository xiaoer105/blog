---
title: "什么是爬虫"
date: 2021-09-18T23:54:16+08:00
draft: false
toc: true
images:
tags: 
  - 爬虫
---

爬虫是一种可以自动访问 web 资源的应用程序

## 爬虫有什么坏处
过度的消耗网络资源，暂用服务器的带宽，影响了网站对真是用户的分析，还可能导致信息的泄露。
所以，又出现了反爬虫的技术，通过各种手段来限制爬虫，例如通过请求 User-Agent来判断请求来自哪一端（爬虫可以伪装 User-Agent 为浏览器躲过这个限制），
通过后台服务器对 IP、访问频次等机制来判断是否为爬虫，不依赖 User-Agent 来判断，应为它太容易伪造。


