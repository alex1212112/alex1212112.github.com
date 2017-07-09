---
layout: post
title: "关于对象等同性判断"
date: 2014-10-19 20:20:26 +0800
comments: true
categories: iOS 
---
![](/images/201410192024.png)

我们经常会遇到判断两个对象是否相等的的情况，如果食用 == 操作符来进行判断，比较出来多结果不一定是我们想要的，因为 == 操作符比较的事两个对象对指针，而不是对象的值。 NSObject 协议中有两个用语判断等同性的关键方法：

```objc
- (NSUInteger)hash;
- (BOOL)isEqual:(id)object;
```

NSObject类对其的默认实现为当且仅当其指针完全相同的时候，两个对象才相等。我们要实现自己的对象等同性就要覆写这两个方法。 有一点要理解，如果 "isEqual:" 方法判断两个对象相等，那么其 hash 方法也必须返回同一个值，但是，如果两个对象的 hash 方法返回同一个值， "isEqual:" 方法却未必认为二者相等。

假如有一个Person类如下:

```objc
@interface Person : NSObject

@property (nonatomic, strong) NSString *name;
@property (nonatomic, strong) NSNumber *age;

@end
```

我们实现其等同性判断的步骤如下：

1. 编写一个等同性判定方法，类似 NSString 类中的 `isEqualToString:` 方法，我们这里可以声明为 `isEqualToPerson:`.
2. 覆写 `isEqual:` 方法和 `hash` 方法.


具体实现如下：

```objc
- (BOOL)isEqualToPerson:(Person *)person
{
    if (self == person)
    {
        return YES;
    }
    if (!([_name isEqualToString:person.name] || _name == person.name))
    {
        return NO;
    }

    if (!([_age isEqualToNumber:person.age] || _age == person.age))
    {
        return NO;
    }

    return YES;

}

- (BOOL)isEqual:(id)object
{
    if ([self class] == [object class])
    {
        return [self isEqualToPerson: (Person *)object];
    }
    else
    {
        return [super isEqual:object];
    }
}

- (NSUInteger)hash
{
    return [_name hash] ^ [_age hash];
}

```

至此，我们已经完成了符合自己需求的对象等同性判断。


###参考资料

1. [值对象](http://www.objccn.io/issue-7-2/)
