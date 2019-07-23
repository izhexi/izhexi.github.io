---
title: Objective-C高级编程读书笔记
date: 2016-01-13
tags: iOS
categories: iOS
---

内存管理（非ARC） 

一 对象的操作类型以及发出对应的动作。 



| **操作**       | **动作**                     |
| -------------- | ---------------------------- |
| 生产并持有对象 | alloc/new/copy/mutableCopy等 |
| 持有对象       | retain 方法                  |
| 释放对象       | release方法                  |
| 废弃对象       | dealloc方法                  |

二 在引用计数的实现上 

`GNUstep将引用计数数据保存在对象占用内存块头部的变量中。`

![](/img/count.png)

通过内存块头部管理引用计数： 



- 用少量代码即可完成 
- 能够统一管理引用计数内存块与对象用内存块 

`Apple苹果采用散列表来管理引用计数。`
![](/img/map_count.png)

- 对象用内存块的分配无需考虑内存块的头部。 
- 引用计数表各记录中存有内存块地址，可以从各个记录追溯到各对象的内存块。这一特性在调试时有着举足轻重的作用，即使出现故障导致对象占用的内存块损坏，只要计数表没有被破坏，就能够确认各内存块的位置。对检测内存泄露很有帮助。 

三 AutoRelease 

用与自动释放对象，类似于C中的局部变量。当超出了作用域后，对象实例的release实例方法被调用。  

使用方法:  

（1）生产`NSAutoreleasePool`对象  

（2）调用已分配对象的`autorelease`实例方法  

（3）废弃`NSAutoreleasePool`对象。  

![](/img/life.png)

示例代码 


```
NSAutoreleasePool *pool = [[NSAutoreleasePool alloc]init]; 
id obj = [[NSObject alloc]init]; 
[obj autorelease]; 
[pool drain]; 
```


虽然有了自动释放池，在大量产生`autorelease`对象时，只要不废弃`NSAutoreleasePool`对象，那么生成的对象就不能被释放，所以，有空产生内存不足的现象。比如通过`NSData`加载很多的图片。在此情况下，有必要在适当的地方生产、持有、或废弃`NSautoreleasePool`对象。  

自动释放 `NSAutoreleasePool`对象将会发生异常。因为在OBJC（Foundation框架）中无论调用哪一个对象的autorelease实例方法，实际都是调用的是`NSObject`类的`autorelease`实例方法。但是`NSAutoreleasePool`类，`autorelease`实例方法已经被该类重载(我理解的是重载没有进行实现或者改变了原来的实现，在重载中没有调用super实现)，因此会抛异常。 