---
layout: post
title: "Dispatch Source"
date: 2015-11-24 00:16:49 +0800
comments: true
categories: iOS 
---

###目录
1. 概述
2. 事件与种类
3. 使用 Dispatch Source

###概述

iOS 中有两种 Source，一种是 Run Loop Source ,一种是 Dispatch Source。

Source 可以理解为产生事件的地方，Source 产生事件，然后 Source 的回调函数负责 处理这些事件。

在 Run Loop 中， Run Loop Source 产生事件，之后唤醒 Run Loop， Run Loop 便执行该 Source 的回调函数。

Dispatch Source 也会产生一些特定的事件，当这些事件发生的时候，其回调的 block 会自动加入到 对应的 dispatch queue 中。

###事件与种类

Dispatch Source 监控的事件主要有以下几种：

1. Timer dispatch sources generate periodic notifications.
2. Signal dispatch sources notify you when a UNIX signal arrives.
3. Descriptor sources notify you of various file- and socket-based operations, such as:
    *  When data is available for reading
    *  When it is possible to write data
    *  When files are deleted, moved, or renamed in the file system
    *  When file meta information changes
4. Process dispatch sources notify you of process-related events, such as:
    * When a process exits
    * When a process issues a fork or exec type of call
    * When a signal is delivered to the process
5. Mach port dispatch sources notify you of Mach-related events.
6. Custom dispatch sources are ones you define and trigger yourself.

对应着系统定义的 Dispatch Source 种类：

1.  DISPATCH_SOURCE_TYPE_TIMER 定时器
2.  DISPATCH_SOURCE_TYPE_SIGNAL 接收到 UNIX 信号
3.  DISPATCH_SOURCE_TYPE_READ 文件可读
4.  DISPATCH_SOURCE_TYPE_WRITE 文件可写
5.  DISPATCH_SOURCE_TYPE_VNODE 文件系统有变更
6.  DISPATCH_SOURCE_TYPE_PROC 与进程相关的事件
7.  DISPATCH_SOURCE_TYPE_MACH_SEND  Mach 端口发送事件
8.  DISPATCH_SOURCE_TYPE_MACH_RECV  Mach 端口接收事件
9.  DISPATCH_SOURCE_TYPE_DATA_ADD 用户自定义的事件－变量相加
10. DISPATCH_SOURCE_TYPE_DATA_OR  用户自定义的事件－变量相或


这些事件都是来自于 XNU 内核中，kqueue 是用来处理这些事件的最好的一种方法，Dispatch Source 就是对 kqueue 的封装。

###使用 Dispatch Source

使用 Dispatch Source 一般分为以下几步：

1. 创建 Dispatch Source
2. 配置 Dispatch Source
3. 启动 Dispathc source
4. 手动或自动发送 Dispatch Source 事件

####自定义 Dispatch Source

首先创建并配置 Dispatch Sourcce

```objc
    self.count = 0;
    dispatch_queue_t queue = dispatch_get_main_queue();
    self.source = dispatch_source_create(DISPATCH_SOURCE_TYPE_DATA_ADD, 0, 0, queue);//创建 Dispatch Source，种类为DISPATCH_SOURCE_TYPE_DATA_ADD，即获取到的变量会相加


    dispatch_source_set_event_handler(self.source, ^{

        UInt64 value = dispatch_source_get_data(self.source);

        self.count += value;
        NSLog(@"n = %ld",(long)self.count);

    });//配置 Dispatch Source 的回调 block，即当收到该 Source 事件时候，就把该 block 追加到对应的queue中

    dispatch_resume(self.source); //启动 Source， Source 默认是 suspend 的，需要手动启动
```

发送 Dispatch Source 事件

```objc
- (IBAction)buttonDidClicked:(id)sender {

    self.count = 0.0;

    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT , 0);

    for (NSInteger i = 0; i < 100; i++) {

        dispatch_async(queue, ^{

            dispatch_source_merge_data(self.source, 1);//发送 Source 事件
            sleep(1);

        });
    }
}
```

之后，我们点击按钮之后，就会通过全局并发队列 发送 100 次 Source 事件，不过并不会 回调 100 次 回调 Block， 这是因为对应的 队列在接收到 Source 事件之后，假如队列处于空闲状态，就会执行对应的 回调 Block，假如队列处于 busy 状态，该事件就会和后面的一系列同种事件通过一定的方式被合并起来（此例中是按照相加的方式），等到队列空闲的时候再执行。


#### Dispatch Source Timer

利用 Dispatch Source 的 DISPATCH_SOURCE_TYPE_TIMER 类型，我们可以创建一个 跨线程的 定时器（我们平时使用的 NSTimer 是基于 Run Loop 的 timer 事件，只能在对应的线程里触发）

```objc
    dispatch_queue_t queue = dispatch_get_main_queue();
    self.timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue);//创建一个 timer；
    dispatch_source_set_timer(self.timer, DISPATCH_TIME_NOW, 2 * NSEC_PER_SEC, 0 * NSEC_PER_SEC);//配置 timer，从现在起，每两秒在主线程触发一次，精度为0s
    dispatch_source_set_event_handler(self.timer, ^{

        NSLog(@"%ld", self.count++);
    });//timer 触发之后的回调 block

    dispatch_resume(self.time); //启动 timer
```



###参考资料
1. [Dispatch Sources](https://developer.apple.com/library/mac/documentation/General/Conceptual/ConcurrencyProgrammingGuide/GCDWorkQueues/GCDWorkQueues.html);

2. [GCD入门（三）: Dispatch Sources](http://www.dreamingwish.com/article/grand-central-dispatch-basic-3.html);

3. [Parse的底层多线程处理思路：GCD高级用法](https://github.com/ChenYilong/ParseSourceCodeStudy/blob/master/01_Parse的多线程处理思路/Parse的底层多线程处理思路.md);
