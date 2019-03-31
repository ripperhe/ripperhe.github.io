---
layout: post
title:  UIWindow 第二弹
categories: iOS
description: UIWindow 进一步分析
keywords: iOS, UIWindow
---

## 前言

近期被一个 UIWindow 的问题坑惨了 🤣，网上查了很久，没什么资料，所以仔细再次深入研究了一下。

本文以问题的形式阐述，以下结论全部是看官方文档以及自己试验得出，如有错误，还望指出。

##  UIApplication ★ keyWindow

> The app's key window.
> 
> This property holds the UIWindow object in the windows array that is most recently sent the makeKeyAndVisible message.

### keyWindow 的作用？

接收键盘事件和其他非触摸性事件。同一时间只有一个 keyWindow。

### keyWindow 指的是哪一个 window？

最后一个调用 `makeyAndVisible` 或 `makeyWindow` 的 window

keyWindow 的 `isHidden` 属性可能为 `YES`。如果一个 window 初始化之后，没有调用 `makeyAndVisible`，直接调用 `makeyWindow`，就会出现这种情况

### 当前 keyWindow 被设置 hidden = YES，系统默认的下一个 keyWindow 是哪一个？

我们销毁一个 window 的时候，往往这样写

```objc
self.window.hidden = YES;
self.window = nil;
```

如果这个 window 恰好是当前 keyWindow，调用 `hidden = YES` 的时候，系统将默认取消其 keyWindow，并在 `hidden = YES` 方法内部设置一个新的 keyWindow

至于系统怎么设置新的 keyWindow

这是一个问题 ⁉️ 🤣

经过多番测试，找到如下规律：

* 一定是 windows 数组中，用户创建的 level 最大的，但是 level 最大的可能有几个，因为 level 可以相同
* level 相同的时候，一定是实际显示在最上面的那个

总结一下就是，如果当前的 keyWindow  设置 `hidden = YES`，系统会去找 windows 数组中 **实际显示在最上层的那个 window** 
（当前的 keyWindow 可能就是实际显示在最上层的那个，系统会将其跳过）

需要注意的是，实际显示在最上面的不一定是 windows 最后一个元素，后面会讲~

## UIApplication ★ windows

> The app's visible and hidden windows.
>
> This property contains the UIWindow objects currently associated with the app. This list does not include windows created and managed by the system, such as the window used to display the status bar.
>
>The windows in the array are ordered from back to front by window level; thus, the last window in the array is on top of all other app windows.

### window 什么时候会加入到这个数组？

系统创建创建的 window（例如 status bar 的 window）不会被加到该数组

其他情况下，只要一个 window 被初始化了，就会自动加到 windows 数组

### windows 数组元素顺序？

从整体上来看，windows 是根据 window level 逆序排列的

但是如果 level 相同，就是后初始化的排在后面

### 如何改变 windows 元素顺序？

调用 `makeKeyWindow` 或者 `makeKeyAndVisible` 都不会改变 windows 数组的顺序，唯一一个改变的方法，就是改变一个 window 的  level

level 改变之后，当然也是按照 level 逆序排序

level 相同的，后初始化的排在后面（也许是通过比较内存地址来判断初始化顺序 😨）

## 显示在最上层的 window

### 怎么成为最上面的 window？

顶部的 window 肯定是 level 最高的 window，所以想成为最上面的，直接将 level 设置的很大即可

如果出现 level 相同的，情况比较复杂

1. 如果一个 window 在其他 window 调用完各种方法之后，调用 `makeyAndVisible` 方法的可以在最上方，不管前面调用了什么方法，反正只要在最后调用 `makeyAndVisible`，都可以提升到同等级的最上面
2. 如果一个 window 从来没有调用过 `makeyAndVisible` 或 `hidden = NO`，那么可以通过调用 `hidden = NO` 这句代码，可以达到情况 `1` 同样的效果

### 最上面的 window 是不是 windows 数组中最后一个元素?

大多数情况下是，但是有 level 相同的 window 的时候可能就不是了。

因为 window 无论怎么调用 `makeyAndVisible` 或者 `makeKeyWindow` 都是不会影响到一个 window 在 windows 数组中的位置，唯有改变 level 才可以。

当 level 确定之后，windows 数组中的元素顺序肯定就不会变了

但是调用 `makeyAndVisible` 方法是可以改变同等级的 window 的上下关系的，所以最上面的 window 不一定就是 windows 数组中最后一个 window

说极端一点的话，甚至可能出现这样的情况，**一个 window 既在 windows 数组的最后一个，并且是 keyWindow，但是它实际上并不是显示在最上面的那一个 window**

### <span id="a">如何取到真正的显示在最上面的 window？</span>

这是一个问题 ⁉️ 🤣

我只能说，如果我们忽略掉 level 相同的那种特殊情况，那我们大多数情况可以这么找，基本不会出问题

```objc
- (UIWindow *)topWindow
{
    UIWindow *topWindow = nil;
    NSArray <UIWindow *>*windows = [UIApplication sharedApplication].windows;
    for (UIWindow *window in windows.reverseObjectEnumerator) {
        if (window.hidden == YES || window.opaque == NO) {
            // 隐藏的 或者 透明的 跳过
            continue;
        }
        if (CGRectEqualToRect(window.bounds, [UIScreen mainScreen].bounds) == NO) {
            // 不是全屏的 跳过
            continue;
        }
        topWindow = window;
        break;
    }
    return topWindow?:[UIApplication sharedApplication].delegate.window;
}
```

## 几个方法

### makeKeyAndVisible 和 makeKeyWindow 的区别？

`makeKeyAndVisible` 会改变显示效果，并设置 keyWindow。其内部调用了 `makeKeyWindow`，并且设置了 `hidden = NO`，除此之外应该还有一些其他操作~

`makeKeyWindow` 只会设置 keyWindow

### 创建了一个 window，需要显示，一定要调用 makeKeyAndVisible 吗？

不一定，其实直接调用 `hidden = NO`，也可以显示的。

官方是这样说的：

> This is a convenience method to show the current window and position it in front of all other windows at the same level or lower. If you only want to show the window, change its hidden property to NO.

大概意思就是说，`makeKeyAndVisible` 方法就是为了方便，大多数 app 调用这个方法来显示 `main window` 并且 设置为 `keyWindow`，并且将其放在 level 小于等于它的 window 之上。如果只是想要显示一个 window，直接修改它的 hidden 属性，设置为 NO 就可以了。

那么其实，我们如果想让一个 window 显示，并且不想让它成为 keyWindow 的话，就直接设置

```objc
self.window.hidden = NO;
```

这样是最好的

### makeKeyWindow 什么时候会被调用？

makeKeyWindow 方法的作用就是将一个 window 设置为 keyWindow

1. 调用 `makeKeyAndVisible` 方法内部会调用 `makeKeyWindow` 方法 
2. 设置 keyWindow 的 `hidden = YES` 的时候，系统会找一个 window，设置为 keyWindow，所以也会调用 `makeKeyWindow`

### window 的 setHidden: 方法

一个 window 被创建的时候，其 hidden 属性默认是为 YES 的。

当设置 `hidden = NO` 即可显示一个 window，无需将 window 添加到一个 superview 上面。

当设置 `hidden = YES` 会将这个 window 隐藏。如果这个 window 是 keyWindow，那么这个 `setHidden:` 的方法内部会取消当前 window 是 keyWindow 的设置，并且会找一个新的 window 来设置为 keyWindow

## 使用时的注意点

因为应用内长期滥用 window，很多弹窗浮层都是 window，并且有些第三方的 SDK 里面也搞了一些奇奇怪怪的 window。导致很多时候在获取 window 会出现一些乱七八糟的问题，目前也没有找到很好的方案。

所以，个人认为，如果不是必须使用 window，就最好不要使用 window，因为 window 多了之后，很不方便管理。可以使 view 或者 viewController 代替~

实在要用的时候，注意以下问题

### 添加一个视图到最上层的 window

不要使用 

* `[[UIApplication sharedApplication] keyWindow]`
* `[[[UIApplication sharedApplication] windows] lastObject]`

这两个都不一定是最上层的 window，而且很有可能有问题，最终发现自己的视图不知道添加到哪里去了，所以最好这样找

参考前面 [如何取到真正的显示在最上面的 window？](#a)

这个方法不一定对，但是相对大部分情况下都是适用的，目前没有找到更完美的解决方法

### 如果你只是想显示一个 window，不想它变成 keyWindow

不要使用 

```objc
[self.window makeKeyAndVisible];
```

直接设置 `hidden = NO`，即

```objc
self.window.hidden = NO;
```

这样可以将它显示出来，并且不会设置为 keyWindow。如果没显示正常，检查一下 window level 是否正确。

### 如果想让一个 winow 永远不要变成 keyWindow

最好是重写 becomeKeyWindow 方法，调用 `[super becomeKeyWindow]` 之后，找到实际显示在最上层的全屏 window，将其设置为 keyWindow

之前已经说过，找实际显示在最上层的 window 其实是有一定问题的（level 相同的情况），将就用吧 😂 ，实在没找到更好的方法，如果你有有更好的办法的话，请告诉我~

## 参考

* <https://developer.apple.com/documentation/uikit/uiapplication/1622924-keywindow?language=objc>
* <https://developer.apple.com/documentation/uikit/uiapplication/1623104-windows?language=objc>
* <https://developer.apple.com/documentation/uikit/uiwindow?language=objc#>
* <https://developer.apple.com/documentation/uikit/uiwindow/1621601-makekeyandvisible?language=objc>
* <https://bugtags.kf5.com/hc/kb/article/77692/>
* <https://jkyin.me/uiwindow/>

> * 本文永久更新链接：<https://ripperhe.com/2018/09/03/ios-uiwindow-second>
> * 作者：[ripperhe](https://github.com/ripperhe)