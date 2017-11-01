---
title: iPhoneX适配实战
date: 2017-10-26
tags: [iOS,iOS 11 ,Xcode9,iPhoneX]
category: iOS
---

### 题外话
网上适配的帖子特别多，大多数都是官方教程的中文版。这篇文章主要记录`触手TV`APP在适配中遇到的问题。
这是一次公司内部技术分享会的内容，内容共分为三个部分：
- Xcode9新特性
- iOS 11 适配
- iPhone X适配
 
这是第三部分。这里我不在重复前两部分的内容，用到的知识点会直接使用，比如`safeAreaInsets`。今天的适配默认已经适配`iOS 11`。

第一部分：[Xcode9新特性](59f834805188252c224d4cca)
第二部分：[iOS 11 适配](https://juejin.im/post/59f836ac51882561a209c641)
言归正传

### 简单介绍 iPhone X

#### ScreenSize
iPhone X具有超视网膜屏。逻辑分辨率为`375pt X 812pt`。屏幕尺寸5.8英寸，分辨率为`1125 x 2436`。四个角有圆角、顶部有齐刘海（传感器槽）、底部有`HomeIndicator`。
`StatusBar`导航栏高度`44pt`。`tabBar`高度为`83pt`.
![ScreenSize](https://user-gold-cdn.xitu.io/2017/10/31/5de16370cf29c9bab93f94d023d20153)

#### 界面呈现最佳实践
- 保持页面元素在安全区域内。
- 布局元素不被圆角切割。
- 布局元素不被齐刘海遮挡。
- 布局元素不占用`statusBar`。`statusBar`部分有供系统使用的边缘手势。
- 有交互的控件不被`HomeIndicator`重叠。

如下图

![超过安全区域](https://user-gold-cdn.xitu.io/2017/10/31/cbe599ccf0c91b21e307aa1404013b7b)


![statusBar占用](https://user-gold-cdn.xitu.io/2017/10/31/bd37d683907c1c8693c5934dca00cdca)


![与HomeIndicator重叠](https://user-gold-cdn.xitu.io/2017/10/31/e70a8eb95218c12c9370a6c0880e9d1e)


![statusBar手势](https://user-gold-cdn.xitu.io/2017/10/31/b3b43a763dae0e64aeb412e39dadbda3)


![横屏布局](https://user-gold-cdn.xitu.io/2017/10/31/a2bacc4fbdbc4facfe54967955f70504)

### 开始适配工作

先看看未适配前运行的结果
1. 启动后，没有充满屏幕，上下部分有黑边。
2. 中间的 `+`号按钮下沉。
![首页](https://user-gold-cdn.xitu.io/2017/10/31/9641f99d9ae835306f9e611c2b8c5b82)

3. “我的”页面内容下沉。
    这个问题属于iOS 11适配问题，上一篇我们讲过，这里不赘述。
![我的页面](https://user-gold-cdn.xitu.io/2017/10/31/60965dfe528444d3823d58af3387806a)

先把第一个问题解决了。因为iPhone X的分辨率问题，APP在启动的时候，没有相应的图片资源。解决这个问题只需对`LaunchImage`新增`1125 x 2346`启动屏图片。或者启动项从`Launch Images Source`->asset变更为`LaunchScreenFile`。指定为`LaunchScreen`的`xib`或者`.storyboard`。

运行再看效果：

![首页重叠](https://user-gold-cdn.xitu.io/2017/10/31/23c892dd81232a0109435ad7cabb8660)
发现问题：
1. 顶部已经与`statusBar`重叠了。
    尝试点击，按钮上半部分不响应事件。`statusBar`的View在顶部，事件被`statusBar`截获了。这种方式也不符合UI规范。
2. 其他页面基本都是这样的问题。
3. 中间按钮在push到其他页面回来后，偶尔会往下掉。

在上一篇我们讲了一般使用`IUNavigationBar`的三种方式。触手使用的是第三种--完全自定义的view。因为`触手TV`的界面风格里，很多页面都是首页这个样式的，顶部是tab样式的，页面是联动的。所以我们有专门的一个控件来封装这个风格的页面。所以很久以前的工程里为了方便，直接弃用了系统的`NavigationBar`。
做出修改之前，我们先来看看整个工程的view层级关系：

![原来的APP结构](https://user-gold-cdn.xitu.io/2017/11/1/e8cb398c2623b4b69860f5ec248d6fc4)
从结构中可以看出，整个APP里共用同一个NavigationController，这种做饭，有缺点也有优点，优点就是针对未适配之前的页面，隐藏导航栏，在源头就可以直接设置。在需要push的地方，除极个别被modal出来的Controller，直接可用`self.navigationController`获得导航属性。缺点就是不能针对定制页面控制导航栏的显示与影藏。可能你会想在`viewWillAppear`和`ViewWillDisappear`里去控制隐藏和显示。考虑到多个视图控制器的执行时序先后对导航栏进行设置，结果就不一定是我们想要的了。
为了更好的使用系统导航栏，又能离散的控制导航栏的隐藏，我们将APP的结构改为这样。

![修改后的APP结构](https://user-gold-cdn.xitu.io/2017/11/1/1b60fa9efe38afa7191ff969e1d4eb63)
显而易见这样的结构更合理一些。我个人比较推荐使用底部`tab`形式的UI，都使用这样的结构。

准备工作做完了，开始适配导航栏。
在原来的`MainViewController：UITabBarcontroller`里有这么一段代码
```objc
- (void)viewWillAppear:(BOOL)animated {
    [super viewWillAppear:animated];
    [self.navigationController setNavigationBarHidden:YES animated:YES];
}

```
这段代码就是在源头把整个APP里的导航栏隐藏的关键代码。现在可以直接删掉。
在基类的`ViewControoler`里，新增`bool`属性`hiddenNavigationBar`。在需要隐藏导航栏的页面，设置为`true`。
在`NavigationController`的基类里实现`UINavigationControllerDelegate`方法,根据控制器的`hiddenNavigationBar`属性，是否隐藏导航栏。不建议在`viewWillAppear`和`viewWillDisappear`设置，这种方式在视觉上的表现不是太好，也不怎么优雅。
```objc
- (void)navigationController:(UINavigationController*)navigationController willShowViewController:(UIViewController*)viewController animated:(BOOL)animated {
    
    if([viewController isKindOfClass:[CSViewController class]]){
        CSViewController *vc = (CSViewController *)viewController;
        if (vc.navigationBarHidden) {
            [navigationController setNavigationBarHidden:YES animated:YES];
        } else {
            if (navigationController.navigationBarHidden) {
                [navigationController setNavigationBarHidden:NO animated:YES];
            }
        }
    }
}
```
我们决定使用系统的导航栏，主要原因是为了减少工作量。在原来的页面里，每个xib页面维护了一个自定义的导航栏，这次改动，每个xib文件都需要改一遍，为了一劳永逸的解决这个问题。我们觉得直接向原生控件靠拢，把适配的工作交给系统来完成。其实工作量也没多少，把每个页面的导航栏删掉就可以了。运行后效果如下：

![导航示例](https://user-gold-cdn.xitu.io/2017/11/1/47cedc569537bb72a792c167fad37981)

针对于上面提出的中间按钮下沉问题。我们先看看iPhone X上tabBar的View层级关系。
![tabBar](https://user-gold-cdn.xitu.io/2017/11/1/9f7719c9d96c8a916ada251144994d70)
高度是83，下面预留了35给虚拟home键。所以原先的直接添加到`UITabBarController`，设置frame就有了偏差,为了布局，使用了一个空的`TabBarItem`作为占位用。
```objc
buttonCenter = [[UIButton alloc] initWithFrame:CGRectMake(kScreenWidth / 2 - 30, kScreenHeight - 60 - offSety, 60, 60)];
    [buttonCenter setImage:[UIImage imageNamed:@"ic_main_center.png"] forState:UIControlStateNormal];
    [buttonCenter setImage:[UIImage imageNamed:@"ic_main_center_p.png"] forState:UIControlStateHighlighted];
    [buttonCenter addTarget:self action:@selector(clickedButtonCenter) forControlEvents:UIControlEventTouchUpInside];
    [self.view addSubview:buttonCenter];
```
解决这个问题，可以在原来的基础上针对iPhone X适配frame。我们没有这么做，而是选择继承`UITabbar`，重写`layoutSubviews`。顺便把很多运营需要的逻辑红点也剥离到这里。核心代码如下
```objc
- (void)layoutSubviews {
    [super layoutSubviews];
    Class class = NSClassFromString(@"UITabBarButton");
    CGFloat width = self.width / BarItemCount;
    for (UIView *view in self.subviews) {
    
    //**
    plusButton 就是需要中间的按钮
    */
    
        if ([view isEqual:self.plusButton]) {
            view.frame = CGRectMake(0, 0, width, width);
            view.center = CGPointMake(self.width / 2,BarItemCenterY);
        } else if ([view isKindOfClass:class]){
            //如果是系统的UITabBarButton，那么就调整子控件位置，空出中间位置
            CGRect frame = view.frame;
            NSInteger index = view.origin.x / width;
            NSInteger originIndex = index;
            if (index >= (BarItemCount - 1) / 2) {
                index++;
            }
            CGFloat x = index * width;
            view.frame = CGRectMake(x, view.frame.origin.y, width, frame.size.height);
            }
        }
    }
    [self bringSubviewToFront:self.plusButton];
}
```
在事件响应链里截获点击事件，对超出tabBar的部分支持事件的响应
```objc
-(UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event{
    //这一个判断是关键，不判断的话push到其他页面，点击发布按钮的位置也是会有反应的，这样就不好了
    //self.isHidden == NO 说明当前页面是有tabbar的，那么肯定是在导航控制器的根控制器页面
    //在导航控制器根控制器页面，那么我们就需要判断手指点击的位置是否在发布按钮身上
    //是的话让发布按钮自己处理点击事件，不是的话让系统去处理点击事件就可以了
    if (self.isHidden == NO) {
        //将当前tabbar的触摸点转换坐标系，转换到发布按钮的身上，生成一个新的点
        CGPoint newP = [self convertPoint:point toView:self.plusButton];
        CGPoint extendPoint = [self convertPoint:point toView:self.extendView];
        //判断如果这个新的点是在发布按钮身上，那么处理点击事件最合适的view就是发布按钮
        if ( [self.plusButton pointInside:newP withEvent:event]) {
            return self.plusButton;
        } else {//如果点不在发布按钮身上，直接让系统处理就可以了
            return [super hitTest:point withEvent:event];
        }
    }
    else {//tabbar隐藏了，那么说明已经push到其他的页面了，这个时候还是让系统去判断最合适的view处理就好了
        return [super hitTest:point withEvent:event];
    }
}
```

目前为止，我们已经解决了上面提出来的问题。
在看看其他页面，发现有几个问题。
动态详情评论框与home键区域重叠了。
![动态详情评论框](https://user-gold-cdn.xitu.io/2017/11/1/a36657d3305a385e80785b155024191d)
直播页面的弹幕框也是这个问题。
![](https://user-gold-cdn.xitu.io/2017/11/1/ba2e361d31995977ba40a47f18dc818b)
直播页面横屏下按钮排布问题。
![直播横屏](https://user-gold-cdn.xitu.io/2017/11/1/8ff38aaeb6e7b7816a96a3d7433c83e6)
这些只需要基本都是UED的范畴了，对特定的约束进行调整，使其不遮挡home键。
有了第一部分：[Xcode9新特性](59f834805188252c224d4cca)的介绍和第二部分：[iOS 11 适配](https://juejin.im/post/59f836ac51882561a209c641)。iPhone X的适配就简单多了。

### 使用`safeAreaInsets`

通篇我们都没有用到`safeAreaInsets`来进行适配，是应为在IB界面里，`safeAreaInsets`支持的最低版本为iOS 9.0，我们还需要兼容之前的版本。所以这里没有介绍这种方法。
如果需要使用`safeAreaInsets`，那么在IB文件打开右侧第一个菜单项`showfileinspector`，勾选`User Safe Area Layout Guides`
![user safe area](https://user-gold-cdn.xitu.io/2017/11/1/44075a86f3c1cd143a3395057b52604e)
在设置约束时，请选择`safe area`作为`secondItem`。这样就可以很好的适配所有机型，保证不同机型下，都能做到官方推荐的布局规范了。
![safe area layout](https://user-gold-cdn.xitu.io/2017/11/1/9c1fad17d5fd11e7c6c520484ffc8442)