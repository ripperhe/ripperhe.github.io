---
layout: post
title:  CYLTabBarController 源码简读
categories: iOS
description: CYLTabBarController 的阅读
keywords: iOS, 源码, CYLTabBarController
---

| 仓库 | 版本 |
| :--- | :--- |
| [ChenYilong/CYLTabBarController](https://github.com/ChenYilong/CYLTabBarController) | 1.10.0 |

CYLTabBarController 是一个自定义的 TabBarController 框架，支持高度自定义，功能强大，使用起来非常方便。

![](https://raw.githubusercontent.com/ripperhe/Resource/master/20170116/1.png)

## 类结构

![](https://raw.githubusercontent.com/ripperhe/Resource/master/20170116/2.png)

1. **CYLTabBarController**

	继承自 `UITabBarController`，可以说该类为本框架的核心。

	2. **NSObject+CYLTabBarControllerItemInternal**
	
		该类是一个内部类，写在 CYLTabBarController 内。主要实现了一个 set 方法，利用 runtime 设置 `cyl_tabBarController` 属性。
		
	3. **NSObjec+CYLTabBarController**
		
		该类则与上面这个类对应，实现了一个 get 方法，用于获取最近的 `cyl_tabBarController`。
	
4. **CYLTabBar**
	
	该类继承 `UITabBar`，处理各个 item 图片、文字等，同时处理中间大按钮的点击事件。

5. **CYLPlusButton**

	该类就是常见的中间大按钮，继承自 `UIButton`。定义了 `CYLPlusButtonSubclassing` 协议，用于子类实现，设置样式等。

6. **UIViewController+CYLTabBarControllerExtention**

	该分类提供了一些很实用的方法，例如退到当前 `NavigationController` 的栈底，并改变 `TabBarController` 的 `selectedViewController` 属性...
	
7. **CYLConstants**

	该头文件定义了一个用于设置废弃 API 废弃的宏，方便。
	
## 使用
	
### 创建控制器

```objc
CYLTabBarController *tabBarController = [CYLTabBarController tabBarControllerWithViewControllers:viewControllers
                                                                           tabBarItemsAttributes:tabBarItemsAttributesForController];
```

仅需传入所有自定义的控制器，并将配置全部传入即可。

### 配置大按钮

写一个子类继承 `CYLPlusButton`，并且遵守 `CYLPlusButtonSubclassing` 协议，实现相应方法即可。

具体细节，查看作者 [Demo](https://github.com/ChenYilong/CYLTabBarController)，这里不再赘述。

## 框架到底做了什么

### CYLTabBarController

1. 从初始化控制器开始

	`CYLTabBarController` 的初始化方法中
	
	```objc
	_tabBarItemsAttributes = tabBarItemsAttributes;
	self.viewControllers = viewControllers;
	```
	
	将传入的 itemsAttributes 赋值，并调用 viewControllers 的 set 方法。	
2. `CYLTabBarController` 的 `setViewControllers` 方法

	1. 如果当前 `_viewControllers` 有值，移除所有，算是做个保护吧
	2. 判断 `viewControllers` 有值，并且 `viewControllers` 为一个数组
		1. YES
			1. 检测 `_tabBarItemsAttributes` 有值，并且 `_tabBarItemsAttributes` 与 `viewControllers` 个数相同，如果不对，抛出错误
			2. 检测是否有 CYLPlusButton 对应的控制器，如果有，根据 CYLPlusButtonIndex 插入到 `viewControllers` 对应位置。同时，给 `_viewControllers` 赋值。
			3. 给 CYLTabbarItemsCount、CYLTabBarItemWidth 等属性赋值
			4. 遍历 `_viewControllers`
				1. 从 `_tabBarItemsAttributes` 中获取到各个 item 信息（避开 `CYLPlusChildViewController` ）
				2. 调用 [配置一个子控制器的方法](#配置一个子控制器的方法)
				3. 利用 `NSObject+CYLTabBarControllerItemInternal` 分类中的 set 方法，设置该 viewController 的 `cyl_tabBarController` 属性。
		2. NO
			1. 将每 viewController 的 `cyl_tabBarController` 设空
			2. `_viewControllers` 设空
			
3. <a name="配置一个子控制器的方法"></a>配置一个子控制器的方法
	
	```objc
	- (void)addOneChildViewController:(UIViewController *)viewController
                        WithTitle:(NSString *)title
                  normalImageInfo:(id)normalImageInfo
                selectedImageInfo:(id)selectedImageInfo
    ```
				
	1. 设置标题
	1. 正常状态图片
	2. 选中状态图片
	3. 设置需要自定义的图片 Insets
	4. 设置需要自定义的标题 Insets
	5. 添加该子控制器
	
4. `viewDidLoad` 方法

	1. 利用 KVC 设置 `tabBar` 属性。（`tabBar` 属性是只读的）
	2. 监听 `tabBar` 的 `swappableImageViewDefaultOffset`。用于动态设计图片偏移
	
5. `viewDidLayoutSubviews` 方法

	1. 调用 tabBar 的 `layoutSubviews`

6. `viewWillLayoutSubviews` 方法
	
	1. 更新 tabBar 的 frame

7. `supportedInterfaceOrientations` 方法，设置支持的横竖屏方向

	1. 获取当前 `selectedViewController`，判断是否为 `UINavigationController`
		1. YES，获取其 `topViewController`，并返回改控制器的 `supportedInterfaceOrientations` 属性
		2. NO，返回 `selectedViewController` 的 `supportedInterfaceOrientations` 

8. 最后在说一下，`NSObjec+CYLTabBarController` 的 `cyl_tabBarController` 方法是如何实现的

	这个一个 `NSObjec` 分类，即可让任意对象调用

	1. 利用 runtime 方法 `objc_getAssociatedObject(id object, const void *key)` 获取 `cyl_tabBarController`，如果有值，直接返回 
	2. 判断当前对象是 `UIViewController` 并且 `parentViewController` 有值
		1. 让其 `parentViewController` 继续调用 `cyl_tabBarController`，形成递归
		2. 最后，将递归结果返回
	3. 获取到 `appdelegate` 的 `window`，取出其 `rootViewController`，如果是 `CYLTabBarController` 控制器，将其返回

### CYLTabBar

1. 初始化方法
	1. 如果有 `CYLExternPlusButton`，`add` 到自己上
	2. 设置 `_tabBarItemWidth = CYLTabBarItemWidth`
	3. 监听 `tabBarItemWidth` 属性变化
2. `layoutSubviews` 方法
	1. 设置 `CYLTabBarItemWidth`
	2. 将子 view 排序 `sortedSubviews`，没看太明白，备注如下
		
		> Deal with some trickiness by Apple, You do not need to understand this method, somehow, it works.
		> 
		> NOTE: If the `self.title of ViewController` and `the correct title of tabBarItemsAttributes` are different, Apple will delete the correct tabBarItem from subViews, and then trigger `-layoutSubviews`, therefore subViews will be in disorder. So we need to rearrange them.
	3. 获取 `tabBarButtonArray`，该方法详情见 [这里](#tabBarButtonFromTabBarSubviews)
	4. 如果有 `CYLExternPlusButton`，调整提UI，并将其 `bringSubviewToFront`
	
3. <a name="tabBarButtonFromTabBarSubviews"></a>tabBarButtonFromTabBarSubviews 方法
	
	该方法还是比较重要的，主要是获取所有 有用的 item，item 真实类型是 `UITabBarButton` 是一个私有类
	
	1. 遍历 tabBarSubviews 将所有 `UITabBarButton` 取出，放入 `self.tabBarButtonArray`
	2. 如果有 `CYLPlusChildViewController`，将对应位置的 `UITabBarButton` 从数组中移除（不是移除 view）

4. 除此之外，比较重要的就是重写 `hitTest` 方法

	截获手势触摸，并根据位置进行返回对应控件，进行事件处理
	
	```objc
	- (UIView *)hitTest:(CGPoint)point withEvent:(UIEvent *)event {
		BOOL canNotResponseEvent = self.hidden || (self.alpha <= 0.01f) || (self.userInteractionEnabled == NO);
		if (canNotResponseEvent) {
		    return nil;
		}
		if (!CYLExternPlusButton && ![self pointInside:point withEvent:event]) {
		    return nil;
		}
		if (CYLExternPlusButton) {
		    CGRect plusButtonFrame = self.plusButton.frame;
		    BOOL isInPlusButtonFrame = CGRectContainsPoint(plusButtonFrame, point);
		    if (!isInPlusButtonFrame && (point.y < 0) ) {
		        return nil;
		    }
		    if (isInPlusButtonFrame) {
		        return CYLExternPlusButton;
		    }
		}
		NSArray *tabBarButtons = self.tabBarButtonArray;
		if (self.tabBarButtonArray.count == 0) {
		    tabBarButtons = [self tabBarButtonFromTabBarSubviews:self.subviews];
		}
		for (NSUInteger index = 0; index < tabBarButtons.count; index++) {
		    UIView *selectedTabBarButton = tabBarButtons[index];
		    CGRect selectedTabBarButtonFrame = selectedTabBarButton.frame;
		    if (CGRectContainsPoint(selectedTabBarButtonFrame, point)) {
		        return selectedTabBarButton;
		    }
		}
		return nil;
	}
	```
	
	1. 先判断是否需要接受触摸
	2. 如果点在当前控件外，并且没有 `CYLExternPlusButton`，直接返回
	3. 如果有 `CYLExternPlusButton`
		1. 判断触摸点是否在其内部，如果是，将 `CYLExternPlusButton` 返回
	4. 遍历 `tabBarButtonArray`，遍历获取每个普通 item
		1. 判断触摸点是否在其内部，如果在，直接返回
	
### CYLPlusButton

1. 最重要的就是一个注册方法 registerPlusButton
	
	```objc
	+ (void)registerPlusButton;
	```
	 
	该方法是让 `CYLPlusButton` 的子类调用，并且子类必须遵守 [`CYLPlusButtonSubclassing`](#CYLPlusButtonSubclassing) 协议，该方法内容如下
	
	1. 利用子类类方法 `plusButton ` 获取 button 实例
	2. 给 `CYLExternPlusButton`、 `CYLPlusButtonWidth` 等全局变量赋值
	3. 如果子类实现了 `plusChildViewController`方法
		1. 将其赋值给 `CYLPlusChildViewController`
		2. 添加点击事件
		3. 是否实现 `indexOfPlusButtonInTabBar`
			1. YES，将其赋值给 `CYLPlusButtonIndex`
			2. NO，抛出异常，`PlusChildViewController` 和 `indexOfPlusButtonInTabBar` 配套使用
		

2. <a name="CYLPlusButtonSubclassing"></a>`CYLPlusButtonSubclassing` 协议

	```objc
	@required
	// 必须实现 用于初始化 button
	+ (id)plusButton;
	
	@optional
	// 按钮的 index
	+ (NSUInteger)indexOfPlusButtonInTabBar;
	// 高度相关
	+ (CGFloat)multiplierOfTabBarHeight:(CGFloat)tabBarHeight;
	+ (CGFloat)constantOfPlusButtonCenterYOffsetForTabBarHeight:(CGFloat)tabBarHeight;
	// 按钮对应的控制器
	+ (UIViewController *)plusChildViewController;
	// 是否显示按钮选中状态
	+ (BOOL)shouldSelectPlusChildViewController;
	```
	
### UIViewController+CYLTabBarControllerExtention

以 `cyl_popSelectTabBarChildViewControllerAtIndex` 为例子，如要是利用 popToRootViewControllerAnimated 退出到栈底，抽离出来确实比较方便。其他的不再一一列举。

```objc
- (UIViewController *)cyl_popSelectTabBarChildViewControllerAtIndex:(NSUInteger)index {
    [self checkTabBarChildControllerValidityAtIndex:index];
    [self.navigationController popToRootViewControllerAnimated:NO];
    CYLTabBarController *tabBarController = [self cyl_tabBarController];
    tabBarController.selectedIndex = index;
    UIViewController *selectedTabBarChildViewController = tabBarController.selectedViewController;
    return [selectedTabBarChildViewController cyl_getViewControllerInsteadIOfNavigationController];
}
```

### CYLConstants

定义了一个宏，标志一个方法被废弃了

```objc
#define CYL_DEPRECATED(explain) __attribute__((deprecated(explain)))
```

举例

```objc
+ (CGFloat)multiplerInCenterY CYL_DEPRECATED("Deprecated in 1.6.0. Use `+multiplierOfTabBarHeight:` instead.");
```

使用时废弃的方法会出现如下警告 ⚠️

![](https://raw.githubusercontent.com/ripperhe/Resource/master/20170116/3.png)

## 总结

第一次去读别人的源码，说实话，真的不知道怎样子写。可能我的文章写的很乱，不过自己读一遍再用伪代码写一遍整体思路，感觉还是学到了不少东西，自己受益了。

> * 本文永久更新链接：<https://ripperhe.com/2017/01/16/cyltabbarcontroller>
> * 作者：[ripperhe](https://github.com/ripperhe)