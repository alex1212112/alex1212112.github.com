---
layout: post
title: "Swift 与 Objective-C 混编及如何调用 Cocoapods"
date: 2015-04-12 11:42:05 +0800
comments: true
categories: iOS 
---
![](/images/201504121200.png)

###目录

1. Swift 与 Objectve-C 混编
2. Swift 如何使用 Objective-C 的 cocoapods


###Swift 与 Objectve-C 混编

####Swift 文件里使用 Objective-C 类

无论是在 Swift 工程中新建 Objectve-C 的类，还是在 Objective-C 中新建 Swift 的类，Xcode 都会提示 `Would you like to configure an Objective-C bridging header?` ，选择是，就会新建一个Bridging 的文件，文件的名称一般是 `ProjectName-Bridging-Header.h`,比如项目名称是 `Cat`，那么名称就会是`Cat-Bridging-Header.h`，然后在这个文件里引用 Swift 需要用到的 Objective-C 的头文件，比如

```objc
#import "GHCatView.h"
```
那么就在 Swift 的类里使用 Objective-C 类了.


####Objective-C 文件里使用 Swift 类

直接在.m 文件里

```objc
#import "Cat-swift.h"
```

###Swift 如何使用 Objective-C 的 Cocoapods

通过 cocoapods 使用第三方的 Objective-C 的组件的时候，在`ProjectName-Bridging-Header.h`里面引入相关的头文件，比如你要使用 Mantle，就可以

```objc
#import <Mantle/Mantle.h>
```

接下来就可以在 Swift 的文件里使用了
