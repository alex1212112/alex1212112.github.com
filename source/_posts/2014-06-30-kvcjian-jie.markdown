---
layout: post
title: "KVC简介"
date: 2014-06-30 18:50:23 +0800
comments: true
categories: 
---

![](/images/2d7462592d67a98dbffc700f70ae01d4423b1c1256049.png)

###目录

什么是KVC？
	
为什么要用KVC？
	
KVC用法介绍.
	
	
###什么是KVC？


KVC是cocoa的一部分，可以使我们在访问对象属性的时候不需要再调用 setter 和 getter存取器，比如我们可以用 [object valueForKey:property]来访问object对象中的property属性，也能用 [object setValue:value forKey:property]来给object对象中的property属性赋值。为了达到这种目的，对象需要用特定的方式来命名方法，这种命名约定就成为KVC.

###为什么要用KVC

使用KVC能让我们在运行时而非编译时决定访问哪个属性，从而得到更灵活和更易于重用的对象，同时也能帮助我们减少代码量,通过KVC,还能实现cocoa中更为强大的KVO功能。

###KVC用法介绍

我们定义一个对象如下

```objc
@interface People : NSObject
@property (nonatomic, strong) NSString *name;
@property (nonatomic, strong) NSString *age;
@property (nonatomic, strong) Car *car;
@end
```

####访问对象属性

我们在获取people对象的name属性的值的时候，就可以通过KVC来获取：

```objc
 NSString *name = [people valueForKey:@"name"];
```

此代码基本等同于下面代码

```objc
NSString *name = people.name;
```
####用KVC赋值

KVC可以用setValue:forKey:修改可写属性：

```objc
[people setValue:@"Alex" forKey:@"name"];
```
此代码基本等同与下面代码

```objc
people.name = @"Alex";
```
####keyPath

KVC 方法有 key 和keyPath 两个版本，比如 valueForkey: 和 valueForKeyPath:版本，这两者的区别在于，后者可以包含嵌套关系，用点分开，valueForKeyPath方法可以遍历所有的关系，如下所示：

```objc
	NSString *carName = [people valueForKayPath:@"car.name"];
```
此方法用来获取people对象的车的名称，可以基本等价于下面代码:

```objc
	NSString *carName = [[people car] name];
```
而 key方法不会遍历关系，假如你使用

```objc
	NSString *carName = [people valueForKay:@"car.name"];
```
程序会去获取 people对象 的 car.name 属性，很明显，people 没有这样的属性，所以系统会抛出异常。

####KVC和非对象

KVC 的 valueForKey:方法总是返回一个id对象，但不是每一个方法都会返回对象，那么对于标量，该方法返回值会自动用NSValue 或 NSNumber 来进行封装。因此我们在通过 KVC 赋值的对象为标量的时候，也应该先用 NSValue 或 NSNumber 进行封装，然后再使用 setValue:forKey:方法。

####高阶消息传递

valueForKey:有很多有用的特例，比如对于 NSArray 或 NSSet 等容器类，使用 valueForKey:方法，实际上该方法会被传递给容器中的每一个对象，而不是对容器本身进行操作，它会对容器中的每个对象来查找这个键值，然后将查询结果打包到另一个容器中并返回给你。这样，我们就很容易用一个容器对象创建另一个容器对象。

比如：

```
People *developer = [[People alloc] init];
[developer setValue:@"Alex" forKey:@"name"];

People *teacher = [[People alloc] init];
[teacher setValue:@"Lucy" forKey:@"name"];
	
NSArray *array = @[developer,teacher];
	
NSArray *nameArray = [array valueForKey:@"name"];
	
```
name 被传递给array中的每一个元素，并返回一个新的数组，新的数组中的元素是一个姓名的字符串。
对于 keyPath 使用方法也类似：

```objc
	NSArray *nameLengthArray = [array valueForKeyPath:@"name.length"];
```
新的数组中的元素是用 NSNumber 封装的姓名字符串的长度。

####通过KVC创建Model

我们经常会遇到需要将一个字典转化成一个对象情况，这时用 KVC 能很好的解决问题， KVC 中有一个setValuesForKeysWithDictionary: 方法，此方法能很好的把字典转换成我们需要的对象。比如我们可以给刚才的 People 类增加一个初始化方法:

```objc
-(id) initWithDictionary:(NSMutableDictionary*) jsonObject;
```

实现如下：

```objc
-(id) initWithDictionary:(NSMutableDictionary*) jsonObject
{
    if((self = [super init]))
    {
        [self setValuesForKeysWithDictionary:jsonObject];
    }
    return self;
}
```

####一些特殊的用法

KVC还提供一些特殊的方法，例如获取一组值的平均值或返回这组值的最小值和最大值。例如：

```
NSArray *array = @[developer,teacher];
NSNumber *totalLength = [array valueForKeyPath:@"name.@sum.length"]; 	
```
@sum是一个操作符，对指定的属性（name.length）求和。
 
