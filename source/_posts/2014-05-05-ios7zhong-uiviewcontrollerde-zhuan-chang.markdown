---
layout: post
title: "iOS7中UIViewController的转场"
date: 2014-05-05 18:57:40 +0800
comments: true
categories: 
---


iOS7提供了一套供开发者方便自定义Viewcontroller间切换动画的API，我们可以利用这些API来自定义自己的转场效果，替代系统提供的默认push、present等动画效果。

## 实现流程 

####首先介绍presentViewController转场动画的实现，主要流程如下：


* 自定义一个转场动画效果。

* 在 presentingViewConttroller 里实现 UIViewControllerTransitioningDelegate 代理	方法，主要有两个，分别是present时候的动画和disMiss时候的动画，
	
present时候要实现的方法：

``` objc		
- (id <UIViewControllerAnimatedTransitioning>)animationControllerForPresentedController:(UIViewController *)presented presentingController:(UIViewController *)presenting sourceController:(UIViewController *)source;
```	
	
disMiss时候要实现的方法：

``` objc
- (id <UIViewControllerAnimatedTransitioning>)animationControllerForDismissedController:(UIViewController *)dismissed;
```	
		
		
这两个方法都返回刚才第一步自定义的转场动画。
	
* 在present的时候设置presentedViewController 的 transitioningDelegate如下：

``` objc	
UIViewController *presentedVC = [[UIViewController alloc] init];
presentedVC.transitioningDelegate = self;
[self presentViewController:presentedVC animated:YES completion:nil];
```

#### 关于navigationController的push、pop的动画流程如下：

* 自定义一个转场动画效果。

* 实现UINavigationControllerDelegate的代理方法，这里就一个

``` objc
- (id <UIViewControllerAnimatedTransitioning>)navigationController:(UINavigationController *)navigationController
                                   animationControllerForOperation:(UINavigationControllerOperation)operation
                                                fromViewController:(UIViewController *)fromVC
                                                  toViewController:(UIViewController *)toVC ;                                                 
```    
                       
这个方法也返回第一步里自定义的转场动画。为什么viewController有present 和disMiss两个方法，为什么这里没有push 和pop 两个方法呢，那是因为在这个代理方法里可以通过(UINavigationControllerOperation)operation 参数来确认当前操作究竟是push 还是pop， 当`operation == UINavigationControllerOperationPush`时候 就是push操作，当`operation == UINavigationControllerOperationPop` 时候就是pop操作。
                                                                          

*  在初始化时候设置NavigationController的delgate如下:

``` objc
self.navigationController.delegate = self;
```	
		
### 自定义动画和相关API介绍

上面流程里第一步转场动画效果该如何实现呢？？？

 
我们要自定义一个继承于NSObject的类，此类要实现UIViewControllerAnimatedTransitioning协议，并实现其代理方法，代理方法主要有两个，其中一个返回整个动画花费的时间，如下：

``` objc
- (NSTimeInterval)transitionDuration:(id <UIViewControllerContextTransitioning>)transitionContext
```

假如我们动画时间为1秒，我们就可以在这样实现此方法

``` objc
- (NSTimeInterval)transitionDuration:(id <UIViewControllerContextTransitioning>)transitionContext
		{
			return 1.0f;
		}
``` 

另一个就是具体动画实现的方法，如下：

``` objc
- (void)animateTransition:(id <UIViewControllerContextTransitioning>)transitionContext;
```

其中	transitionContext可以理解为动画切换的上下文，它是一个实现了 `UIViewControllerContextTransitioning`协议的NSObject，它提供了VC切换的一切必需内容，如从哪个VC到哪个VC，动画切换的容器等，这个NSObject里的几个比较重要的方法如下：
	
   * `-(UIView *)containerView; `

	VC切换所发生的view容器，开发者应该将切出的view移除，将切入的view加入到该view容器中。
	
   * `-(UIViewController )viewControllerForKey:(NSString )key; `

	提供一个key，返回对应的VC。现在的SDK中key的选择只有UITransitionContextFromViewControllerKey和UITransitionContextToViewControllerKey两种，分别表示将要切出和切入的VC。

   * `-(CGRect)initialFrameForViewController:(UIViewController *)vc; `

	某个VC的初始位置，可以用来做动画的计算。

   * `-(CGRect)finalFrameForViewController:(UIViewController *)vc;`

	与上面的方法对应，得到切换结束时某个VC应在的frame。

   * `-(void)completeTransition:(BOOL)didComplete; `

	向这个context报告切换已经完成。
	
	
下面是一个该方法的简单实现

``` 
- (void)animateTransition:(id <UIViewControllerContextTransitioning>)transitionContext
	{
    // 1. Get controllers from transition context
    UIViewController *toVC = [transitionContext viewControllerForKey:UITransitionContextToViewControllerKey];
 
    // 2. Set init frame for toVC
    
    CGRect screenBounds = [[UIScreen mainScreen] bounds];
    CGRect finalFrame = [transitionContext finalFrameForViewController:toVC];
    
    UIView *toViewSnapshot = [toVC.view resizableSnapshotViewFromRect:toVC.view.frame afterScreenUpdates:YES withCapInsets:UIEdgeInsetsZero];
 	
    toViewSnapshot.frame = CGRectOffset(finalFrame, 0, screenBounds.size.height);
    
    UIView *toView = toVC.view;
    
    // 3. Add toVC's view to containerView
    UIView *containerView = [transitionContext containerView];
    [containerView addSubview:toViewSnapshot];
    
    // 4. Do animate now
    NSTimeInterval duration = [self transitionDuration:transitionContext];
    [UIView animateWithDuration:duration
                          delay:0.0
         usingSpringWithDamping:0.4
          initialSpringVelocity:10.0
                        options:UIViewAnimationOptionCurveEaseInOut
                     animations:^{
                         toViewSnapshot.frame = finalFrame;
                     } completion:^(BOOL finished) {

                         if ([transitionContext transitionWasCancelled]) {
                            
                         } else {
							//5. add the real toView and remove the snapshot
                             [containerView addSubview:toView];
                             for (UIView *view in containerView.subviews) {
        							if (view != toView) 
        								{
            								[view removeFromSuperview];
        								}
    }

                         }
                         //6. transition is finished
                         [transitionContext completeTransition:![transitionContext transitionWasCancelled]];
                     }];
    

	}
```
	
		

解释：

>1. 	从上下文中获取toVC(根据需要，这里也可以获取到fromVC).

>2. 	获取toVC的view的快照和初始的frame.

>3. 	获取动画发生的的容器View——containerView,并将toView的快照加入到该容器View中.

>4. 	通过视图动画来执行自定义动画，此代码中的实现了一个iOS7中新增加的弹性动画效果，该动画使视图滑动时，像弹簧一样，稍微拉伸一些，再弹回正确位置.

>5. 	移除toView的快照，并将真正的toView加入到容器View中.

>6. 	向context报告动画切换已经完成.


### 参考资料

1. [WWDC 2013 Session笔记 - iOS7中的ViewController切换](http://onevcat.com/2013/10/vc-transition-in-ios7/);

2. [View Controller 转场](http://objccn.io/issue-5-3/);

3. [iOS7教程系列：自定义导航转场动画以及更多](http://www.cocoachina.com/gamedev/misc/2013/1224/7597.html);

4. [UIViewControllerContextTransitioning](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIViewControllerContextTransitioning_protocol/Reference/Reference.html);

5. [UIViewControllerAnimatedTransitioning](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIViewControllerAnimatedTransitioning_Protocol/Reference/Reference.html);

6. [UIViewControllerTransitioningDelegate](https://developer.apple.com/library/ios/documentation/uikit/reference/UIViewControllerTransitioningDelegate_protocol/Reference/Reference.html);

