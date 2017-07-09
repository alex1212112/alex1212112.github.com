---
layout: post
title: "CIKernel 的学习"
date: 2015-03-23 21:39:36 +0800
comments: true
categories: iOS 
---
![](/images/201503232142.png)

在使用 Core Image 的时候，Apple 提供的效果有时候无法满足我们的需求，我们希望能实现自己的滤镜，这个时候我们可以用 CIKernel。CIKernel 是 iOS8 的时候出现的，它是一种类似于 OpenGL 着色器的处理程序。苹果提供了一系列的函数和数据类型供我们使用它。具体参见[Core Image Kernel Language](https://developer.apple.com/library/mac/documentation/GraphicsImaging/Reference/CIKernelLangRef/ci_gslang_ext.html)。

###子类

CIKernel 有两个子类：`CIColorKernel`和`CIWarpKernel`。

###初始化

我们通过一个 `NSString` 来初始化一个 `CIKernel`；

```objc
   NSString *kernelString =
    @"kernel vec4 moveUpTwoPixels (sampler image) {\n"
    @"  vec2 dc = destCoord();\n"
    @"  vec2 offset = vec2(0.0, 2.0);\n"
    @"  return sample (image, samplerTransform (image, dc + offset));\n"
    @"}";

    CIKernel *kernel = [CIKernel kernelWithString:kernelString];
```

###使用

使用的时候可以这么用：

```objc
UIImage *image = [UIImage imageNamed:@"flower"];
CIImage *inputImage = [CIImage imageWithCGImage:image.CGImage];

CIImage *outImage = [kernel applyWithExtent:inputImage.extent roiCallback:^CGRect(int index, CGRect rect) {
        return inputImage.extent;

    } arguments:@[inputImage]];
```

我们可以看到，核心代码为：

```objc
kernel vec4 moveUpTwoPixels (sampler image) {
    vec2 dc = destCoord();
    vec2 offset = vec2(0.0, 2.0);
    return sample (image, samplerTransform (image, dc + offset));
}
```

这是一个 函数，其中返回值 vec4 是一个 由 float 构成的向量，苹果在文档里说这个函数的返回值必须是这种类型。函数的名称 叫 `moveUpTwoPixels`,函数的参数为一个 sampler 类型，对于我们的程序来说，这个参数要求我们传递一个CIImage，这里的参数也可以有多个，对应着

```objc
- (CIImage *)applyWithExtent:(CGRect)extent
                   arguments:(NSArray*)args
```
 方法里的 args 的参数。


###CIColorKernel

我们也可以创建一个CIColorKernel:

```objc
NSString *kernelString =
    @"kernel vec4 chromaKey( __sample s, __color c, float threshold ) { \n"
    @"  vec4 diff = s.rgba - c;\n"
    @"  float distance = length( diff );\n"
    @"  float alpha = compare( distance - threshold, 0.0, 0.5 );\n"
    @"  return vec4( s.rgb, alpha ); \n"
    @"}";

CIColorKernel *kernel = [CIColorKernel kernelWithString:kernelString];
```


使用的时候:

```objc
UIImage *image = [UIImage imageNamed:@"flower"];
CIImage *inputImage = [CIImage imageWithCGImage:image.CGImage];
CIImage *outImage = [kernel applyWithExtent:inputImage.extent arguments:@[inputImage,[CIColor colorWithRed:0.0f green:1.0f blue:0.0f],@0.0f]];
```

###更多

 进一步学习中。。。



###参考资料

 1. [iOS8 Day-by-Day :: Day 19 :: CoreImage Kernels](http://www.shinobicontrols.com/blog/posts/2014/08/19/ios8-day-by-day-day-19-coreimage-kernels)

 2. [Core Image Kernel Language](https://developer.apple.com/library/mac/documentation/GraphicsImaging/Reference/CIKernelLangRef/ci_gslang_ext.html#//apple_ref/doc/uid/TP40004397-CH206-TPXREF101)

 3. [About Core Image](https://developer.apple.com/library/mac/documentation/GraphicsImaging/Conceptual/CoreImaging/ci_intro/ci_intro.html#//apple_ref/doc/uid/TP30001185)

 4. [What You Need to Know Before Writing a Custom Filter](https://developer.apple.com/library/mac/documentation/GraphicsImaging/Conceptual/CoreImaging/ci_advanced_concepts/ci.advanced_concepts.html#//apple_ref/doc/uid/TP30001185-CH9-SW1)

 5. [Creating Custom Filters](https://developer.apple.com/library/mac/documentation/GraphicsImaging/Conceptual/CoreImaging/ci_custom_filters/ci_custom_filters.html#//apple_ref/doc/uid/TP30001185-CH6-TPXREF101)

 6. [Packaging and Loading Image Units](https://developer.apple.com/library/mac/documentation/GraphicsImaging/Conceptual/CoreImaging/ci_image_units/ci_image_units.html#//apple_ref/doc/uid/TP30001185-CH7-SW12)

 7. [如何制作Core Image滤镜插件](http://www.cocoachina.com/b/?p=174#more-174)
