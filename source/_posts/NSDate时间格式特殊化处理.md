---
title: NSDate时间格式特殊化处理
date: 2016-06-21 21:53:10
tags: iOS
categories: iOS基础
---

iOS特殊格式化时间公式，可以应对90%的场景了。
大多数时候我们采用结合`NSCalendar` 和 `NSDateComponents` 的方式去获取 `年` `月` `周` `日`相关的信息。 
特殊的其他形式我们可以如下处理：
``` objc
NSCalendar *calendar = [[NSCalendar alloc] initWithCalendarIdentifier:NSGregorianCalendar];
    NSDate *now = [NSDate date];
    NSDateComponents *comps = [[NSDateComponents alloc] init];

NSDateFormatter *dateFormatter = [[NSDateFormatter alloc]init]; 

[dateFormatter setDateFormat:@"'公元前/后:'G '年份:'u'='yyyy'='yy '季度:'q'='qqq'='qqqq '月份:'M'='MMM'='MMMM '今天是今年第几周:'w '今天是本月第几周:'W '今天是今天第几天:'D '今天是本月第几天:'d '星期:'c'='ccc'='cccc '上午/下午:'a '小时:'h'='H '分钟:'m '秒:'s '毫秒:'SSS '这一天已过多少毫秒:'A '时区名称:'zzzz'='vvvv '时区编号:'Z "]; 

NSLog(@"%@", [dateFormatter stringFromDate:[NSDate date]]); 

OutPut: 

1. 区域格式：美国 

公元前/后:AD 年份:2013=2013=13 季度:3=Q3=3rd quarter 月份:8=Aug=August 今天是今年第几周:32 今天是本月第几周:2 今天是今天第几天:219 今天是本月第几天:7 星期:4=Wed=Wednesday 上午/下午:AM 小时:1=1 分钟:24 秒:32 毫秒:463 这一天已过多少毫秒:5072463 时区名称:China Standard Time=China Standard Time 时区编号:+0800 

2. 区域格式：中国 

公元前/后:公元 年份:2013=2013=13 季度:3=三季度=第三季度 月份:8=8月=8月 今天是今年第几周:32 今天是本月第几周:2 今天是今天第几天:219 今天是本月第几天:7 星期:4=周三=星期三 上午/下午:上午 小时:1=1 分钟:44 秒:30 毫秒:360 这一天已过多少毫秒:6270360 时区名称:中国标准时间=中国标准时间 时区编号:+0800 

```

