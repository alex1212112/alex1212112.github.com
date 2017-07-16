---
layout: post
title: "iOS8 中的实时模糊 View－UIVisualEffectView"
date: 2015-05-10 13:08:00 +0800
comments: true
categories: iOS
---
![](/images/201505101313.png)

### 介绍

在iOS7 以后，整个扁平化的 iOS 界面都大量采用了高斯模糊的效果来展示层次，比如在主界面上从底面上划出来 的控制面板，或者从顶部向下划出来的通知栏界面，都实时模糊了主界面背景，在 iOS7 的时候我们总共有 3.5 个办法来实现这些效果：

1. 利用苹果开源的 UIImage+ImageEffects  
2. 利用 GPUImage 的高斯模糊滤镜
3. 利用CoreImage 的高斯模糊滤镜

在 iOS7 初期的时候还有人使用 UIToolBar 覆盖在 superView 上的方法来实现，但是后来这个方法失效了，所以只能算半个。不过到了 iOS8，苹果提供了一个非常好用的方案－UIVisualEffectView。

UIVisualEffectView 是 UIView 的子类，因此使用的时候就是实例化一个 UIVisualEffectView，然后添加到superView 上就 OK 了。 That‘s it，就这么简单。


UIVisualEffectView 有两种效果，一种是 UIBlurEffect，即高斯模糊效果，一种是 UIVibrancyEffect，这种效果能把背景 View 和 当前 View 混合起来，通常使用的时候要在模糊后的 View 上 显示一些内容的话，就要在 UIBlurEffect 的 View 上 再添加一层 UIVibrancyEffect，这样可以使整个显示效果更舒适一点，下面这张图展示了在背景图片完全相同的情况下使用和不使用 Vibrancy 的不同:

![](/images/10310Q192-19.png)

### 代码

Objectivie-C

```objc
    //初始化一个 UIVisualEffect，UIBlurEffect 是 UIVisualEffect 的子类
    UIBlurEffect *blurEffect = [UIBlurEffect effectWithStyle:UIBlurEffectStyleExtraLight];

    //初始化 模糊效果的 UIVisualEffectView
    UIVisualEffectView * blurEffectView = [[UIVisualEffectView alloc] initWithEffect:blurEffect];

    //设置 blurEffectView 的 frame
    blurEffectView.frame = self.view.bounds;

    //添加 blurEffectView 到 self.View 上,如果只是做模糊效果，那么到这一步已经完了
    [self.view addSubview:blurEffectView];

    //初始化 Vibrancy 类型的 UIVisualEffect，vibrancyEffect 也是UIVisualEffect 的子类，而且初始化这个 Effect 需要用到 blurEffect，我们就还用上面的好了，假如重新初始化一个 blurEffect，要注意这个新的 blurEffect 的 style 要和上面的保持一致，不然就实现不了 Vibrancy效果
    UIVibrancyEffect *vibrancyEffect = [UIVibrancyEffect effectForBlurEffect:blurEffect];

    //初始化 Vibrancy 效果的 UIVisualEffectView
    UIVisualEffectView *vibrancyEffectView = [[UIVisualEffectView alloc] initWithEffect:vibrancyEffect];

    //设置 vibrancyEffectView 的frame
    vibrancyEffectView.frame = blurEffectView.bounds;


    //注意是 blurEffectView.contentView. 苹果官方注释说了  Do not add subviews directly to UIVisualEffectView, use this view instead
    [blurEffectView.contentView addSubview:vibrancyEffectView];


    //初始化一个 label
    UILabel *label = [[UILabel alloc] initWithFrame:vibrancyEffectView.bounds];

    label.textAlignment = NSTextAlignmentCenter;

    label.text = @"Live long and Prosper";

    label.font = [UIFont fontWithName:@"SnellRoundhand-Black" size:29.0f];

    //添加 label
    [vibrancyEffectView.contentView addSubview:label];
```


### 参考资料

1. [UIVisualEffectView Tutorial](http://www.raywenderlich.com/84043/ios-8-visual-effects-tutorial)
