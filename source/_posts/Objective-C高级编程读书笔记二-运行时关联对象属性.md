---
title: Objective-C高级编程读书笔记二-运行时关联对象属性
date: 2016-01-17
tags: iOS
categories: iOS
---

当想使用Category对已存在的类进行扩展时。 

一般只能添加实例方法或类方法，而不适合添加额外的属性。虽然可以在Category头文件中声明property属性，但在实现文件中编译器是无法synthesize任何实例变量和属性访问方法。这时需要自定义属性访问方法并且使用Associated Objects来给已存在的类Category添加自定义的属性。Associated Objects提供三个API来向对象添加、获取和删除关联值。 

在一个类中，使用了多个UIAlertView或者类似的弹窗控件时，使用Block进行事件绑定。 

在有多个相同类弹窗场景下，使用Block与buttonindex绑定的方式，解耦因不同弹窗需要定义进行不同判断，进行相应Action的麻烦。 

使用API 

- void objc_setAssociatedObject (id object, const void *key, id value, objc_AssociationPolicy policy ) 
- id objc_getAssociatedObject (id object, const void *key ) 
- void objc_removeAssociatedObjects (id object ) 

其中objc_AssociationPolicy是个枚举类型，它可以指定Objc内存管理的引用计数机制。 

 
```
typedef OBJC_ENUM(uintptr_t, objc_AssociationPolicy) { 

OBJC_ASSOCIATION_ASSIGN = 0,           */**< Specifies a weak reference to the associated object. \*/* 
OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1, */**< Specifies a strong reference to the associated object.*  
**   The association is not made atomically. \*/* 
OBJC_ASSOCIATION_COPY_NONATOMIC = 3,   */**< Specifies that the associated object is copied.*  
**   The association is not made atomically. \*/* 
OBJC_ASSOCIATION_RETAIN = 01401,       */**< Specifies a strong reference to the associated object.* 
**   The association is made atomically. \*/* 
OBJC_ASSOCIATION_COPY = 01403          */**< Specifies that the associated object is copied.* 
**   The association is made atomically. \*/* 
 };

```
 

Associated Objects的key要求是唯一并且是常量，而SEL是满足这个要求的，所以上面的采用隐藏参数_cmd作为key。 

直接上代码 
```
static char alert_view_complete_block_key;
@implementation UIAlertView (Expand)
- (void)showAlertViewWithCompleteBlock:(AlertViewCompleteBlock)block
{
    if(block)
    {
        objc_setAssociatedObject(self, &alert_view_complete_block_key, block, OBJC_ASSOCIATION_COPY);
        self.delegate = self;
    }
    [self showWithRecord];
}
- (void)alertView:(UIAlertView *)alertView clickedButtonAtIndex:(NSInteger)buttonIndex
{
    AlertViewCompleteBlock block = objc_getAssociatedObject(self, &alert_view_complete_block_key);
    if(block)
    {
        block(buttonIndex, alertView);
    }
}

```
 
 