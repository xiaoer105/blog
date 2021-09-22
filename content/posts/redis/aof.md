---
title: "Redis 持久化之 AOF"
date: 2021-08-04T20:34:21+08:00
draft: false
tags: 
  - redis
keywords: 
  - Redis持久化
  - AOF
description: "Redis 持久化之 AOF"
toc: true
---

## 前言
通过这篇文章我们会学习到什么？
1. 什么是 AOF?
2. AOF 做了什么?
3. AOF 提供哪些配置项（写回策略）
4. AOF 日志过大怎么办？
5. AOF 重写会影响主线程工作吗？

## 持久化是什么
我们都知道 Redis 属于内存数据库，所有的数据都是存储在内存当中。
在使用内存作为数据存储方式的时候，就必须考虑一个问题，就是当系统出现宕机、崩溃等致命问题时，内存中的数据将全部丢失。

对于请求不频繁的数据来说，可以直接通过请求数据库的方式读取数据，将数据写入内存，恢复内存中的数据。

对于请求频繁的数据来说，采取请求数据库的方式来恢复内存中的数据，会给数据库带来很大的压力，而且从数据库读取数据肯定没有从内存读取数据快。

那么针对上面说的问题，Redis 提供 RDB 和 AOF（Append Only File）两种持久化方案，避免数据的丢失。

## 什么是 AOF
AOF 是 Redis 提供数据持久化的一种方案，当 Redis 收到一条写入命令时，会将这条命令记录到 AOF 文件当中，保证 aof 文件记录的是数据最新的状态。

### 开启 Redis AOF 
Redis AOF 默认是关闭的，提供有两种开启的方式。

方式一：通过修改 redis.conf 配置文件中 appendonly 为 yes，重启 Redis 服务。
```
# 默认为 no，开启设置 yes 即可
appendonly no
# AOF 文件名称，默认 appendonly.aof
appendfilename "appendonly.aof"
```
方式二：在 redis-cli 中输入 `config set appendonly yes` 命令。
```
127.0.0.1:6379> config set appendonly yes
OK
```
需要注意的是，该命令会导致 Redis 进行阻塞，直到 aof 文件创建完成，如果此时 Redis 中存在数据，那么数据会被记录到 aof 文件当中。


通过以上两种方式都可以开启 aof 持久化，开启成功后在根目录会生成 appendonly.aof（文件名可以通过 redis.conf 中 appendfilename 属性进行修改）文件。

### AOF 文件存储格式
向 Redis 发送一条写入命令时，aof 文件会记录哪些内容呢？
```
127.0.0.1:6379> set name libaitan
OK
```
往 Redis 写入一条 name 为 libaitan 的数据，此时 aof 文件内容为
```
*3
$3
set
$4
name
$8
libaitan
```
我们可以发现 aof 文件内容是按照网络通信协议进行保存的，其中`*3`表示当前命令有3个参数 ，`$3`表示参数数据长度为3。

## AOF 提供哪些配置项（写回策略）
Redis 目前提供三种 AOF 策略

1. Always：
2. Everysec：
3. No：

## 什么是 AOF 重写
### 重写导致文件过大怎么办?
### 重写会造成主线程阻塞吗？



