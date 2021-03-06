---
title: "什么是 DNS"
date: 2021-09-23T18:58:59+08:00
draft: false
toc: true
images:
tags: 
---

在 TCP/IP 协议中使用 IP 标识计算机，数字对计算机来说很方便，但是对于我们来说难以记忆。
DNS 的出现就是为了方便我们记忆 IP 地址。

## 域名
DNS（Domain Name System）又称域系统，用一些有意义的名字代替了 IP 地址，例如 [www.younghot.cn](http://www.younghot.cn) 就是一个域名（Domain Name），也可以称为主机名（Host）。

### 域名等级
域名由`.`分隔成多个单词，级别从左到右逐渐升高，分别对应顶级域名、二级域名、三级域名，例如 `www.younghot.cn、www.bilibili.com` 等。

#### 一级域名
一级域名又称为顶级域名，在域名最右边的就是顶级域名，例如`com、cn、net`等，顶级域名分为国家顶级域名和国际顶级域名。

##### 国家顶级域名
按照由 ISO 3166 国家代码分配的顶级域名进行表示，每个国家都有自己的顶级域名（国家英文缩写），例如中国（cn）。

##### 国际顶级域名
例如工商企业`.com、top`，网络供应商`.net`，教育`.edu`等。

#### 二级域名
二级域名指的是在顶级域名下的域名，例如`.baidu.com、.younghot.cn`等。
#### 三级域名
三级域名由字母、数字和连接符`~`组成，长度不能超过20个字符，例如`.image.younghot.cn`等

### 域名映射
在使用 TCP/IP 进行通信时，需要使用 IP 地址，所以需要把域名进行一个转换，映射到其真实的 IP 地址，也就是域名映射。