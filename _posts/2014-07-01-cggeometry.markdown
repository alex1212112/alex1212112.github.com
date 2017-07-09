---
layout: post
title: "CGGeometry"
date: 2014-07-01 12:35:54 +0800
comments: true
categories: iOS
---

![](/images/201407011244.png)

CGGeometry 是一个 Quartz 2D 框架中非常有用且好用的处理几何问题的基本组件，这里列出一些它的方法并简单说明如何使用。

###变换

####CGRectOffset

CGRectOffset: 返回一个原点在源矩形基础上进行了偏移的矩形。

```objc
CGRect CGRectOffset(
  CGRect rect, //源矩形
  CGFloat dx,  //x方向偏移距离
  CGFloat dy   //y方向偏移距离
)
```

为什么要用 CGRectOffset ？[NSHipster](http://nshipster.cn/cggeometry/)解释如下:

> 它不仅能让你在同时改变水平和垂直位置的时候减少一行代码，更重要的是，它所表示的平移比直接分开操作原点的值更具有几何意义。


####CGRectInset

CGRectInset: 返回一个与源矩形共中心点的，或大些或小些的新矩形。

```objc
CGRect CGRectInset(
  CGRect rect, //源矩形
  CGFloat dx,  // x方向左边减去dx，右边也减去dx（共2dx）
  CGFloat dy   // y方向上边减去dy，右边也减去dy（共2dy）
)
```

如果用 CGRectInset 作为缩放矩形的快捷方法，一般通用的做法是嵌套调用CGRectOffset，把CGRectInset的返回值作为CGRectOffset的参数。

如：

```objc
    CGRect frame ＝ CGRectOffset(CGRectInset(rect, 10.0f, 10.0f), 10.0f, 10.0f);
```

上述代码把源矩形rect的大小缩小了10，然后又向右平移了10，向下平移了10，


####CGRectIntegral

CGRectIntegral: 返回包围源矩形的最小整数矩形。

```objc
CGRect CGRectIntegral (
  CGRect rect
)
```

CGRectIntegral 用来对矩形取整，可以保证矩形对齐到像素边界，在非retina屏幕上能防止像素模糊。


###取值辅助函数

####CGRectGet[Min|Mid|Max][X|Y]

```objc
CGFloat  CGRectGetMinX (CGRect rect) //获取矩形x坐标的最小值
CGFloat  CGRectGetMinY (CGRect rect) //获取矩形y坐标的最小值
CGFloat  CGRectGetMidX (CGRect rect)//获取矩形x坐标的中间值
CGFloat  CGRectGetMidY (CGRect rect) //获取矩形y坐标的中间值
CGFloat  CGRectGetMaxX (CGRect rect) // 获取矩形x坐标的最大值
CGFloat  CGRectGetMaxY (CGRect rect)） // 获取矩形y坐标的最大值
```

引用[NSHipster](http://nshipster.cn/cggeometry/)说明如下：

> 用这些函数代替诸如frame.origin.x + frame.size.width之类的代码将更加清晰、语义上更为生动的（特别是用取中间和取最大函数）

####CGRectGet[Width|Height]

```objc
CGFloat  CGRectGetHeight (CGRect rect) //获取矩形的高
CGFloat  CGRectGetWidth  (CGRect rect)  //获取矩形的宽
```

###常量

####CGRectZero

```objc
const CGRect CGRectZero;
```

一个原点在(0, 0)，且长宽均为 0 的常数矩形。这个零矩形与 CGRectMake(0.0f, 0.0f, 0.0f, 0.0f) 是等价的。当我们初始化一个视图时，它们的边框通常设置为CGRectZero，把具体的布局放到 -layoutSubviews中。

####CGRectNull

```objc
const CGRect CGRectNull；
```

空矩形。这个会在，比如说，求两个不相交的矩形的相交部分时返回。注意，空矩形不是零矩形。


####CGRectInfinite

```objc
const CGRect CGRectInfinite；
```

无穷大矩形,它与所有的点或矩形相交，包含所有矩形，且它与任何矩形的并集等于它自身。可以用 CGRectIsInfinite 来检查一矩形是否为无限大。


###分割矩形

####CGRectDivide

CGRectDivide: 将源矩形分为两个子矩形。

```objc
void CGRectDivide(
  CGRect rect,
  CGRect *slice,
  CGRect *remainder,
  CGFloat amount,
  CGRectEdge edge
)
```

CGRectDivide 用以下方式将矩形分割为两部分：

* 传入一个矩形并选择一条边（edge）（上，下，左，右）；

* 平行那个边在矩形里量出amount的长度；

* 从edge 到量出的amount区域都保存到slice 参数中；

* 剩余的部分保存到remainder 参数中。


其中 edge 参数是一个CGRectEdge 枚举类型：


```objc
enum CGRectEdge {
   CGRectMinXEdge, //矩形的左边
   CGRectMinYEdge, //矩形的上边
   CGRectMaxXEdge, //矩形的右边
   CGRectMaxYEdge  //矩形的下边
}
```

###比较

判断两个点是否相等

```objc
bool  CGPointEqualToPoint (CGPoint A,CGPoint B)  
```  

 CGSizeAB是否相等

```objc
bool  CGSizeEqualToSize (CGSize A，CGSize B)
```

矩形AB的位置大小是否相等

```objc
bool  CGRectEqualToRect (CGRect A，CGRect B)
```   
矩形AB是否相交

``` objc   
bool  CGRectIntersectsRect (CGRect A，CGRect B)
```

###包含关系

检测矩形A是否包含指定的点B

```objc
bool  CGRectContainsPoint (CGRect A, CGPoint B)
```     

检测矩形A是否包含矩形B

```objc
bool  CGRectContainsRect (CGRect A，CGRect B)  
```

###检测矩形是否存在或是无穷大

矩形A是否长和宽都是0，或者是个NULL

```objc
bool  CGRectIsEmpty (CGRect A)
```
矩形A是否为NULL

```objc      
bool  CGRectIsNull (CGRect A)
```
矩形A是否无穷大，没有边界

```objc
bool  CGRectIsInfinite (CGRect A)
```        

###参考资料

[NSHipster](http://nshipster.cn/cggeometry/);

[Objective-c 中CGGeometry几何类常用方法简单整理](http://www.cnblogs.com/xuling/archive/2012/02/09/2343427.html);

[CGGeometry Reference](https://developer.apple.com/library/mac/documentation/GraphicsImaging/Reference/CGGeometry/Reference/reference.html);
