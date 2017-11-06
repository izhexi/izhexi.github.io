---
title: 适配iOS11
date: 2017-10-25
tags: [iOS,iOS 11 ,Xcode9]
category: iOS 
---
### 题外话
这是一次公司内部技术分享会的内容，内容共分为三个部分：
 - Xcode9新特性
 - iOS 11 适配
 - iPhone X适配
 
 这是第二部分，如有需要请持续关注。
第一部分[Xcode9新特性](/category/XCode9新特性)

言归正传

### 掀起江湖恩怨
iOS 11正式版已经来了，作为一个iOS开发者，这意味着没有适配iOS 11都晚了。好在还在Beta阶段我司技术大牛达叔第一时间体验了一把，并仔细的跑了一遍播放端APP`触手TV`和录制端APP`触手录`，除了有一个由第三方库`WebViewJavascriptBridgeBase`引起的严重crash，两个APP在iOS 11下基本没什么问题，发现的问题已经被达叔提前fixed了。所以组里一直没有进行适配工作，而是把精力放在了最近的大版本开发上。观察发现，直接从AppStore下载的应用，在iOS 11上跑起来是没有什么问题的，如果使用Xcode 9  Building后在运行，就或多或少的出现问题。因为Xcode 9 的Base SDKS是基于iOS 11的。所以还是需要进行适配的。

#### 风云再起
通过阅览网上适配iOS 11的同行案列，结合`触手TV`APP实际问题，经过汇总，以下可能是需要适配的点。找到问题的根源，才能帮助我们解决问题。
为什么会出现上述问题呢？我们看看iOS 11有些什么新增改动。
这里只列出部分以及跟今天主题相关的部分。详情可以看看官网[what's new in iOS 11](https://developer.apple.com/library/content/releasenotes/General/WhatsNewIniOS/Articles/iOS_11_0.html)。

##### UIViewController 与 UIView

###### 主要变化部分：
    
- `UIViewController`废弃`LayoutGuide`。iOS7之后，为了辅助Autolayout布局系统，Apple新增`UILayoutSupport`协议。就是被很多人忽略的`topLayoutGuide`，`bottomLayoutGuide`。

```objc
@interface UIViewController (UILayoutSupport)
// These objects may be used as layout items in the NSLayoutConstraint API
@property(nonatomic,readonly,strong) id<UILayoutSupport> topLayoutGuide API_DEPRECATED_WITH_REPLACEMENT("-[UIView safeAreaLayoutGuide]", ios(7.0,11.0), tvos(7.0,11.0));
@property(nonatomic,readonly,strong) id<UILayoutSupport> bottomLayoutGuide API_DEPRECATED_WITH_REPLACEMENT("-[UIView safeAreaLayoutGuide]", ios(7.0,11.0), tvos(7.0,11.0));
@property(nonatomic) UIEdgeInsets additionalSafeAreaInsets API_AVAILABLE(ios(11.0), tvos(11.0));
```

这个两个属性是`readonly`的，一般我们也用不到，主要作用是辅助Controller的View在布局时知道从哪里开始布局，到什么地方结束布局。在使用SB或者XIB文件布局的时候，可以进行设置，确定开始布局和结束布局的参考点。在iOS 11 已经标记`API_DEPRECATED_WITH_REPLACEMENT`。取而代之的是`UIView`新的API`safeAreaInsets`。
![LayoutGuide](https://user-gold-cdn.xitu.io/2017/10/31/f8546c3c053d641cb83d758f308716aa "LayoutGuide")
- `UIView`增加`safeAreaInsets:UIEdgeInsets`安全区概念。用于替代辅助自动布局的`LayoutGuide`。安全区域定义了布局View除各种Bar后剩下的可见区域。此属性是`readonly`的，想要改动，需要操作`UIViewController`的`additionalSafeAreaInsets`属性。

    <font color = red>简单理解就是去除系统的各种Bar以及左右`margin（如果有设置过)` 后的可用布局区域。</font>

    图中淡蓝色区域即为安全区域:

![test](./img/layoutGuide.png)

    如果在View上的控件没有被遮挡，`safeAreaInsets = {(0,0),(0,0)}`分别对应于`（top,left,bottom,right）`。
    假设子ViewA顶部被`navigationBar`遮挡20pt，相应`safeAreaInsets = {(20,0),(0,0)}`。

    <font size = 5>注意：如果采用SB、XIB布局UI的，`safeAreaInsets`最低支持版本为iOS 9.0。考虑到兼容性，触手TV是不能使用安全区域的。</font>

- `UIViewController`废弃`automaticallyAdjustsScrollViewInsets`API。这是个bool值属性，描述了是否自动适配scrollView的contentInsets。值为`true`时，scrollView的内容不会被系统可见的各种bar所遮挡（`statusbar`、`navigationBar`，`tabBar`，`toolBar`），一般会引起`tableView`，`collectionView`，`scrollView`的content下移bar的高度值个像素。如果不需要自动调整，将值设为`false`。

- `UIViewController`新增修改关联`View`安全区域的API`additionalSafeAreaInsets`。可对安全区域的值进行增减，满足自定义UI的需求。并提供方法`viewSafeAreaInsetsDidChange`，用于在安全区域改变后，进行布局的调整。

- 相应的，`UIScrollView`增加枚举属性`contentInsetAdjustmentBehavior`，描述`scrollView`如何调整contentInset（实际是调整`adjustedContentInset`属性），配合安全区域使用。最后`UIscrollView`的contentInset值为`adjustedContentInset`与`safeAreaInsets`之和。

    有4个可选值：
    ```objc
    typedef NS_ENUM(NSInteger, UIScrollViewContentInsetAdjustmentBehavior) {
            UIScrollViewContentInsetAdjustmentAutomatic, //与 automaticallyAdjustsScrollViewInsets 类似
            UIScrollViewContentInsetAdjustmentScrollableAxes, // 在滚动的当前轴向上进行自动调整
            UIScrollViewContentInsetAdjustmentNever,  //不进行任何的调整
            UIScrollViewContentInsetAdjustmentAlways, // contentInset等于View的safeAreaInsets
        } API_AVAILABLE(ios(11.0),tvos(11.0));
    ```
    `UIScrollViewDelegate`新增`scrollViewDidChangeAdjustedContentInset`方法，当`adjustedContentInset`改变后通知使用者进行布局调整。


总结：
 - 如果使用SB、XIB布局时启用了安全区域，IB会参考安全区域进行布局。
 - 布局时没有以安全区域作为参照时，直接设置安全区域，系统并不会对布局做出自适应动作，可以通过安全区域改变回调方法进行调整。
 - 如果是`UIscrollView`或子类，如果设置了`UIScrollViewContentInsetAdjustmentBehavior`不等于`UIScrollViewContentInsetAdjustmentNever`，系统会自动适配`UIScrollView`的内容都在安全区域内，保证内容不被各种Bar遮住。

##### NavigationBar的改变
`NavigationBar` 多了个`contentView`，这个View在开启了大标题时，上面会有个titleLable。
`NavigationBar` 的`titleView`支持自动布局。需要使用者自动撑开添加到上面的View。

##### 定位权限
在iOS11，原有的`NSLocationAlwaysUsageDeion`被降级为`NSLocationWhenInUseUsageDeion`。
在`.plist`没文件中配置`NSLocationAlwaysAndWhenInUseUsageDeion`，系统框才会弹出，使用requestAlwaysAuthorization获取权限。

##### 其他变更
0. 设置UIBarItem.landscapeImagePhone 与UIBarItem.largeContentSizeImage 来适配在竖屏和横屏下的BarItem图标和双击放大后的图标，如果使用 PDF 资源图，系统会自动从PDF资源图提取相应图标，就不用设置上述属性了。
1. navigationBar.prefersLargeTitles = true 新增大标题属性，大标题displaymode控制显示枚举：navigationItem.largeTitleDisplayMode 
    枚举属性：

        - automatic：自动保存上一次设置的值
        - always：总是显示大标题
        - never：不显示

2. Navigation集成searchBar。navigationItem.searchController 属性赋值可以集成searchBar在navigationbar下方，navigationItem.hidesSearchBarWhenScrolling属性控制在滚动时是否自动隐藏。

3. 确保避免 size 为 0 的自定义view，实现intrinsicContentSize 方法提供默认尺寸。

4. tableview开启开启高度估算（Self-Sizing），设置下面三个属性，使高度估算失效：
```objc
        tableView.estimatedRowHeight = 0 

        tableView.estimatedSectionHeaderHeight = 0 
        
        tableView.estimatedSectionFooterHeight = 0 
```
5. tableview 新增左滑右滑交互。

6. 废弃iOS7 以后的layoutMargins，取而代之的是新增的安全区域的概念。
    新增directionalLayoutMargins。对应layoutMargins。 
    新增systemMinimumLayoutMargins，当directionalLayout小于systemMinimumLayoutMargins，使用systemMinimumLayoutMargins。
    新增 UIViewController=.viewRespectsSystemMinimumLayoutMargins,默认为FALSE。设为TRUE，可设置任意值。

### 新仇旧恨
明白了iOS 11大法，再来看看，这大法带来的各种问题。

#### NavigationBar问题。
一般使用`NavigationBar`基本有三种手法。
1. 纯正血统，使用标注控件，不在`NavigationBar`上添加任何的控件。
2. 混血，在`NavigationBar`上有定制的控件。
3. 毫无血统，完全自定义。隐藏了系统的`navigationBar`，直接用了View替代。

针对上面三种情况：

* 使用第一种姿势的人，很幸运，你不需要进行适配。（估计很少有人不定制）

* 使用第二种姿势的人，可能会有返回按钮、titleView、添加的控件`position`不正确的问题。

* 使用第三种姿势的人，很好，在非iPhone X上，也没什么大问题。

#### UIscrollView、UItableView、UICollectionView内容下沉问题。
升级iOS 11后，发现有些使用`UItableView`布局的页面，顶部多出来了20pt或者44pt，也或者64pt。也就是顶部可见Bar的高度的总和。
`包括使用MJRefresh引起的问题。也属于这类。`

#### Xcode9 打出的包（iOS 11 SDK）页面卡顿问题
升级Xcode9 后，你会发现，基于iOS 11打出来的包，tableView滑动的时候，一卡一顿的。

#### 请求定位框不弹出
在iOS 11有些应用在请求定位时，未能弹出系统请求权限的对话框。

#### 返回按钮位置偏移问题
iOS 11的的leftBarButtonItem 或者右边都距离边距20像素。

#### 其他问题
1. 使用`YYKit`的大图预览控件`YYPhotoGroupView`，dismiss的时候，有些页面会抖一下。
 
### 一笑泯恩仇
既然已经知道了iOS 11初出江湖的各种套路和带来的血雨腥风。那就春风化雨，见招拆招了。
#### navigationBar 问题。
* 因为`NavigationBar`引入了`AutoLayout`，以往的`frame`方式可能位置有偏差。以前使用`CGRectMakeZero`自动撑大已经行不通。那么使用自动布局。或者实现下面`View`的方法，提供默认尺寸。

```objc
- (CGSize)intrinsicContentSize {
    return CGSizeMake(100,100);
}
```
* 添加到`NavigationBar`View可能会出各种问题，尝试添加到`contentView`上。并设置约束。
* 返回按钮问题，请设置好`frame`在赋值给`navigationItem.leftBarButtonItem`。

#### UIscrollView、UItableView、UICollectionView内容下沉问题
因为`controllerView`废弃了`automaticallyAdjustsScrollViewInsets`，请使用`UIScrollViewContentInsetAdjustmentNever`来告诉系统，不要调整。或者设置`additionalSafeAreaInsets`来增加`safeAreaInsets`来抵消。

#### 滚动卡顿问题
因为`tableView`默认开启`Self-Sizing`。设置下面三个属性，使高度估算失效：
```
    tableView.estimatedRowHeight = 0 
    tableView.estimatedSectionHeaderHeight = 0 
    tableView.estimatedSectionFooterHeight = 0 
```

#### 请求定位框不弹出
在iOSi11，原有的NSLocationAlwaysUsageDeion被降级为NSLocationWhenInUseUsageDeion。因此，在原来项目中使用requestAlwaysAuthorization获取定位权限，而未在plist文件中配置NSLocationAlwaysAndWhenInUseUsageDeion，系统框不会弹出。建议新旧key值都在plist里配置。

#### 页面跳动问题
参照内容下沉问题解决。

#### 按钮偏移问题
iOS 11后，navigationBar新增了`contentView`来承载开发者添加的`barButton`。左右两边新增了20像素。现在很几种解决方案。如果只是想要调节返回按钮，可以直接使用系统的API:
```objc
@property(nullable,nonatomic,strong) UIImage *backIndicatorImage;
@property(nullable,nonatomic,strong) UIImage *backIndicatorTransitionMaskImage;
```
但是这样的处理方式，在有多个按钮的使用情景下就引起问题。20个像素还是存在的。
还有调整`UIButton`的`imageEdgeInsets`的，其实也会在多按钮的时候出现布局问题。
[比如这篇帖子](http://m.blog.csdn.net/zhaotao0617/article/details/78063485)。
也有的在`push`和`pop`的时候进行设置的。修改约束会引起有些约束丢失。[也有重写drawRect的](http://www.jianshu.com/p/383cdad95a32)。

其实不需要这么麻烦。新建一个类，继承自`UINavigationBar`，然后重写`layoutSubviews`，如果是ios11，设置`contentView`的`layoutMargins`为需要的值，之前的版本就执行`super`。
核心代码如下：
```objc
@interface CustomNavigationBar:UINavigationBar
@end


const CGFloat LeftFiexSpace = 0;
const CGFloat RightFiexSpace = 8.0;

@implementation CustomNavigationBar
- (void)layoutSubviews {
    [super layoutSubviews];
    // 修正 ios 11 左右两边的边距
    if (SYSTEM_VERSION_GREATER_THAN_OR_EQUAL_TO(@"11.0")) {
        self.layoutMargins = UIEdgeInsetsZero;
        for (UIView *subview in self.subviews) {
            if ([NSStringFromClass(subview.class) containsString:@"ContentView"]) {
                subview.layoutMargins = UIEdgeInsetsMake(0, LeftFiexSpace, 0, RightFiexSpace);
                [self layoutIfNeeded];
            }
        }
    }
}
@end
```

在生成`NavigationController`的地方使用`KVC`将原来的`NavigationBar`替换成自己的.
```objc
 UINavigationController *nvc = [super initWithRootViewController:rootViewController];    
    CSNavigationBar *naviBar = [[CustomNavigationBar alloc] init];
    [nvc setValue:naviBar forKey:@"navigationBar"];
```


### 最后

如果有写得不对的欢迎指正，有更高好的解决方法，也欢迎交流。

### 参考文章

[苹果官网适配视频教程](https://developer.apple.com/videos/play/wwdc2017/204/)

[如何设置返回按钮](http://www.jianshu.com/p/0103cd689cfa)

[你可能需要为你的APP适配iOS11](http://wetest.qq.com/lab/view/326.html)此篇基本上是官方视频的文字版



