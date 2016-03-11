---
layout: post
title: "关于 UILocalNotification 的几个问题"
date: 2015-05-27 18:47:25 +0800
comments: true
categories: 
---
![](/images/201505271852.png)


###问题

1. 如何修改已经启动的`UILocalNotification`

2. 为何`cancelAllLocalNotifications`方法调用完毕之后并不生效


###回答

1. 停止已经启动的`UILocalNotification`，创建一个新的`UILocalNotification`并启动，直接修改已经启动的`UILocalNotification`是不生效的，因为你在`[[UIApplication sharedApplication] scheduleLocalNotification:notification]`的时候，复制了一个当前的notification，并提交到系统的推送队列上，假如不提交的话，系统是不认识这个notification的。
 
2. 执行完 `cancelAllLocalNotifications` 后，你再打印`[UIApplication sharedApplication].scheduledLocalNotifications`,发现其并不为空,stackoverflow 上有人说这个方法不是同步执行的，只会在当前的 run loop 结束之后生效,有人说可以通过

```objc
[UIApplication sharedApplication].scheduledLocalNotifications = nil
```

的方法使其立即生效，测试发现并没有什么作用，真正有用的方法还是 stackoverflow 上一个开发者提出的变通方法:

```objc
[[UIApplication sharedApplication] cancelAllLocalNotifications];
    long count;
    while ((count = [[[UIApplication sharedApplication] scheduledLocalNotifications] count]) > 0) {
        NSLog(@"Remaining notificaitons to cancel: %lu",(unsigned long)count);
        [NSThread sleepForTimeInterval:.01f];
    }
```

对于问题1，有时候在 ` [[UIApplication sharedApplication] cancelLocalNotification:notification]`的时候可能会发现并没有立即cancel掉，也可以采用这样的方法，在 cancel 完之后 执行下 `[NSThread sleepForTimeInterval:.01f]`;

###参考资料

1. [scheduledLocalNotifications](http://stackoverflow.com/questions/25948037/ios-8-uiapplication-sharedapplication-scheduledlocalnotifications-empty)

2. [cancelAllLocalNotifications](http://stackoverflow.com/questions/13163535/cancelalllocalnotifications-not-working-on-iphone-3gs)

