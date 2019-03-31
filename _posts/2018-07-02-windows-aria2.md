---
layout: post
title:  Windows 下使用 BaiduExporter + Aria2 下载百度网盘文件
categories: Windows
description: 使用 Aria2 下载文件
keywords: Windows, Aria2, BaiduExporter
---


## 简介

百度盘下载限速，Aria2 堪称神器，大大加大下载速度，最近研究了一下如何在 Windows 下使用 Aria2，需要以下三个工具配合使用：

* [BaiduExporter](https://github.com/acgotaku/BaiduExporter)：百度云盘导出下载的 `Chrome` 插件
* [Aria2](https://github.com/aria2/aria2)：下载工具
* [Yaaw](https://github.com/binux/yaaw)：在线显示 Aria2 下载进度的 [Web UI](http://binux.github.io/yaaw/demo/#)

文章文件分享地址：

* [https://pan.baidu.com/s/17Dme1SMNYJ3HQfftghO_bg](https://pan.baidu.com/s/17Dme1SMNYJ3HQfftghO_bg)

## 安装 BaiduExporter

### 下载安装

1. 进入网址下载 [https://github.com/acgotaku/BaiduExporter](https://github.com/acgotaku/BaiduExporter)
	![](https://raw.githubusercontent.com/ripperhe/Resource/master/20180702/snip_20180701212558.png)
2. 解压文件夹，在浏览器输入 `chrome://extensions` 进入扩展程序页面，选中页面右上角的 `开发者模式`，然后将`BaiduExporter.crx` 文件拖入 Chrome 扩展程序页面，进行安装
	![](https://raw.githubusercontent.com/ripperhe/Resource/master/20180702/snip_20180701100117.png)

### 加入白名单

因为不是 Chrome 扩展程序商店直接安装的扩展，所以需要加入白名单，否则会影响使用

1. 下载 [chrome.adm](https://pan.baidu.com/s/17Dme1SMNYJ3HQfftghO_bg)
2. Win+R 调起 `运行` 程序，输入 `gpedit.msc`，进入 `本地组策略编辑器`sdf
	![](https://raw.githubusercontent.com/ripperhe/Resource/master/20180702/snip_20180702133428.png)
3. 选中 `计算器配置`——`管理模板`，右键调出菜单，点击 `添加/删除模板`
	![](https://raw.githubusercontent.com/ripperhe/Resource/master/20180702/Jietu20180702-102755.png)
4. 在 `添加/删除模板` 页面，点击 `添加`，找到之前下载的 `chrome.adm`，选中打开，点击关闭
	![](https://raw.githubusercontent.com/ripperhe/Resource/master/20180702/Jietu20180702-102918@2x.png)
5. 选中 `计算器配置`——`管理模板`——`经典管理模板(ADM)`——`Google`——`Google Chrome`——`扩展程序`——`配置扩展程序安装白名单`，打开
	![](https://raw.githubusercontent.com/ripperhe/Resource/master/20180702/snip_20180702140402.png)
6. 在 `配置扩展程序安装白名单` 页面，点击 `已启用`，点击 `显示`
	![](https://raw.githubusercontent.com/ripperhe/Resource/master/20180702/Jietu20180702-103532.png)
7. 在 `显示内容` 页面，输入在 Chrome 浏览器扩展程序页面拷贝的 `BaiduExporter` 插件的 ID，点击 `确定`
	![](https://raw.githubusercontent.com/ripperhe/Resource/master/20180702/snip_20180702100117.png)
	![](https://raw.githubusercontent.com/ripperhe/Resource/master/20180702/Jietu20180702-103732.png)

> 一般情况下，到这儿就 OK 了，不过我遇到一次这样也没法解决，不过通过添加到 `配置强制安装的扩展程序的列表` 解决了，接着上面继续操作

1. 选中 `计算器配置`——`管理模板`——`经典管理模板(ADM)`——`Google`——`Google Chrome`——`扩展程序`——`配置强制安装的扩展程序的列表`，打开
	![](https://raw.githubusercontent.com/ripperhe/Resource/master/20180702/Jietu20180702-104124.png)
2. 在 `配置强制安装的扩展程序的列表` 页面，点击 `已启用`，点击 `显示`
	![](https://raw.githubusercontent.com/ripperhe/Resource/master/20180702/Jietu20180702-104218.png)
3. 在 `显示内容` 页面，输入在 Chrome 浏览器扩展程序页面拷贝的 `BaiduExporter` 插件的 ID，点击 `确定`
	![](https://raw.githubusercontent.com/ripperhe/Resource/master/20180702/snip_20180702100117.png)
	![](https://raw.githubusercontent.com/ripperhe/Resource/master/20180702/Jietu20180702-104353.png)

## 安装 Aria2

### 下载安装

1. 进入网址下载 [https://github.com/aria2/aria2/releases/tag/release-1.34.0](https://github.com/aria2/aria2/releases/tag/release-1.34.0)
	![](https://raw.githubusercontent.com/ripperhe/Resource/master/20180702/snip_20180702105317.png)
2. 将压缩包解压，将解压出来的文件放到 `C:\Program Files\aria2` 文件夹下面
	![](https://raw.githubusercontent.com/ripperhe/Resource/master/20180702/snip_20180702142705.png)
	
### 配置

配置文件是直接利用 `BaiduExporter` 开发者推荐的一个配置

1. 下载配置文件 [aria2.conf](https://pan.baidu.com/s/17Dme1SMNYJ3HQfftghO_bg)
2. 将配置文件放到 `C:\Program Files\aria2` 文件夹下面

### 关于启动

这里介绍两种启动方法

#### 1. 直接在终端输入命令启动

如果你是按照我上面的文件目录放的文件，直接利用在终端输入以下命令即可启动

```shell
C:\PROGRA~1\aria2\aria2c --conf-path=C:\PROGRA~1\aria2\aria2.conf
```

其中 `PROGRA~1` 代表的是 `Program Files` 文件夹，终端里面不方便使用空格，所以用此代替。如果你不是按照我的目录存放的文件，那么将 `C:\PROGRA~1\aria2` 替换成你自己的文件夹路径

![](https://raw.githubusercontent.com/ripperhe/Resource/master/20180702/snip_20180702112631.png)

#### 2. 制作一个 `.vbs` 快捷启动

在桌面新建一个 `.txt` 文件，写入一下代码

```vb
CreateObject("WScript.Shell").Run "C:\PROGRA~1\aria2\aria2c --conf-path=C:\PROGRA~1\aria2\aria2.conf",0
```

然后保存，将文件改名为 `Aria2Run.vbs` （注意后缀改为了 `.vbs` ），双击该文件即可启动 Aria2。不过为了方便，我已经制作好，并放到了网盘，可以及直接下载 [Aria2Run.vbs](https://pan.baidu.com/s/17Dme1SMNYJ3HQfftghO_bg)


## 配置 Aria2 Web Frontend

进入网址 [Aria2 Web Frontend](http://binux.github.io/yaaw/demo/#)，需要将 `JSON-RPC Path` 修改为 `http://localhost:6800/jsonrpc` 即可显示，将 `Auto Refresh` 修改为 `1s` 看起来更方便一些

![](https://raw.githubusercontent.com/ripperhe/Resource/master/20180702/snip_20180702141826.png)

## 使用方法

以上所有工具都安装配置好之后，则可按照以下操作下载百度网盘文件

1. 在终端输入命令打开 `Aria2`，或者利用点击 `.vbs` 文件打开
2. 将百度网盘需要下载的文件创建分享链接
3. 用 `Chrome` 浏览器打开分享链接（此时最好**退出网页中百度云盘账号登陆状态**，以免查封）
4. 点击页面的 `导出下载` 按钮，选中 `ARIA2 RPC`
	![](https://raw.githubusercontent.com/ripperhe/Resource/master/20180702/snip_20180702113955.png)
5. 打开 [Aria2 Web Frontend](http://binux.github.io/yaaw/demo/#) 网页查看下载进度
	![](https://raw.githubusercontent.com/ripperhe/Resource/master/20180702/snip_20180702114030.png)

## 参考

* [https://aria2.github.io/](https://aria2.github.io/)
* [https://blog.icehoney.me/posts/2015-01-31-Aria2-download](https://blog.icehoney.me/posts/2015-01-31-Aria2-download)
* [https://steemit.com/chrome/@free1x/chrome](https://steemit.com/chrome/@free1x/chrome)
* [http://saili.science/2017/03/31/bdy/](http://saili.science/2017/03/31/bdy/)

> * 本文永久更新链接：<https://ripperhe.com/2018/07/02/windows-aria2>
> * 作者：[ripperhe](https://github.com/ripperhe)