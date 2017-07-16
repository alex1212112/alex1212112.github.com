---
layout: post
title: "UIKit 动力学(UIKit Dynamics)学习"
date: 2014-06-11 11:17:46 +0800
comments: true
categories: iOS
---

![](/images/4908ab0eae4b946b9066eafd11f8030580c2c4f79dfd.png)
#### 简介

UIKit dynamics(UIKit动力学)是UIKit的一套动画和交互体系。和我们用的CoreAnimation或者UIView animations一样，只不过UIKit 动力学将现实世界动力驱动的动画引入了UIKit，比如重力，铰链连接，碰撞，悬挂、吸附等效果。

#### 基本概念和元素

* UIdynamicItem: 实现了UIDynamicItem委托的对象，它是动画的执行者，相关的物理属性也是加在这个对象上，由于iOS7之后，UIView和UICollectionViewLayoutAttributes类默认实现了该协议，所以UIView就可以看作是一个UIdynamicItem；

* UIDynamicBehavior：动力行为的描述，用来指定UIDynamicItem应该如何运动，即定义适用的物理规则。一般我们使用这个类的子类对象来对一组UIDynamicItem应该遵守的行为规则进行描述，在iOS7中，系统默认提供了如下的动力行为：

	1、UIGravityBehavior：重力行为

	2、UICollisionBehavior：碰撞行为

	3、UIAttachmentBehavior：铰链行为，两个物体连接的状态，可以模拟无形变或者弹性行遍的情况

	4、UISnapBehavior：吸附行为，将UIView通过动画吸附到某个点上。

	5、UIPushBehavior：推行为，比如为一个UIView施加一个力的作用，我们可以指定力的大小，方向和租用点等信息。

	6、UIDynamicItemBehavior：动力学辅助元素行为，系统有一组自定义的默认值

* UIDynamicAnimator；动画的播放者，动力行为（UIDynamicBehavior）的容器，添加到容器内的行为将发挥作用；

* ReferenceView：力学参考系，只有当想要添加力学的UIView是ReferenceView的子view时，动力UI才发生作用，比如你可以把ViewController的view作为ReferenceView；

#### 代码示例（一个简单的重力行为）

```objc
/**
 *  初始化subView
 */
- (void)initSubView
{
    _subView = [[UIView alloc] initWithFrame:(CGRect){

        .origin.x =   100.0f,
        .origin.y =   108.0f,
        .size.width = 100.0f,
        .size.height = 100.0f,
    }];
    _subView.backgroundColor = [UIColor lightGrayColor];

    [self.view addSubview:_subView];
}
/**
 *  简单的重力行为
 *
 *  @param sender leftButton
 */
- (void)leftButtonClicked:(id)sender
{

    _subView.frame =  (CGRect){

        .origin.x =   100.0f,
        .origin.y =   108.0f,
        .size.width = 100.0f,
        .size.height = 100.0f,

    };

    if (!_animator)
    {
        // 1 假如animator为空的话，就以当前ViewController的View为参考View初始化一个animator
        _animator = [[UIDynamicAnimator alloc] initWithReferenceView:self.view];
    }

        //  移除animator上所有的力学行为
    [_animator removeAllBehaviors];

        // 2 初始化重力行为
    UIGravityBehavior *gravityBeahvior = [[UIGravityBehavior alloc] initWithItems:@[_subView]];

       // 3 添加重力行为到animator上，使重力行为生效
    [_animator addBehavior:gravityBeahvior];

}
```

代码中关键的地方已经用注释表示出来，在制作一个简单的UIKit 动力学行为的的步骤如下：

1. 通过参考系(ReferenceView)来初始化一个UIDynamicAnimator；
2. 初始化一个力学行为，上述代码中是一个重力行为UIGravityBehavior；
3. 把配置好的力学行为添加到animator中；

表现出来的效果如下图：

![](/images/201406111150gravity.gif)

####代码示例（带碰撞的重力行为）

```objc
/**
 *  带重力的碰撞行为
 *
 *  @param sender middleButton
 */
- (void)middleButtonClicked:(id)sender
{

    _subView.frame =  (CGRect){

        .origin.x =   100.0f,
        .origin.y =   108.0f,
        .size.width = 100.0f,
        .size.height = 100.0f,

    };

    if (!_animator)
    {
        _animator = [[UIDynamicAnimator alloc] initWithReferenceView:self.view];
    }

    [_animator removeAllBehaviors];

    //初始化一个碰撞的行为
    UICollisionBehavior *collisionBehavior = [[UICollisionBehavior alloc] initWithItems:@[_subView]];

    //配置碰撞的边界，下述代码设置碰撞边界为参考系(self.view)的边框作为碰撞边界
    collisionBehavior.translatesReferenceBoundsIntoBoundary = YES;

    [_animator addBehavior:collisionBehavior];

    //重力行为
    UIGravityBehavior *gravityBeahvior = [[UIGravityBehavior alloc] initWithItems:@[_subView]];
    [_animator addBehavior:gravityBeahvior];

}
```
表现出来的效果如下图：

![](/images/201406111150gravityCollision.gif)

注：碰撞行为有自己的delegate回调，可以帮助我们了解碰撞的具体情况，包括哪个item和哪个item开始发生碰撞，碰撞接触点是什么，是否和边界碰撞，和哪个边界碰撞了等信息。参见:[UICollisionBehavior](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UICollisionBehaviorDelegate_Protocol/Reference/Reference.html).

####代码示例（吸附行为）

```objc
/**
 *  吸附行为
 *
 *  @param sender rightButton
 */
- (void)rightButtonClicked:(id)sender
{

    _subView.frame =  (CGRect){

        .origin.x =   100.0f,
        .origin.y =   108.0f,
        .size.width = 100.0f,
        .size.height = 100.0f,

    };

    if (!_animator)
    {
        _animator = [[UIDynamicAnimator alloc] initWithReferenceView:self.view];
    }

    [_animator removeAllBehaviors];

    //初始化一个吸附行为
    UISnapBehavior *snap = [[UISnapBehavior alloc] initWithItem:_subView snapToPoint:(CGPoint){100.0f,300.0f}];
    //添加吸附行为到animator
    [_animator addBehavior:snap];
}
```
表现出来的效果如下图：

![](/images/201406111151snap.gif)

####下载

[Demo下载地址](https://github.com/alex1212112/DynamicsDemo)

####参考资料
[UIKit Dynamics入门](http://onevcat.com/2013/06/uikit-dynamics-started/)

[iOS7新特征汇总[05]初窥UIKit动力(UIKit Dynamics)](http://beyondvincent.com/blog/2013/06/16/88/)

[UIKit-力学教程](http://www.raywenderlich.com/zh-hans/52617/uikit-%E5%8A%9B%E5%AD%A6%E6%95%99%E7%A8%8B)
