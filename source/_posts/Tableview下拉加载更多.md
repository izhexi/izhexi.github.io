---
title: Tableview下拉加载更多
date: 2016-03-04 
tags: iOS
categories: iOS基础
---
新建`Class`继承 `UITableView`.

`.h` 文件
```

@interface TestTableView : UITableView 

@end

```
`.m` 文件 

 ```

@implementation TestTableView

- (void)setContentSize:(CGSize)contentSize

{
    
    if (!CGSizeEqualToSize(self.contentSize, CGSizeZero))
        
    {
        
        if (contentSize.height > self.contentSize.height)
            
        {
            
            CGPoint offset = self.contentOffset;
            
            offset.y += (contentSize.height - self.contentSize.height);
            
            self.contentOffset = offset;
            
        }
        
    }
    
    [super setContentSize:contentSize];
    
}

@end 

```