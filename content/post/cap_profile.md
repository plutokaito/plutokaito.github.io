---
title: "CAP 的基础了解"
date: 2021-06-30
tags: ["分布式"]
description : "这篇文章讲述的是 CAP 的基础理论。"
---

## 前言
该篇文章主要是讲述了分布式系统中 CAP 的基本理解，主要来源是极客时间专栏 [从0开始学架构](https://time.geekbang.org/column/intro/100006601), 有兴趣的可以了解一下。以下的英文原文来自于 [CAP Theorem: Revisited](https://robertgreiner.com/cap-theorem-revisited/)


# 什么是 CAP 理论


CAP 定理又被称为布鲁尔定理。只有在互联和共享数据的分布式系统中讨论 CAP 才会有意义，由此可以看出 CAP 讨论的是对数据的读写操作，它关注的粒度是数据，而不是整个系统。当我们落地实践时，需要将系统内的数据按照不同应用场景和要求进行分类，每类数据选择不同的策略（AP 还是 CP）, 而不是直接限定整个系统所有数据都是同一个策略。


C: Consistency(一致性):


> A read is guaranteed to return the most recent write for a given client.



对于某个指定的客户端，读操作能保证返回最新的写操作的结果。因为在事务在执行过程中， client 是无法读取到未提交的数据的，有点类似于数据库的 rr 机制， 只有等到事务提交后， client 才能读取到事务写入的数据，而如果事务失败则进行回滚， client 也不会读取到事务中间写入的数据。核心是：保证客户端获取的数据都是最新的。


A: Availability(可用性):


> A non-failing node will return a reasonable response within a reasonable amount of time (no error or timeout).



非故障节点在合理的时间内返回一个合理的结果（非错误和超时）。 重点是 返回的结果是非错误和超时。


P: Partition Tolerance (分区容忍性):


> The system will continue to function when network partitions occur.



当出现网络分区后，系统能够继续“履行职责”。 网络分区导致的原因： 不论什么原因，可能是丢包，可能是链接中断，还有可能是拥塞， 只要是导致了一部分节点和另外一部分无法通信。就通通算在里面。但出现网络故障时：系统还能运行。


## CAP 应用


在分布式系统中， 我们无法保证网络 100% 可靠，可能出故障，分区是一个必然的现象，因此必须选择 P 要素。


- CP ： 一致性和分区容忍性

![image.png](/images/post/architecture/cap/cp.png)

A1 当更新 x 的值为 3 时， A2 此时还是 x = 2, 当 client 去读该系统的 x 值， 按照一致性原则，此时应该返回 Error， 这样就不满足了可用性了。 


- AP:  可用性和分区容忍性

![image.png](/images/post/architecture/cap/ap.png)

A1 更新 x = 3时， A2 还是 x = 2， 当 client 去读该系统的 x 的值时, 按照 A 可用性来说，此时返回不是应该 error 或者超时，而是一个合理的值 2。
​

很多工程上都选择了 AP 并保证了最终一致性，此时用的方式为：人工数据订正和补偿，定时脚本批量检查和修复。