---
layout: post
title:  iOS 一个实用的悬浮球工具
categories: iOS
description: 悬浮球
keywords: iOS, ZYSuspensionView
---

## 前言

已经很久没有写文章了，前面写过一篇 [关于 UIWindow 的文章](http://www.jianshu.com/p/98cd7fc4bfba)，在文章的最后提过悬浮球。用 UIWindow 来做悬浮球，可以悬浮在任意页面，基于这个特性，我将其做成了一个工具，用在我们的项目开发中，今天把它作为主角来讲一讲。

先上代码 → [ZYSuspensionView GitHub 传送门](https://github.com/ripperhe/ZYSuspensionView)

## 为什么会有它

### 想要指定执行一些代码

在做公司项目的时候，可能会有这样的情况，我需要在第一个进某个页面的时候弹出一些东西，简单做法，可能就会再、在 NSUserDefaults 中记录一个值，表明弹出过。可能我第一次弹出之后又一些问题，我改了代码，运行工程，重新进页面让它弹出，可是却弹不出来了，因为已经标记为弹出过，把包删了，重运行工程即可。

![testmanager.gif](http://upload-images.jianshu.io/upload_images/939125-c7ec6a881d461bd0.gif?imageMogr2/auto-orient/strip)

不过有了悬浮球，我通过点击悬浮球，弹出一个列表，列表中每个 cell 对应我预先写好的一些常用代码，例如清除 NSUserDefaults 所有键值对等等，然后就达到了想要的效果。当然这个比喻有很多槽点，不过就这样吧，不纠结。

真实用例如下，希望对你有点启发：

1. 通过手动插入一条的指定体重体脂率的体重数据

	我们公司做体脂秤，所以开发的时候经常需要这么做，不然就得光着脚在秤上称，还不一定是自己想要的数值
2. 清除 NSUserDefaults，让包里的标记干净得像初恋一样
3. 显示 log 

	有时候测试人员测试的时候，发现页面的 bug，可能我们在有些 case 预留了一些 log，但是想要导出就麻烦一些，通过点悬浮球列表的某个 cell 跳转到一个显示 log 的页面，快捷方便
	
4. 显示当前账号的 user id 及其他信息

	我们需要经常查当前账号信息
	
5. 跳转到指定页面

	正在开发某个页面，需要依赖工程里的东西，所以在工程里开发，但是还没有开发完，由于入口深，每次改了一些东西重新跑工程还得点几下，麻烦。修改 app 首页的某个按钮点击事件，直接点击进入也不合适，影响正式代码，而且容易忘记改回来。
	
6. 其他一些杂七杂八的...

### 不想每次都输入账号密码

因为项目的原因，很多功能展示都和当前账号的数据有关系，所以想要自测自己的功能是否 OK，就需要频繁切换账号，以保证每种情况都是真实 cover 到了。

![loginmanager.gif](http://upload-images.jianshu.io/upload_images/939125-c954813dc16624d4.gif?imageMogr2/auto-orient/strip)

用悬浮球，弹出一个账号菜单，点击自己想要登陆的账号，直接自动登陆，妈妈再也不用担心我每天都要输入几百次密码了。 🤣


## 好用吗？怎么用？

个人感觉还是很好用了，为自己开发省去了不少时间，同时也为测试同学省去不少麻烦（有点自卖自夸的性质了哈 😠）。页面简单朴素，就是悬浮球 + 一个列表，咱不整那些虚的，主要就是实用。好不好用，试试就知道了。

我已经将这个悬浮球做成了一个 Pod 库，公司的项目用了近一年了，稳定，暂未发现有什么问题，如有 bug 可以给我提 issue。

这个组件主要分为个部分

* SuspensionView
* TestManager
* LoginManager

SuspensionView 就是单纯一个悬浮球，可以单独集成。

### TestManager 

TestManager 是一个单例，将你预先写好的代码块保存好，然后通过点击悬浮列表的某个 cell 执行相应代码块。

使用的时候，一般分为两个部分：

一些是常驻的一些测试代码，例如展示 log，清除 user defaults 等...

```objc
NSArray *baseArray = @[
                       @{
                           kTestTitleKey: @"clean user defaults",
                           kTestAutoCloseKey: @YES,
                           kTestActionKey: ^{
                               NSLog(@"click 'clean user defaults'");
                               [self cleanUserDefaults];
                           }
                           },
                       @{
                           kTestTitleKey:@"something",
                           kTestAutoCloseKey: @NO,
                           kTestActionKey:^{
                               NSLog(@"click 'something'");
                           }
                           },
                       ];
    
[ZYTestManager setupPermanentTestItemArray:baseArray];
```

* `kTestTitleKey` 是测试条目的名字
* `kTestAutoCloseKey` 为点击该条目之后是否自动关闭悬浮列表
* `kTestActionKey` 很显然，这就是点击之后想要执行的代码

当然，一些不常用的测试代码，可能就是你突发奇想要跳转到某个页面，但是不影响项目的其他功能正常运行，就可以使用另一个方法

```objc
[ZYTestManager addTestItemWithTitle:@"new item" autoClose:YES action:^{
    // click new item : do something ~~~~~~~~~~
}];
```
这里需要说明一些，传入 block 的时候，如果用到 self，最好使用 __weak，避免影响到你正常功能，毕竟 TestManager 是一个单例。


### LoginManager

LoginManager 顾名思义，就是拿来做自动登录的，同样，点击 login 悬浮球，弹出账号列表，再点击对于 cell，LoginManager 会回调给你储存好的这个账号的账号和密码。

```objc
- (void)loginManager:(ZYLoginManager *)loginManager loginWithAccout:(NSString *)account password:(NSString *)password
{
	// 这里去实现你的自动登录
}
```

可能你会说，自动登录都是我实现了，拿你还有什么用呢。LoginManager 做了如下事情：

* 帮你存储账号密码
* 列表展示你预先设置或者登录过的账号，并通过你的点击回调给你

自动登录的代码是不可能帮你做好的，因为每个工程里的与登录对于的代码完全不同，不可能通用，所以，咱只能做到这一步了，哈哈哈。不过这里还是给你提供了一个获取当前控制器的方法，可以直接使用 

```
UIViewController *currentVC = [ZYLoginManager currentViewControllerWithWindow:nil];
```

其实自动登录无非就是两步

1. 获取当前控制器，查看是否为登录的控制器
2. 如果是，利用 `performSelector` 调用登录部分的代码

如果不是登录的控制器怎么办呢？一般的 app，在没有登录的时候一般第一个页面就是登录页面，或者点击一下某个按钮，跳转到登录页面，其实获取当前控制器的时候，完全可以判断是否为第一个页面，如果是，就 `performSelector` 跳转到登录页面，再在登录页面 `performSelector` 调用登录部分的代码。根据不同情况再做一些调整吧。

除了是实现这个方法，你还需要做两件事情：

* 告诉我你登录成功了，并且告诉我是哪个账号，我这时候需要隐藏登录的悬浮球，并且为你保存账号

```objc
[[NSNotificationCenter defaultCenter] postNotificationName:@"kZYLoginSuccessNotificationKey"
                                                    object:@{account:password}];
```

* 告诉我你退出登录了，我需要重新展示登录按钮，方便下一次自动登录

```objc
[[NSNotificationCenter defaultCenter] postNotificationName:@"kZYLogoutSuccessNotificationKey"
                                                    object:nil];
```

## 安全吗？

这东西，能跟着上线吗，不小心蹦出来个悬浮球给用户看到那不是尴尬了。不能携带上线的话，每次上线还得删除，多麻烦啊。😎 这个大可放心，我已经带着它上线了 n 个版本了，而且几乎所有代码我都加上了

```
#if DEBUG
...
#endif
```

所以根本不需要担心，只要工程改为 `release` 模式运行压根就出现不了悬浮球，里面的代码根本也不会执行。

我的建议是像我 demo 里面那样，写一个 `config` 类，将上面所有常用的测试代码和测试账号都写在这里类里面，放在一起，将污染做到最小。如果你这样做了，在这里 `config` 类的 `setup` 方法再加上一个 `#if DEBUG`，你会发现 `TestManager` 和 `LoginManager` 这两个单例对象根本就不会初始化，还有什么可担心的。

基本就这么多吧，如果你觉得有用的话，请不要吝啬你的 star🌹 哦！