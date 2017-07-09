---
layout: post
title: "KVO 简介"
date: 2014-09-02 16:24:18 +0800
comments: true
categories: iOS 
---
![](/images/201409021638.png)

###目录

什么是KVO？

KVO使用流程

KVO实例

相关技巧

参考资料

###什么是KVO？

KVO提供了一种观察者机制，当指定的对象的属性发生变化的时候，观察者就会收到通知，我们可以根据通知来进行UI更新等操作。它能降低代码之间的藕合度，显著减少开发者编写的代码量。KVO 是 Cocoa 的一项特性，我们可以在 Foundation 的框架里找到它。

###KVO使用流程

*  添加观察者（注册KVO),一般在观察者对象初始化的时候，添加观察者

```objc
- (void)addObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath options:(NSKeyValueObservingOptions)options context:(void *)context;
```

* 属性变化时的回调

```objc
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context;
```

* 移除观察者（注销KVO),一般在观察者对象释放的时候移除，比如在dealloc中移除

```objc
- (void)removeObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath context:(void *)context
```


###KVO实例

这里举一个例子，比如我有一个Person的对象如下：

Person.h

```objc
@interface Person : NSObject

@property (nonatomic, strong) NSString *name;
@property (nonatomic, strong) NSNumber *age;

@end
```
我们的viewController中会引用到该person对象，并在viewController的 - (void)viewWillAppear:(BOOL)animated 方法中注册KVO：

```objc
 [[Person sharePerson] addObserver:self forKeyPath:@"name" options:NSKeyValueObservingOptionInitial context:(__bridge void*)self];
 [[Person sharePerson] addObserver:self forKeyPath:@"age" options:NSKeyValueObservingOptionInitial context:(__bridge void*)self];
```
当person的属性name或age变化的时候，viewController就会调用

```objc
- (void)addObserver:(NSObject *)anObserver
         forKeyPath:(NSString *)keyPath
            options:(NSKeyValueObservingOptions)options
            context:(void *)context
```

我们实现该方法如下：

```objc
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context
{
    if ((__bridge id)context == self)
    {
        if ([keyPath isEqualToString:@"name"])
        {
            self.nameLabel.text = [Person sharePerson].name;
        }
        if ([keyPath isEqualToString:@"age"])
        {
            self.ageLabel.text = [[Person sharePerson].age stringValue];
        }
    }
    else
    {
        [super observeValueForKeyPath:keyPath ofObject:object change:change context:context];
    }
}
```

这样当person属性变化的时候，viewController就会更新nameLabel 和 ageLabel.

不要忘了最后要移除KVO，因为我们是在 - (void)viewWillAppear:(BOOL)animated 中添加的KVO，所以我们要在 - (void)viewWillDisappear:(BOOL)animated 中移除KVO，假如添加了KVO 而在观察者释放的时候没有移除KVO，那么对象会因为向已经释放的观察者发送消息而crash.让然移除的时候也不要移除没有添加到观察者的 keyPath，同样会导致程序crash.

```objc
[self removeObserver:self forKeyPath:@"name" context:(__bridge void*)self];

[self removeObserver:self forKeyPath:@"age" context:(__bridge void*)self];
```

这样，一个实现KVO的简单例子就完成了.  [Demo](https://github.com/alex1212112/KVODemo).

###相关技巧与说明

1. 有时候需要在第一次运行代码的时候更新一次 UI。我们可以设置KVO的选项   NSKeyValueObservingOptionInitial 的选项。这将会让 KVO 通知在调用 -addObserver:forKeyPath:... 的时候也被触发。

2. KVO 能降低代码的藕合度，是因为Model对象在改变的时候，所有观察者能自动的去处理相关逻辑，这样我们就不用在模型对象里去编写专门的代码去通知观察者了，从而保持了模型类的简洁性。

3. 有时候你可能想在第一次更新UI的时候去检查Model的属性是否为空，如果为空，就通过一些方法（比如方法A）去获取这个属性的值，再重新触发KVO，来更新获取到的新值，不过这里要注意方法A里假如也全是在主线程运行的话，第二次KVO就不会触发，因为KVO是同步运行在主线程上的，这样做可能是为了防止出现死循环吧，这时候就要在方法A里通过后台线程去获取属性的值，然后在主线程里把值赋给Model的属性，这样就能触发KVO了。



###参考资料

1. [KVC 和 KVO](http://objccn.io/issue-7-3/);

2. [Key-Value Observing](http://nshipster.com/key-value-observing/);

3. [[深入浅出Cocoa]详解键值观察（KVO）及其实现机理](http://blog.csdn.net/kesalin/article/details/8194240);

4. [Key-Value Coding Programming Guide](https://developer.apple.com/library/ios/documentation/cocoa/conceptual/KeyValueCoding/Articles/KeyValueCoding.html#//apple_ref/doc/uid/10000107-SW1);

5. [了解 Objective-C 上的 KVO(Key-Value Observing) 機制](http://blog.riaproject.com/objective-c/2147/%E4%BA%86%E8%A7%A3-objective-c-%E4%B8%8A%E7%9A%84-kvokey-value-observing-%E6%A9%9F%E5%88%B6.html);

6. [Objective-C中的KVC和KVO](http://yulingtianxia.com/blog/2014/05/12/objective-czhong-de-kvche-kvo/)
7. [iOS设计模式(01):观察者](http://beyondvincent.com/blog/2013/05/05/18/)；
