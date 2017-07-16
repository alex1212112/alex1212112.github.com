---
layout: post
title: "iOS 中线程同步的一些方法"
date: 2015-10-23 15:21:04 +0800
comments: true
categories: iOS
---

### 目录
1. 使用串行队列
2. 使用 dispatch_group
3. 使用 dispatch_barrier
4. 使用 dispatch_semaphore


### 串行队列
使用串行队列，对于一个资源，同一时刻只有一个任务（线程）访问，这样就避免了资源竞争，实现了资源的同步。我们可以通过 NSOperationQueue 或 dispatch queue来实现串行队列。

*  NSOperationQueue

	创建一个	NSOperationQueue 队列，然后将该队列的最大并发数设置为 1，就是一个串行队列。

```objc
    NSOperationQueue * queue = [[NSOperationQueue alloc] init];

    queue.maxConcurrentOperationCount = 1;//每次只执行一个任务

    for (NSInteger count = 0 ; count < 10; count ++) {

        [queue addOperation: [NSBlockOperation blockOperationWithBlock:^{

            NSLog(@"%ld",count);
        } ]];
    }
```


* dispatch queue
  dispatch queue 也分串行队列和并发队列，我们这里需要创建一个串行队列。

```objc
    dispatch_queue_t serialQueue = dispatch_queue_create("com.ghren.MultiThread.test", DISPATCH_QUEUE_SERIAL); //生成一个串行队列

    for (NSInteger count = 0; count < 3; count ++) {

        dispatch_async(serialQueue, ^{

            NSLog(@"%ld",count);
        });//向该串行队列中添加任务
    }
```




### 使用 dispatch_group

dispatch_group 可以把很多个任务加入到一个组中，然后等组中所有的任务都执行完毕，再去执行新的任务，同步的操作就在这新的任务中实现。

```objc
  dispatch_group_t group = dispatch_group_create();

    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

    for (NSInteger count = 0; count < 3; count ++) {

        dispatch_group_async(group, queue, ^{

            NSLog(@"%ld",count);
        });//把任务添加到组中
    }

    dispatch_group_wait(group, DISPATCH_TIME_FOREVER);//阻塞当前线程，一直到组中任务完成

    NSLog(@"done"); //数据同步点
```



### 使用 dispatch_barrier

假如有这样一个情况，先要执行一组并发的任务，然后在执行一个基于这组并发任务的任务，然后再执行另外一组基于这个任务的并发任务，这种情况虽然也可以通过 dispatch_group 来实现， 但是 GCD 提供了另外一个很优雅的方法，就是 dispatch_barrier,要注意的是对于串行队列使用 dispatch_barrier 意义并不大，因为所有的任务本来就是串行执行的，对于全局的并发队列 `dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)` , dispatch_barrier 也不会生效，所以 dispatch_barrier 一般使用在 自定义的全局并发队列里

```objc
 dispatch_queue_t concurrentQueue = dispatch_queue_create("com.ghren.MultiThread.concurrent", DISPATCH_QUEUE_CONCURRENT); //生成一个并发队列, dispatch_barrier 对于全局并发队列并不起作用，所以要用在自己创建的并发队列里

    for (NSInteger count = 0; count < 3; count ++) {

        dispatch_async(concurrentQueue, ^{

            NSLog(@"%ld",count);
        });
    } // 向并发队列里添加任务


    dispatch_barrier_async(concurrentQueue, ^{

        NSLog(@"do something");
    });//同步点


    for (NSInteger count = 0; count < 3; count ++) {

        dispatch_async(concurrentQueue, ^{

            NSLog(@"%ld",count + 3);
        });
    }//向并发队列里添加新的任务

```

### 使用 dispatch_semaphore

dispatch_semaphore 就是传说中的信号量。其基本原理是，我们可以创建一个信号量，这个信号量有一个值，来表示目前信号量总数有多少，比如我们可以创建一个初始值为 2 的信号量，然后当我们执行一个任务的时候，就消耗一个信号量，信号量总数会减 1 ，当信号量为 0 的时候，即将执行的任务将会被阻塞，然后一直等到超时或信号量又大于0 为止，当我们在一个任务执行完毕的时候，可以为信号量的总量加 1 ，当信号量从 0 变成 1 的时候，等待信号量的任务就会执行下去，通过这个机制，我们可以实现数据的同步。

```objc
    dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);

    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

    dispatch_time_t time = dispatch_time(DISPATCH_TIME_NOW, 3 *NSEC_PER_SEC);

    NSLog(@"task will begin");

    dispatch_after(time, queue, ^{

        NSLog(@"task is done");
        dispatch_semaphore_signal(semaphore);

    });//3 秒之后在全局并发队列里添加一个任务，任务执行完毕之后，增加一个信号量

    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);//当前线程处于阻塞状态，直到信号量不为0

    NSLog(@"start another task");
```
