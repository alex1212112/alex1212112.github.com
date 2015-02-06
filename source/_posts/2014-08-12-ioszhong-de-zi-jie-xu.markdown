---
layout: post
title: "iOS中的字节序"
date: 2014-08-12 12:07:08 +0800
comments: true
categories: 
---

![](/images/b8c60f715c7d7ad3736a3efde8f56eae65d9d45810251-ck6IkC_fw658.png)

####起因

最近在做 iOS 蓝牙开发的时候，将多字节数据从APP端发到蓝牙设备端的时候，发现字节顺序都是反的，比如发送一个 0x0500 的数据给蓝牙设备端，设备端收到以后就变成了0x0005,研究发现这就是字节序不对导致的问题。

####什么是字节序

采用维基百科的解释如下：

>在几乎所有的机器上，多字节对象都被存储为连续的字节序列。例如在C语言中，一个类型为int的变量x地址为0x100，那么其对应地址表达式&x的值为0x100。且x的四个字节将被存储在存储器的0x100, 0x101, 0x102, 0x103位置。

>而存储地址内的排列则有两个通用规则。一个多位的整数将按照其存储地址的最低或最高字节排列。如果最低有效字节在最高有效字节的前面，则称小端序；反之则称大端序。在网络应用中，字节序是一个必须被考虑的因素，因为不同机器类型可能采用不同标准的字节序，所以均按照网络标准转化。

>例如假设上述变量x类型为int，位于地址0x100处，它的十六进制为0x01234567，地址范围为0x100~0x103字节，其内部排列顺序依赖于机器的类型。大端法从首位开始将是：0x100: 01, 0x101: 23,..。而小端法将是：0x100: 67, 0x101: 45,..

![](/images/280px-Big-Endian.svg.png)

![](/images/280px-Little-Endian.svg.png)

####相关体系

* x86，MOS Technology 6502，Z80，VAX，PDP-11等处理器为Little endian。

* Motorola 6800，Motorola 68000，PowerPC 970，System/370，SPARC（除V9外）等处理器为Big endian

* ARM, PowerPC (除PowerPC 970外), DEC Alpha, SPARC V9, MIPS, PA-RISC and IA64的字节序是可配置的。

* 网络传输一般采用大端序，也被称之为网络字节序，或网络序。IP协议中定义大端序为网络字节序。伯克利socket API定义了一组转换函数，用于16和32bit整数在网络序和本机字节序之间的转换。htonl，htons用于本机序转换到网络序；ntohl，ntohs用于网络序转换到本机序。

####iOS中的字节序

```objc
if (NSHostByteOrder() == NS_LittleEndian) {
        NSLog(@"LittleEndian");
    }
    else if(NSHostByteOrder() == NS_BigEndian){
         NSLog(@"BigEndian");
    }
    else {
         NSLog(@"Unknow");
    }
```

通过上述代码打印出来的log，可以知道iOS系统目前采用的是小端序。因此在进行socket网络传输之类的工作时，要记得先把字节序进行转换，然后再传输。iOS自身提供了相应的转换方法，如下：

```objc
UInt16  Byte = 0x1234;
HTONS(Byte);//转换
NSLog(@"Byte == %x",Byte);//打印出来发现顺序变了
```
上述代码中 HTONS(x) 是对2字节进行转换，如果要对4字节进行转换，就要用 HTONL(x)进行转换了，要对更高字节，比如8字节（64位）进行转换，就要自己写转换的方法了。

####参考资料


1. [关于字节序](http://ixhan.com/2009/10/little-about-byte-order/);

2. [Mac&iOS Socket](http://geeklu.com/2012/01/macios-socket/);

3. [ 网络字节序与主机字节序 高低位](http://blog.csdn.net/ernest201210/article/details/8690686);

4. [字节序](http://zh.wikipedia.org/wiki/%E5%AD%97%E8%8A%82%E5%BA%8F);

5. [Byte Ordering](https://developer.apple.com/library/mac/documentation/corefoundation/Conceptual/CFMemoryMgmt/Concepts/ByteOrdering.html);
