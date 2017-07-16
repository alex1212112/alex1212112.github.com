---
layout: post
title: "iOS 中的断点续传"
date: 2014-07-16 17:25:31 +0800
comments: true
categories: iOS
---
![](/images/201407161730.png)

#### 关键点

实现断点续传的关键是自定义http request的头部的range域属性

Range头域可以请求实体的一个或者多个子范围。例如，

表示头500个字节：bytes=0-499

表示第二个500字节：bytes=500-999

表示最后500个字节：bytes=-500

表示500字节以后的范围：bytes=500-

第一个和最后一个字节：bytes=0-0,-1

同时指定几个范围：bytes=500-600,601-999

#### 基本思想

1、获取已下载文件的大小，用来确定下载的文件从什么地方开始续传(即获取range属性的范围);

2、设置http request请求头文件，要包含range属性;

```objc
NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url cachePolicy:NSURLRequestUseProtocolCachePolicy timeoutInterval:5.0f];
   // 设置请求头文件
NSString *rangeValue = [NSString stringWithFormat:@"bytes=%llu-", from];
[request addValue:rangeValue forHTTPHeaderField:@"Range"];
```

3、 发起http下载请求;

#### 具体实现

可以用iOS自带的NSURLConnection，也可以使用第三方如AFNetWorking实现。

[NSURLConnection实现方法](http://www.cnblogs.com/liufeng24/p/3555303.html)；

[AFNetworking实现方法](http://blog.csdn.net/zhaoxy_thu/article/details/21383515);

[TCBlobDownload](https://github.com/thibaultCha/TCBlobDownload);

#### 参考资料

1.[ios 实现断点续传 一 nsurlconnection](http://blog.csdn.net/sirchenhua/article/details/7286312);

2.[IOS Http断点续传浅析](http://longminxiang.blog.163.com/blog/static/5933298520137933235997/);

3.[iOS开发网络编程之断点续传-NSURLConnection](http://www.cnblogs.com/liufeng24/p/3555303.html);

4.[AFNetworking实现程序重新启动时的断点续传](http://blog.csdn.net/zhaoxy_thu/article/details/21383515)；
