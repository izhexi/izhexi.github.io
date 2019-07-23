---
title: Translucent与edgesForExtendedLayout和automaticallyAdjustsScrollViewInsets对布局的影响
date: 2016-02-13 21:09:08
tags: iOS
categories: iOS基础
---
`Translucent`与`edgesForExtendedLayout`和`automaticallyAdjustsScrollViewInsets` 对布局的影响

- iOS7 以后 `translucent` 默认为 `true`，`rootView` 从（0，0）开始布局，修改 `edgesForExtendedLayout`` 属性可以改变布局； 
- `translucent` 为 `false`，`rootView` 从导航栏底部开始布局，修改 `edgesForExtendedLayout` 属性无法改变布局，可以通过设置 `extendedLayoutIncludesOpaqueBars` 从（0，0）开始布局； 
- `automaticallyAdjustsScrollViewInsets` 默认值是 `true`，表示在全屏模式下会自动修改第一个添加到 `rootView` 的 `scrollview` 的 `contentInset` 为(64,0,0,0)，用来纠正`scrollview`在全屏模式下的显示； 
- 设置 `UINavigationBar` 的背景图片可以改变导航栏背景色，如果背景图片包含 `alpha` 的色值，系统会默认将 `translucent` 设置为 `true`，没有包含 `alpha` 色值会将 `translucent` 设置为 `false`。但这是针对没有手动设置 `translucent` 值的情况，如果我们手动设置了 `translucent`，那么系统就不会根据背景图片的 `alpha` 来修改 `translucent`