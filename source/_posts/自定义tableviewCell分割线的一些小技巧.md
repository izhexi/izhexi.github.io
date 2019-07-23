---
title: 自定义tableviewCell分割线的一些小技巧
date: 2016-03-05 
tags: iOS
categories: iOS基础
---

如果只是设置分割线，那么系统提供的API就可以实现了。
- 设置cell的分割线颜色 
    `[tableView setSeparatorColor:color]; `

- 设置cell分割线的长短 
    `[tableViewsetSeparatorInset:UIEdgeInsetsMake(top,left,bottom,right)]; `

- 需要重绘Cell的分割线 
```
- (void)drawRect:(CGRect)rect 

{ 

        CGContextRef context = UIGraphicsGetCurrentContext(); 

        CGContextSetFillColorWithColor(context, [UIColor clearColor].CGColor); 

        CGContextFillRect(context, rect); 

        //上分割线， 

        CGContextSetStrokeColorWithColor(context, [UIColor colorWithHexString:@"ffffff"].CGColor); 

        CGContextStrokeRect(context, CGRectMake(5, -1, rect.size.width - 10, 1)); 

        //下分割线 

        CGContextSetStrokeColorWithColor(context, [UIColor colorWithHexString:@"e2e2e2"].CGColor); 

        CGContextStrokeRect(context, CGRectMake(5, rect.size.height, rect.size.width - 10, 1)); 

} 
```
- 设置cell的描边 （其实就是分组） 
除非需要设置每个section的header，不需要用代理去返回header的view，使用xib或SB布局时，设置style为Group就OK了。
