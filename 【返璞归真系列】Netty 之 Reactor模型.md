---
title: 【返璞归真系列】Netty 之 Reactor模型
date: 2020-09-17 18:30:03
tags: Java
categories: Java
---

# Reactor

> 是基于NIO中实现多路复用的一种模式设计模型，Netty的基石。

wiki对Reactor的解释：
<img src="http://cdn.wdaking.com/2020-09-22-%E6%88%AA%E5%B1%8F2020-09-22%20%E4%B8%8B%E5%8D%882.54.44.png" alt="Wiki-Reactor" style="zoom: 200%;" />

从上述文字中我们可以看出以下关键点 ：

 - 事件驱动（event handling）
 - 可以处理一个或多个输入源（one or more inputs）
 - 通过Service Handler同步的将输入事件（Event）采用多路复用分发给相应的Request Handler（多个）处理

![Reactor模型介绍](http://cdn.wdaking.com/2020-09-22-070214.png)

## 1. I/O 痛点

### 1.1 BIO

在上一章节我们实现了BIO，我们采取多线程的方式来处理读写服务。但这么做依然有很明显的弊端：

>1. 同步阻塞IO，读写阻塞，线程等待时间过长
>
>2. 在制定线程策略的时候，只能根据CPU的数目来限定可用线程资源，不能根据连接并发数目来制定，也就是连接有限制。否则很难保证对客户端请求的高效和公平。
>
>3. 多线程之间的上下文切换，造成线程使用效率并不高，并且不易扩展
>
>4. 状态数据以及其他需要保持一致的数据，需要采用并发同步控制

### 1.2 NIO

## 2. Reactor模型实例

### 2.1 单Reactor单线程模型

### 2.2 单Reactor多线程模型

### 2.3 多Reactor多线程模型

## 3. 时间处理模式

