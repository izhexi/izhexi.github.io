---
title: AppleWatchKit学习
date: 2015-05-18 20:21:33
tags: [iOS,AppleWatch]
categories: [iOS ,AppleWath]
---


WatchAPP需要与实体APP为载体。即运行在iOS下的APP，iOS下的APP主要用于提供WatchAPP的包配置和资源的初始化。打包时候会打在一起。WatchAPP只是作为宿主APP的一个扩展。  

建议过一遍的资料：  

1.WatchKit Framework Reference  

2.WatchKit Development Tips : Optimize your WatchKit apps with these tips and best practices.  

3.Apple Watch Programming Guide  

4.Developer Forum 

一、工程加入WatchAPP 

1.在当前的target下新建WatchExtension target。  

Select File > New > Target, and navigate to the Apple Watch section. 

2.如果需要glance功能勾选相应的选项 

3.点击next > finish  

添加成功后，可以看到项目里多了两个target。 

二、APP的target Structure  

1、添加WatchKit后，项目中增加了连个可执行也可更新的依赖选项（target），一中红色框的两个东西。两个target都可以独立的build和run，也可以在宿主APP 中运行的时候一起打包到手机，由手机通过蓝牙或者安装到Watch中。WatchExtension 控制着WatchAPP。 

零散知识点：  

1.在WatchKit中，管理scene的类是interfaceController，作用类似与UIViewController。它响应并管理着scene。  

2.如果用户点击了手表的glance，WatchKit会从Watch的storyboard启动glance，如果用户直接启动APP，WatchKit将从初始化的界面开始启动。启动后WatchkitExtension创建interface controller object。 

。 

3.interface viewcontroller方法及作用 

4.如果有长时间的后台线程在跑，或者需要大量的数据交互，最好是依赖于宿主APP，使用WKInterfaceController的openParentApplication:reply: 方法，拿到数据后在传回WatchAPP。 

5.使用NSUserDefaults和APPGroups来进行数据的分享工作。 

6.使用handoff来进行深度的连接glance和notification。可以把目前正在做的工作数据在双方之间传送。 

7.不能动态的添加或者删除UI组件，可以隐藏或者将alpha值设为0，hidden时，其他空间元素会自动填充两者之间的空隙，可能会引起布局的不协调，将alpha设为0较为优选的方法。 

8.使用懒加载方式在willActivate方法里加载数据  

使用 dispach_async 

界面类型 

Ø  常规Watch app，这个是必须的，一些简单内容的显示和简单操作，可以看成手机上的精简版；  

Ø  glance类型的界面，是一种纯提示型的界面，是不能和用户互动的，这部分可有可无；  

Ø  notification类型的界面，这是消息通知时用到的，手机也可以接收推送通过这种界面显示，这种 是可以和用户互动的。 

填充和更新glance内容 

使用init 和 awakeWithContext：方法初始化数据，设置最初的label的值和image的路径。  

使用willActivate来在出现在屏幕前加载最新的数据  

程序会自动使用NSTimer来定时刷新数据，不需要个人自己设置 

注：glance只有一个，不能有多个，不参与交互，紧显示label和image，点击任意都可以启动WatchAPP。 

从glance启动相应的WatchAPP 

 

1. 通常情况下，当点击了glance视图后，会启动相应的WatchAPP。入口为 main interface controller。 

2. 启动内容： 

3. 通常在glance的init 和 willActivate里装配好内容。 

• 使用initWithContext:方法来初始化你的glance界面，并且设置标签和图像的初始值。 

• 基于内容的改变，使用willActivate来更新glance。 

4. 然后在适当的时间点调用updateUserActivity:userInfo:webpageURL: 方法，参数在userinfo里。可以将参数在启动的时间传递给目标interface controller. 

5.  

![](https://leanote.com/api/file/getImage?fileId=568a595cab64415898000d74) 

在main interface controller里实现 handleUserActivity：方法。用传递过来的userInfo（类型为字典）来配置相应的参数。 

调用updateUserActivity:userInfo:webpageURL: 来告诉main interface controller的handUserActivity：方法来激活WatchAPP。 

花田里犯了错 

[WKInterfaceController openParentApplication: reply：与  

application:(UIApplication *)application handleWatchKitExtensionRequest:reply:方法里传递的参数不能是自定义对象，只能是普通类型。 