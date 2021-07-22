---
title: "Golang 源码解析系列之Channel"
date: 2021-07-22T23:56:32+08:00
hidden: true
draft: false
tags: ["golang", "源码解析", "channel"]
keywords: ["channel"]
description: ""
slug: "Theme Preview"
toc: true
---

# 什么是channel

channel 又称管道，可以在多个 goroutine 中进行访问（内存共享），管道采用的是 FIFO（先进先出）队列，对 FIFO 队列的操作都是属于原子操作，不需要进行加锁。

# 初始化
channel 提供了使用变量声明和使用内置函数 make 进行初始化，默认是双向读写，但是可以设置 channel 只负责读或者写操作。

```
ch := make(chan int) // 可以读取和发送类型 int 的数据
ch := make(chan<- int) // 只可以发送 int 类型的数据
ch := make(<-chan int) // 只可以读取 int 类型的数据
```
## 变量声明方式：
```
var ch chan int
```
变量声明, 默认为 nil，对 nil 的 channel 进行读取数据或者写入数据都会造成永久阻塞，这是致命错误，需要注意

## make方式：
```
ch := make(chan int, 1) // 有缓冲通道
ch := make(chan int) // 无缓冲通道
```
使用内置函数make()，可以创建带缓存和无缓存的 channel

# 常规操作
使用 `<-` 操作符表示数据的流向，channel 在左边表示往 channel 中写入数据，channel 在右边表示读取 channel 内的数据。

下面简单介绍写入、关闭的使用方式
```
ch := make(chan int, 10) // 声明 int 类型 channel, 缓冲 10 个元素
ch <- 1 // 把 1 写到 channel 中
ch <- 2 // 把 2 写到 channel 中
n1 := <- ch // 从 ch 中读取数据
n2, ok := <- ch // 从 ch 中读取数据
```
从上面的例子中我们可以发现，n2 存在两个返回参数，第一个变量表示读取出来的数据，第二个变量表示是否读取成功，与管道关闭状态无关，区分 channel 关闭和为关闭读取数据。

也就是说，当管道已经关闭了，缓冲区没有数据，那么第一个变量返回类型对应的默认值，第二个变量为 false 。当管道没有关闭，缓冲区存在数据，那么第一个变量返回读取到的数据，第二个变量为true。

使用内置函数 close() 可以关闭管道。


## 注意事项
1、不要在读取端关闭 channel，因为写入端不知道 channel 已经关闭，往已经关闭的 channel 写数据会发生 `panic：close of closed channel`

2、有多个写入端时，不要在写入端进行关闭管道，因为其他写入端不知道管道是否已经关闭，关闭已经关闭的管道会发生`panic：send on closed channel`

3、不要对为 nil 的 channel 进行关闭，否则会发生`panic：close of nil channel `

4、如果只有一个写入端，那么可以放心的关闭 channel

5、对已经 close 的 channel 进行读取数据，会返回 channel 类型所对应的类型零值，不会发生 panic

6、对空通道也就是 channel 为 nil（定义后未 make 初始化），进行读或者写会照成永久的堵塞
