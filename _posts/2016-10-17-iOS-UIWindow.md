---
layout: default
title:  "iOS UIWindow及悬浮球"
date:   2016-10-17 23:29:00
categories: main
---

# iOS UIWindow及悬浮球

![UIWindow](http://upload-images.jianshu.io/upload_images/939125-37108f75b88c3a3a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#### 1.UIWindow简介

一个**UIWindow**对象为应用程序的用户界面提供了背景以及重要的事件处理行为。
UIWindow继承自UIView，我们一般不会直接去设置其UI展现，但它对展现程序中的views至关重要。每一个view，想要出现在屏幕上都依赖于window，但是程序中的window之间是相互独立的。应用程序收到事件之后会先转发给适当的window对象，从而又将事件转发给view对象。

#### 2.程序中有哪些UIWindow

所有view的展现都依赖于window，创建一个新的iOS工程，将其运行会执行以下事情

* Xcode会自动创建一个window，即`app delegate`中的window属性。
* 同时,Xcode会默认创建一个Main.storyboard,其`instantiateInitialViewController`的显示需要window，依赖的即为前面的window。
* 此时，该window的`rootViewController`即为Main.storyboard的`instantiateInitialViewController`。

很显然，一个应用程序当中，不是只能有一个window，可以存在多个window。已知的有以下window：

* app delegate里的window
* 状态栏的window（比较特殊，虽然在程序内部可以调用某些api显示隐藏或改变其UI，但它的window是不被我们的应用程序内部所持有的）
* 键盘的window

#### 3.获取程序中的UIWindow
`UIApplication`这个类是一个单例类，通过其**sharedApplication**方法进行调用，一个程序可以看做是一个UIApplication对象,可以通过UIApplication对象的以下属性来获取想要的window。

```objc
@property(nullable, nonatomic,readonly) UIWindow *keyWindow;
@property(nonatomic,readonly) NSArray<__kindof UIWindow *>  *windows;
```

**keyWindow** 应用程序的关键window。用来接收键盘以及非触摸类的消息事件的UIWindow，而且程序中每个时刻只能有一个UIWindow是keyWindow。					
**windows** 应用程序中所有的window对象，包括正在显示的或隐藏的window。

新建一个iOS工程,在没有触发键盘时，在控制台打印`winodws`如下：
>(lldb) po [[UIApplication sharedApplication] windows]    
\<__NSArrayM 0x61800024d7d0>(    
\<**UIWindow**: 0x7fd8e1a06370; frame = (0 0; 414 736); autoresize = W+H; gestureRecognizers = \<NSArray: 0x61800024d170>; layer = \<UIWindowLayer: 0x61000003e7c0>>    
)

该window就是 app delegate 的window，即系统自动生成的那个window。当文本编辑，触发键盘之后，打印`windows`如下：
>(lldb) po [[UIApplication sharedApplication] windows]    
\<__NSArrayM 0x61000005d1f0>(   
\<**UIWindow**: 0x7fd8e1a06370; frame = (0 0; 414 736); autoresize = W+H; gestureRecognizers = \<NSArray: 0x61800024d170>; layer = \<UIWindowLayer: 0x61000003e7c0>>,     
\<**UITextEffectsWindow**: 0x7fd8dfc1cde0; frame = (0 0; 414 736); opaque = NO; autoresize = W+H; layer = \<UIWindowLayer: 0x60800022dd60>>,        
\<**UIRemoteKeyboardWindow**: 0x7fd8dfc27e40; frame = (0 0; 414 736); opaque = NO; autoresize = W+H; layer = \<UIWindowLayer: 0x608000231300>>   
)

打印中的`UIRemoteKeyboardWindow`就是键盘的window，与此同时，还出现了`UITextEffectsWindow`，这个window我没有找到官方的说明，不过可以推测它也是和文本输入有关系的。

#### 4.UIWindow的属性与方法

```objc
@property(nonatomic,strong) UIScreen *screen
```
该属性默认为`[UIScreen mainScreen]`，一个`UIScreen`对象对应一个实际设备的物理屏幕，一般情况下，我们不需要对其进行设置。一个iPhone默认也就一个屏幕，一个屏幕可以存在多个window，那也是为什么我们一个程序里面可以有多个window的原因。

当一个iPhone连接一个外接屏幕的时候，系统会发送通知。然而如果我们什么都不做，外接屏幕会一片漆黑，因为在那个屏幕上不存在任何window对象。如果真的想要在外接的屏幕中显示一些东西的话，那就应该监听系统通知，在接收通知的方法里创建一个新的window，并将其显示，当然，断开连接的时候，应该将window对象置为nil释放。以下为官方示例代码：

```objc
- (void)handleScreenConnectNotification:(NSNotification*)aNotification {
    UIScreen*    newScreen = [aNotification object];
    CGRect        screenBounds = newScreen.bounds;
 
    if (!_secondWindow) {
        _secondWindow = [[UIWindow alloc] initWithFrame:screenBounds];
        _secondWindow.screen = newScreen;
 
        // Set the initial UI for the window and show it.
        [self.viewController displaySelectionInSecondaryWindow:_secondWindow];
        [_secondWindow makeKeyAndVisible];
    }
}
 
- (void)handleScreenDisconnectNotification:(NSNotification*)aNotification {
    if (_secondWindow) {
        // Hide and then delete the window.
        _secondWindow.hidden = YES;
        [_secondWindow release];
        _secondWindow = nil;
 
        // Update the main screen based on what is showing here.
        [self.viewController displaySelectionOnMainScreen];
    }
}
```
---
```objc
@property(nonatomic) UIWindowLevel windowLevel;
```
window等级，即window在z轴上的层级关系,默认是0。`UIWindowLevel`本身是一个`CGFloat`类型,可以随意设置或进行加减，高等级会显示在低等级上面。系统给出了三种常用等级:

```objc
UIKIT_EXTERN const UIWindowLevel UIWindowLevelNormal;      0
UIKIT_EXTERN const UIWindowLevel UIWindowLevelAlert;       2000
UIKIT_EXTERN const UIWindowLevel UIWindowLevelStatusBar;   4000
```
---
```objc
@property(nonatomic,readonly,getter=isKeyWindow) BOOL keyWindow;
```
当前window对象是否为程序的keyWindow，系统会自动赋值更新，我们不需要，也不能手动设置。

---
```objc
- (void)becomeKeyWindow;// override point for subclass. Do not call directly
- (void)resignKeyWindow;// override point for subclass. Do not call directly
```
这个是在继承的时候进行重写的，不要手动去调用。在一个window的keyWindow属性改变时会调用，当你写一个子类继承UIWindow,如果需要在window变成keyWindow,或是keyWinow变为NO的时候想做一些事情，就可以重写这两个方法，以下为官方解释。
>You should rarely need to subclass UIWindow. The kinds of behaviors you might implement in a window can usually be implemented in a higher-level view controller more easily. One of the few times you might want to subclass is to override the becomeKeyWindow or resignKeyWindow methods to implement custom behaviors when a window’s key status changes.

---
```objc
- (void)makeKeyWindow;
- (void)makeKeyAndVisible;
```
一个window的hideen属性默认是YES的，`makeKeyWindow`是将一个window设置为keyWindow，但是`makeKeyAndVisible`会将一个window设置为keyWindow并将其显示。如何没有变成keyWindow,则其内部的文本框没法输入文字。

> UIWindow: 0x12dd3ef20; frame = (0 200; 200 200); hidden = YES; gestureRecognizers = \<NSArray: 0x12dd40530>; layer = \<UIWindowLayer: 0x12dd3f230>    
> UIWindow: 0x12dd3ef20; frame = (0 200; 200 200); gestureRecognizers = \<NSArray: 0x12dd40530>; layer = \<UIWindowLayer: 0x12dd3f230>

以上为将一个window调用`makeKeyAndVisible`前后对比，可以发现，其hidden从YES变为NO。所以某个window调用`makeKeyAndVisible`之后，系统对该window至少做了以下事情：

* 将UIApplication对象的keyWindow设置为当前这个window
* 当前window的`hidden`设置为NO，同时该window的keyWindow属性变为YES

---
```objc
@property(nullable, nonatomic,strong) UIViewController *rootViewController;
```
该属性为window的根控制器，现在这个属性是不能为空的，必须进行赋值，否则程序会崩溃。

---
```objc
- (void)sendEvent:(UIEvent *)event;
```
有事件需要处理的时候UIApplication会调用该方法派发事件。

---
```objc
- (CGPoint)convertPoint:(CGPoint)point toWindow:(nullable UIWindow *)window;
- (CGPoint)convertPoint:(CGPoint)point fromWindow:(nullable UIWindow *)window;
- (CGRect)convertRect:(CGRect)rect toWindow:(nullable UIWindow *)window;
- (CGRect)convertRect:(CGRect)rect fromWindow:(nullable UIWindow *)window;
```
window之间是相互独立的，如果想要将两个window的坐标相互映射的时候，就需要用到以上几个方法。

#### 5.如何创建一个UIWindow并显示

主要有以下几个步骤：
> 1. 创建一个window对象，并用一个对象**强持有**它
> 2. 创建一个控制器，赋值为window的根控制器
> 3. 显示窗口

代码如下：

```objc
//1. 创建一个window对象，并用一个对象强持有它
UIWindow *testWindow = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];
self.testWindow = testWindow;
//2. 创建一个控制器，赋值为window的根控制器
UIViewController *controller = [[UIViewController alloc] init];
testWindow.rootViewController = controller;
//3. 显示窗口
[testWindow makeKeyAndVisible];
```
这里，需要注意一下：

* window的frame决定了这个窗口大小，所以需要进行设置
* 新的控制器之所以能正常显示，是因为window强持有它，window能正常运行，则是因为我们用了一个暂时不会销毁的对象强持有window（当然，直接用一个静态变量持有也可以，本质上是一样的）。
* 如果window看不见，可以试试修改以下其`windowLevel`属性。高等级的window一定会显示在低等级的window上面，同等级的window,后makeKeyWindow的window就会显示在上面。
* 无论是通过代码,storyboard或xib初始化一个控制器来显示，都是以上三步，只是创建控制器的方法有所区别罢了，这里不做讨论了。

#### 6.如何销毁一个UIWindow

前面已经说过，对于一个UIWindow对象，之所以显示，是因为有一个对象强持有它，要销毁一个window，只需要将这个强持有去掉即可。但是,这种持有去掉之后，可能window可能不会立即消失，所以，为了确保能够立即将其不展现，最好按以下步骤：
> 1. 将window的hidden属性置为YES
> 2. 将持有该window的那个对象对window的持有去掉（有点绕😄）

代码如下：

```objc
self.testWindow.hidden = YES;
self.testWindow = nil;
```
假如当前这个window是keyWindow，这个window被销毁之后，系统会自动将上一个keyWindow设置为keyWindow，不需要我们去管理。简单说就是假如以 A->B->C 这个顺序变为keyWindow之后，C销毁了，B会自动变为keyWindow。需要注意的是，不要去调用`resignKeyWindow`方法，该方法是用于子类重写的，手动调用之后，结果也是未知的。

#### 7.我们什么时候需要自己创建一个UIWindow

苹果官方是这么说的😝
>Most apps need only one window, which displays the app’s content on the device’s main screen. You can create additional windows and display them on the device’s main screen, but extra windows are more commonly used to display content on an attached external display.

新建的UIWindow一般用于外接的屏幕，那在我们手机的主屏幕什么时候会有这种需求呢？我觉得，如果我们需要个一个控件，需要独立于其他的view,并悬浮于应用程序中的时候，也许就需要用到UIWindow了，这里所谓的悬浮，不过就是windowLevel比较高罢了。

公司工程里所集成的测试控件[Bugtags](https://www.bugtags.com/)就是利用UIWindow实现的，可以悬浮在任意页面，主要用于测试人员提bug，直接手机上提bug。当然提bug这件事和本文关系不大，在此只是想表明这种情况就可以用UIWindow。

#### 8.关于悬浮球
对于这个可拉拽的悬浮球，我也比较好奇，所以自己着手实现了一下，原理也挺简单。

* 创建一个按钮大小的window并显示
* 将其windowLevel设置得较高
* 在按钮上添加拖拽手势，随着手势移动，并添加一些边界控制

那就有人问了，这个东西有什么用？

因为公司的工程里确实没有什么需要需要用到这个东西，但是我后来发现这个东西还是有那么一点用😁。不过不是用在正式代码之中，而是开发测试阶段。

* 做个一键登陆功能（公司的项目开发需要频繁换号，输密码太麻烦）
	* 如果不用换账号，直接写死一个账号，点击悬浮球直接登录
	* 如果需要频繁换账号的，可以把登录过的账号都记录下来，写到NSUserDefaults等地方，以后每次需要登陆时，点击浮球，出来一个列表，选其中一个登陆
* 一些调试的时候想要反复执行的某句代码

---
✨具体细节可到GitHub下载demo查看。[GitHub地址 😁](https://github.com/ripperhe/ZYSuspensionView)		
✨如果有用，还望朋友能给个star，谢谢。