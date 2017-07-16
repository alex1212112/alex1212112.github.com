---
layout: post
title: "Foundation 框架中集合类的一些用法-NSArray"
date: 2014-10-18 21:21:07 +0800
comments: true
categories: iOS
---
![](/images/201410191739.png)

### 目录

1. 遍历
2. 排序
3. 过滤
4. 其他
5. 参考资料

### 遍历

#### 普通for循环

```objc
 for (NSInteger n = 0 ; n < [array count]; n ++)
    {

    }
```

#### for in 方式（NSFastEnumeration方式）

该方式无法获取到数组的下标，在不需要下标且数组元素很多的时候，可以用这种方式

```objc
 for (id object in array)
    {

    }
```
#### enumerateObjectsUsingBlock方式

数组遍历的 block 方式 ，其中的stop标志提供了一个优雅的停止遍历的方法，当 *stop = Yes 的时候，遍历就会停止

```objc
 [array enumerateObjectsUsingBlock:^(id object, NSUInteger idx, BOOL *stop) {

    }];
```


#### enumerateObjectsWithOptions方式

option 设置为 NSEnumerationConcurrent 为并发遍历方式,

```objc
[array enumerateObjectsWithOptions:NSEnumerationConcurrent usingBlock:^(id obj, NSUInteger idx, BOOL *stop) {

    }];
```
这里要注意使用并发方式进行遍历的话，最好不要在block里使用 NSMutableArray 之类的非线程安全的对象，当引起读写冲突的时候，会抛出 double free 的异常。


option 设置为 NSEnumerationReverse 为倒序遍历方式

```objc
[array enumerateObjectsWithOptions:NSEnumerationReverse usingBlock:^(id obj, NSUInteger idx, BOOL *stop) {

    }];            
```


### 排序

假如数组里要排序的对象如下；

```objc
@interface User : NSObject

@property (nonatomic, copy) NSString *name;
@property (nonatomic, copy) NSNumber *age;

@end
```


#### 使用NSDescriptor进行排序

按年龄和姓名进行排序,该方法可以对属性的多个key进行排序

```objc
NSArray *sortDescriptors = @[[NSSortDescriptor sortDescriptorWithKey:@"age" ascending:YES],[NSSortDescriptor sortDescriptorWithKey:@"name" ascending:YES]];

NSArray *sortedArray =  [array sortedArrayUsingDescriptors:sortDescriptors];
```

#### 使用NSComparator进行排序

```objc
NSArray *sortedArray = [array sortedArrayUsingComparator:^NSComparisonResult(User *user1, User *user2){

    return [user1.age compare:user2.age];

}];
```

#### 使用selctor


```objc
NSArray *sortedArray = [array sortedArrayUsingSelector:@selector(compare:)];

```

其中compare为 User对象的方法

```objc
- (NSComparisonResult)compare:(User *)otherUser {

    return [self.age compare:otherUser.age];
}

```

### 过滤

有时候需要按照一定规则从数组中过滤出符合要求的元素，这时候就要用到NSPredicate了,常用的格式化的用法如下：

```objc
NSPredicate *predicate = [NSPredicate predicateWithFormat:@"((age >= %@) AND (age < %@))",@20,@30];

NSArray *result = [array filteredArrayUsingPredicate:predicate];
```
这个便返回了 user 数组中 age 在 20 到 30 之间的元素。

NSPredicate 也有个 block 用法

```objc
NSPredicate *predicate = [NSPredicate predicateWithBlock:^BOOL(id evaluatedObject, NSDictionary *bindings) {

        return  ((User *)evaluatedObject).age >= 20 && ((User *)evaluatedObject).age <= 30;

    }];

NSArray *result = [array filteredArrayUsingPredicate:predicate];
```



### 其他

#### 去重

有一个优雅的KVC用法

```objc
NSArray *result = [array valueForKeyPath:@"@distinctUnionOfObjects.self"];
```
别忘了前面 distinctUnionOfObjects 前面的 @ 符号,假如只想返回一个由user数组中的姓名组成的不重复的新数组，可以

```objc
NSArray *result = [array valueForKeyPath:@"@distinctUnionOfObjects.name"];
```

还要注意假如数组里面存储的是我们自己的对象多话，这个对象还要实现等通性判断的方法：

```objc
- (BOOL)isEqual:(id)object;
- (NSUInteger)hash;
```
假如不实现等同性判断的话，系统默认的会把两个对象的指针相同才认为是同一个对象，有时候这显然不是我们想要的. 对象等同性具体实现可参考[对象等同性判断](http://alex1212112.github.io/blog/2014/10/19/guan-yu-dui-xiang-deng-tong-xing-pan-duan/)。


### 参考资料

1. [对 NSArray 中自定义的对象进行排序](http://beyondvincent.com/blog/2014/01/26/how-to-sort-nsarray-with-custom-objects/)

2. [基础集合类](http://objccn.io/issue-7-1/)

3. [ios中集合遍历方法的比较和技巧](http://blog.sunnyxx.com/2014/04/30/ios_iterator/)

4. [NSPredicate](http://nshipster.cn/nspredicate/)

5. [KVC Collection Operators](http://nshipster.cn/kvc-collection-operators/)
