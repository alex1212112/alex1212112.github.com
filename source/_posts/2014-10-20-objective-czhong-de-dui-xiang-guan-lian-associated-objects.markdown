---
layout: post
title: "Objective-C 中的对象关联(Associated Objects)"
date: 2014-10-20 12:14:09 +0800
comments: true
categories: 
---
![](/images/201410201218.png)

###目录

1. 什么是对象关联（Associated Objects）
2. 如何使用
3. 注意事项
4. 参考资料

###什么是对象关联(Associated Objects)

对象关联是Objective-C 2.0运行时的一个特性，起始于OS X Snow Leopard和iOS 4。我们有时候需要在对象中存放一些相关的信息，而又无法通过别的手段实现时，就可以把这些信息通过对象关联的方法与对象关联起来，在需要的时候通过对应的 key 就可以获取到对象的这些信息，比如说我们可以在 Category 中通过对象关联添加自定义的属性。对象关联通过  <objc/runtime.h> 中定义的三个方法实现的:

```objc
void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy);

id objc_getAssociatedObject(id object, const void *key);

objc_removeAssociatedObjects(id object)；
```

###如何使用对象关联

我们可以使用关联对象在 Category 中添加自定义的属性，比如为一个 GHPerson 类添加一个体重的属性（weight）:

GHPerson+GHAssociatedObject.h 
 
```objc
#import "GHPerson.h"

@interface GHPerson (GHAssociatedObject)

@property (nonatomic, strong) NSNumber *weight;

@end

```

GHPerson+GHAssociatedObject.m
 
```objc
#import "GHPerson+GHAssociatedObject.h"
#import <objc/runtime.h>

static char weightKey;

@implementation GHPerson (GHAssociatedObject)
@dynamic weight;

- (void)setWeight:(NSNumber *)weight
{
    objc_setAssociatedObject (self,&weightKey,weight,OBJC_ASSOCIATION_RETAIN);

}

- (NSNumber *)weight
{
   return (NSNumber *)objc_getAssociatedObject(self, &weightKey);
}
@end
```
其中 weightKey 是我们关联对象时用到的key，一般最好使用一个 char 类型的静态全局变量。从关联这个角度上看，我们用的属性不过是把对象关联到了一个实例变量上面。

我们看到 objc_setAssociatedObject 中的第 4 个参数，它表明了一个引用的关系，和 @property 中的属性等同：

> OBJC_ASSOCIATION_ASSIGN   等同于 @property (assign) 或 @property (unsafe_unretained)

> OBJC_ASSOCIATION_RETAIN_NONATOMIC	等同于 @property (nonatomic, strong),其不可以被原子化

> OBJC_ASSOCIATION_COPY_NONATOMIC	    等同于 @property (nonatomic, copy),其不可以被原子化


> OBJC_ASSOCIATION_RETAIN  等同于  @property (atomic, strong),  可以被原子化


> OBJC_ASSOCIATION_COPY  等同于  @property (atomic, copy), 可以被原子化





###注意事项

1. 不应该用 objc_removeAssociatedObjects() 来删除对象的属性，因为可能会导致其他客户对其添加的属性也被移除了。规范的方法是：调用 objc_setAssociatedObject 方法并传入一个 nil 值来清除一个关联。
2. 应该小心使用对象关联，不要滥用，因为可能会引起难以调试的bug。


###参考资料

1. [Associated Objects](http://nshipster.cn/associated-objects/)
