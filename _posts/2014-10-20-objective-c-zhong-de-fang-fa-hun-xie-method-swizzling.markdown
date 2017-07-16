---
layout: post
title: "Objective-C 中的方法混写(Method Swizzling)"
date: 2014-10-20 17:47:07 +0800
comments: true
categories: iOS
---
![](/images/201410201751.png)
### 目录

1. 什么是方法混写（Method Swizzling）
2. 如何使用方法混写
3. 注意事项
4. 参考资料

### 什么是方法混写(Method Swizzling)

Objective-C 中对象调用方法，或者叫“消息传递”,是通过一种动态绑定机制实现的，即对象收到消息后，究竟调用哪个方法，完全于运行期决定，而且方法名和其对应的实现也是可以在运行期改变的，这样，我们不需要源代码，也不用通过继承子类的来覆写方法，就能改变这个类中的方法。这种在运行期改变类的方法实现的手段被称之为方法混写（Method Swizzling）.


### 如何使用方法混写

举一个UIAlertView的例子，UIAlertView 有一个 show 的方法，我们想改变这个 show 的方法，可以按如下实现:

UIAlertView+GHSwizzling.m

```objc
#import "UIAlertView+GHSwizzling.h"
#import <objc/runtime.h>

@implementation UIAlertView (GHSwizzling)


+ (void)load
{
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{

        SEL show = @selector(show);
        SEL replaceSHow = @selector(_gh_show);

        Method existingMethod = class_getInstanceMethod(self, show);
        Method newMethod = class_getInstanceMethod(self, replaceSHow);
        method_exchangeImplementations(existingMethod, newMethod);//交换两个方法
    });
}



- (void)_gh_show
{
    [self _gh_show];
    NSLog(@"alert show!");
}
@end
```

从上面代码我们可以看到，实现方法混写的步骤：

1. 编写自己希望实现方法的代码，上述中为 `_gh_show`,我们看到 该方法里调用了自己，好像会陷入递归调用的死循环，不过我们需要知道，这个方法是要和 `show` 方法交换的。

2. 获取自己实现方法的 method 和原实现方法的 method.

3. 通过 `method_exchangeImplementations()` 函数交换两个方法.


注意: 这只是方法混写中的一种，交换两个实现方法，还有 `method_setImplementayion()`等方法.


### 注意事项

1. 方法混写最好在类的 category 中的 `+load` 方法中进行方法交换.

2. 方法混写的代码最好在 dispatch_once 中实现。

3. 要注意给自己实现的方法加上命名空间，以防止出现重复命名的问题，比如上述中的 "_gh_" 前缀.

4. 不要滥用这个方法,该方法被称之为 Objective-C 中的黑魔法，滥用会导致难以调试的bug.

5. 只要在 `+load` 中实现了 `method_exchangeImplementations()`, 那么该类的这个方法在整个应用的生命周期以内都被 Swizzling 了，要想调用原来正常的方法，就只能通过调用自己实现的方法名了。



### 参考资料

1. [Method Swizzling](http://nshipster.com/method-swizzling/)

2. [Objective-C的hook方案（一）: Method Swizzling](http://blog.csdn.net/yiyaaixuexi/article/details/9374411)

3. [[Cocoa]深入浅出Cocoa之Method Swizzling](http://www.cnblogs.com/kesalin/archive/2012/01/05/objc_method_swizzling.html)

4. [Objective-C Method Swizzling](http://sjpsega.com/blog/2014/09/17/oc-method-swizzling/)

5. [关于swizzling](http://billwang1990.github.io/blog/2014/01/04/about-swizzling/)
