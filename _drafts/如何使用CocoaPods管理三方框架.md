---
layout: post
title: 如何使用CocoaPods管理三方框架
category: iOS
description: cocoapods的简单使用
keywords: cocoapods, 三方库
---

#### CocoaPods是什么
CocoaPods是开发macOS和iOS应用程序的一个第三方库的依赖管理工具。利用它管理三方库，在三方库版本更新的时候尤为方便。

#### 安装CocoaPods
安装CocoaPods需要ruby环境，不过mac上默认自带ruby环境。国内访问[RubyGems](https://rubygems.org/)向来比较困难，不过Ruby China社区做了一个国内[RubyGems 镜像](http://gems.ruby-china.org/)，我们只需要将源地址改为ruby-china即可。

```
➜  ~ gem source -l
```
如果没有修改过source，结果会是如下

```
*** CURRENT SOURCES ***

https://rubygems.org/
```

接下来执行以下命令添加新的source

```
➜  ~ gem sources --add https://gems.ruby-china.org/
```
并移除旧的source

```
➜  ~ gem sources --remove https://rubygems.org/
```
这时再执行第一条命令，结果如下就对了

```
➜  ~ gem source -l
*** CURRENT SOURCES ***

https://gems.ruby-china.org/
```
接下来安装CocoaPods

```
➜  ~ sudo gem install cocoapods
```
没有报错的话，这样就安装完成了，当想要升级CocoaPods的时候，也是利用上面这条命令。
完成之后，执行以下命令配置本地pod仓库,可能需要点时间

```
➜  ~ pod setup
```

#### 将pod集成到iOS工程
先利用终端切换到需要使用Pod的工程文件夹内，初始化pod文件

```
➜  ~ pod init
```
现在工程文件夹内会出现一个名叫Podfile的文件，接下来搜索想要导入的三方库

```
➜  ~ pod search SDWebImage     #以SDWebImage为例 

```










