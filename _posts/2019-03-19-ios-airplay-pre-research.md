---
layout: post
title: AirPlay 投屏调研
categories: iOS
description: AirPlay 的一些基础认识
keywords: iOS, AirPlay
---

苹果投屏功能主要基于 AirPlay，可以隔空播放音视频和图片。苹果原生的 AirPlay 有发送端和接收端之分，分别为以下设备

**发送端**：

1. iPhone
2. iPad
3. iPod touch
4. Mac

**接收端**：Apple TV

按照常规的使用方法，只能投到 Apple TV 上面。

虽然 AirPlay 传输协议是苹果私有的，不过很多厂商逆向破解了 AirPlay 传输协议，所以只要国内的各种安卓电视盒子安装了对应的接收软件，也能够接收 iPhone 的投屏。

AirPlay 使用的时候有几种使用模式，在 iPhone 上目前主要分为两种模式：

1. Mirror device's screen （屏幕镜像）
2. AirPlay video （隔空播放音视频）

## Mirror device's screen

屏幕镜像模式是一种全局的操作，在手机的控制面板开启。一般来说，在该模式下，手机上显示什么，TV 上就显示什么，直接将手机内容展示到 TV 的显示器上面。

虽然这种模式叫镜像模式，但是，其实在这种模式下，APP 内部可以写代码检测是否开启了镜像模式，可以针对 Apple TV 的屏幕写一屏新的 UI，也就是手机和 TV 的显示屏展示内容可以不同，相当于外接了一个屏幕，展示新的内容。通过这种方法，可以把手机页面作为一个遥控器，真正展示内容可以放到 TV 的屏幕。

在使用镜像模式的时候，如果手机锁屏，TV 端也看不到任何内容了。也就是说，在这种模式下，手机必须一直是常亮的。个人认为这种模式比较适合用来做操作演示，不太适合用来播放视频。

镜像方式开启方法：<https://support.apple.com/zh-cn/HT204289#mirroriOS>

## AirPlay video

隔空播放的方式其实是在嵌入在某个 APP 内部的，在 APP 内部进行操作之后，可以隔空将音视频和图片投送到 TV 上播放。

这种方式就像是把一个文件推送到 TV 端，然后用 TV 的屏幕进行播放，例如：

1. 腾讯视频 APP 投视频
2. 得到 APP 投音频
3. 系统相册投图片

该方式需要 APP 开发者在软件内部写一些代码来支持投屏，比较适合单个视频的播放。比如用腾讯视频看电影的时候就可以把视频投到 TV 上去播放，这个时候手机是可以直接锁屏的，投放的效果也非常不错。

但是这种方式仅仅是对文件的隔空播放，尚未发现可以自定义 TV 端播放页面的 API。

隔空播放开启方法：<https://support.apple.com/zh-cn/HT204289#iOS>

## Apple TV 之外的设备接收 AirPlay

一般来说只能用 Apple TV 作为接收端，不过有很多第三方破解了苹果的传输协议，所以很多平台通过安装一个接收端软件来模仿 Apple TV 接收 iPhone 等设备投送的数据。

以下为一些第三方接收软件

### AirServer

该软件有 PC 和 Mac 版本的接收端，安装后可以直接接收 iPhone 投送的数据。如果想将 Mac 作为 AirPlay 接收端进行投屏，该软件是首选。

产品官网：<https://www.airserver.com>

### 乐播投屏

乐播投屏有安卓电视盒子的 TV 版本的接收端，电视盒子安卓该软件之后可以直接接受数据。

目前国内的很多厂商的电视盒子都内置了乐播投屏的 TV 版本的接收端软件，安装该软件之后，即可直接接收 iPhone 的 AirPlay 投屏。

而且乐播投屏不仅有接受端的软件，还有 Android 手机和 PC 的发送端软件，安装该软件之后，可以像 iPhone 一样直接投屏，然后乐播投屏的 TV 端可以接收到数据。

产品官网：<http://www.hpplay.com.cn>

## 玄学问题

在镜像模式下手机端频繁切换和播放视频，AirPlay 会莫名其妙地断开。因为我是在 Mac 上安装 AirServer 进行测试的，所以不知道是 AirPlay 本身的问题，还是 AirServer 的问题。

## 参考

* <https://developer.apple.com/airplay/>
* <https://developer.apple.com/library/archive/documentation/AudioVideo/Conceptual/AirPlayGuide/Introduction/Introduction.html>
* <https://support.apple.com/zh-cn/HT204289>
* <https://www.airserver.com/>
* <http://www.hpplay.com.cn/>
