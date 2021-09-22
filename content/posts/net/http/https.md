---
title: "HTTPS 相关介绍"
date: 2021-09-22T10:06:24+08:00
draft: false
toc: true
images:
tags: 
  - https
---

HTTPS 又称 HTTP over SSL/TLS，运行在 SSL/TLS 上的 HTTP。
SSL/TLS 是负责加密通信的协议，建立在 TCP/IP 之上，所以也是一个可靠的传输协议。

## SSL
SSL（Secure socket layer）由网景公司发明，当发展到 3.0时被标准化，改名为 TLS（Transport layer security），由于历史原因很多人称为 SSL/TLS 或者 SSL。

SSL 参考了很多来自密码学的研究成果，比如对称加密、非对称加密、摘要算法、数字签名、数字证书等，给通信的两方创建一个私密、安全的传输通道。

## 浏览器变化
当我们为域名启动了 SSL/TLS（https）时，在浏览器上我们可以发现多出来要给小锁头的标志，并且 url 也变成 https 开头的地址