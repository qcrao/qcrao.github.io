---
title: 意犹未尽 —— GPM 的状态流转（十）
author: qcrao
top: false
cover: false
toc: true
mathjax: true
date: 2019-09-06 11:51:50
img:
coverImg:
password:
summary:
categories: 编程语言
tags: golang
---

最开始的时候，我们讲了 GPM 到底是什么，当时没有看过太多源码，所以对 GPM 没有一个整体上的认识。

现在我们终于到了快要结束的时候，可以从宏观上总结一下 GPM，这篇文章尝试从它们的状态流转角度总结。

首先是 G 的状态流转：

![G 的状态流转图](https://user-images.githubusercontent.com/7698088/64057782-d98dd600-cbd3-11e9-918d-8320fd9609c0.png)

图上除了 park_m 和 ready 这一块外其他都有涉及。这一对函数在前面讲 channel 的时候有提到，后面我们有机会再单独说一下这部分。

上图省略了一些垃圾回收的状态，毕竟这个系列没有讲这一块。完整的状态流转图可以到参考资料【欧神 调度器初始化】里看。

接着是 P 的状态流转：

![P 的状态流转图](https://user-images.githubusercontent.com/7698088/64058164-93d40c00-cbd9-11e9-9095-7bc7248a0fb9.png)

> 通常情况下（在程序运行时不调整 P 的个数），P 只会在上图中的四种状态下进行切换。 当程序刚开始运行进行初始化时，所有的 P 都处于 `_Pgcstop` 状态， 随着 P 的初始化（`runtime.procresize`），会被置于 `_Pidle`。

> 当 M 需要运行时，会 `runtime.acquirep` 来使 P 变成 `Prunning` 状态，并通过 `runtime.releasep` 来释放。 

> 当 G 执行时需要进入系统调用，P 会被设置为 `_Psyscall`， 如果这个时候被系统监控抢夺（`runtime.retake`），则 P 会被重新修改为 `_Pidle`。 

> 如果在程序运行中发生 `GC`，则 P 会被设置为 `_Pgcstop`， 并在 `runtime.startTheWorld` 时重新调整为 `_Prunning`。

上面这段引用自【欧神 调度器初始化】。

最后，我们来看 M 的状态变化：

![M 的状态流转图](https://user-images.githubusercontent.com/7698088/64058333-09d97280-cbdc-11e9-8a4d-1843d5be88d0.png)

M 只有自旋和非自旋两种状态。自旋的时候，会努力找工作；找不到的时候会进入非自旋状态，之后会休眠，直到有工作需要处理时，被其他工作线程唤醒，又进入自旋状态。

这是调度器系列的最后一篇文章了。整个系列的核心在于： 

1. GPM 的初始化；
2. M 是怎样一步步找工作；
3. 用户栈和 g0 栈的切换；
4. schedule 的调度循环是怎样运转的；
5. 监控线程做了什么。

其他和 GC 相关的，我们暂时并未涉及到，留到后续再探索。

调度器的探索我们就要告一段落了，感谢陪伴。

# 参考资料
【欧神 调度器初始化】https://github.com/changkun/go-under-the-hood/blob/master/book/zh-cn/part2runtime/ch06sched/init.md

【Go 夜读 boya】https://reading.developerlearning.cn/reading/12-2018-08-02-goroutine-gpm/




