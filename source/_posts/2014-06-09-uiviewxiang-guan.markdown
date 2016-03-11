---
layout: post
title: "UIView 相关"
date: 2014-06-09 10:21:49 +0800
comments: true
categories: 
---

![](http://ww3.sinaimg.cn/large/8bcaa2dfjw1ec8pxpionej20dw07sdgj.jpg)

#### 问题描述:
 	
有时候设置subView的center如下：
 	  
 	  
```objc
subView.center = view.center;
```	
 	

但实际显示出来发现subView的位置不对，该如何处理？
 
####解决办法:
 	  	
 改为
 
```objc
subView.center = CGPointMake(view.frame.size.width / 2, 
view.frame.size.height / 2);
``` 	  
 	 
不要使用view.center,除非view和subView共用同一个superView,center这个属性指的是子view的中心点在父view中的位置,是以父view的坐标系为参考的；
     

*** 
 	
 	
####问题描述:
 	
UIView在自己实现draw方法时候，为什么整个view的背景都变成黑色？
 
####解决办法:
 	 
设置下view的backgroundColor就OK了。
 	 
说明：当view的backgroundColor为nil并且opaque属性为YES，自己实现draw方法，view的背景颜色就会变成黑色。
     
*** 
 	
####问题描述:
 	
UIView 通过layer 的 shadowColor、shadowOpacity、shadowOffset、shadowRadius 几个属性可以很方便的为 UIView 添加阴影效果。但是在添加了阴影后，会出现动画卡顿的现象，如何解决？

####解决办法:
 	 
为阴影指定路径，即设置 layer 的 shadowPath 属性。如下：
 	 
``` objc	
view.layer.shadowPath = [UIBezierPath  bezierPathWithRect:view.bounds].CGPath;
```
 	 
说明：不指定阴影路径时，绘制阴影会产生大量的 Offscreen-Rendered 。而 Offscreen-Rendered（离屏渲染）和 Blending（混合）是 iOS 绘图中对性能影响比较大的两方面。
 	 
参考：[绘制阴影引发的 iOS 绘图性能问题总结](http://blog.devdlh.com/blog/2013/03/18/performance-problerm-caused-by-shadowpath/)
     
