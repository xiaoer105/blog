---
title: "Redis 持久化之 RDB"
date: 2021-09-07T09:46:07+08:00
draft: false
tags: 
  - redis
keywords: 
  - Redis持久化
  - RDB
description: "Redis 持久化之 RDB"
toc: true
---

## 前言
Redis 提供两种避免数据丢失的解决方案，一种是 AOF 日志恢复，当出现故障时，通过读取 AOF 文件，把文件中的操作日志都执行一遍。
但是如果 AOF 文件过大时，会导致恢复的时间变得很缓慢，那么有没有其他的方式进行恢复数据呢？

接下来介绍另外一种避免数据丢失的解决方案，那就是 RDB 内存快照。 
## 什么是 RDB 快照
快照可以理解为保存某一时刻 Redis 内存中所有的数据。
## 给哪些数据进行 RDB 快照

## RDB 会造成主线程阻塞吗？

## 在执行 RDB 过程中，允许数据写入吗?

## RDB 多久执行一次？

## AOF 和 RDB 怎么选择？

## 总结