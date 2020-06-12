---
layout: post
title: 喜马拉雅 FM 已购付费音频下载
categories: Skill
description: 喜马拉雅 FM 音频下载
keywords: 喜马拉雅 FM, 音频, 下载
---

如题，本文主要讲解一下如何下载在喜马拉雅 FM 中**已购买**的付费音频。之前想分享自己购买的付费音频给朋友听，碍于喜马拉雅 FM 的音频不能直接导出，所以准备自己搞个下载的小软件。后来发现已经有大佬实现了，在此也算是借花献佛吧！重要的事情说三遍哈：

* **仅可下载已购买的付费音频**
* **仅可下载已购买的付费音频**
* **仅可下载已购买的付费音频**

当然，如果你是会员，会员免费听的节目也可以下载~

## 准备工作

本文写的很基础，旨在让更多的朋友能够直接看懂每一步，大佬勿喷。

下载音频有两个步骤，第一步是获取包含账号信息的 token，第二步是使用专门下载软件进行下载。由于下载软件是 Windows 应用，所以需要准备一台 **Windows 系统的电脑**，如果用的 Mac 电脑，可以使用虚拟机。

获取 token 需要使用 Chrome 浏览器进行获取（当然你也可以用别的方式），先进入以下链接下载安装，如已安装，直接跳过。

👉 Chrome 浏览器链接: <https://pan.baidu.com/s/1-3M3ByIEudpbhekH3cJU5g>  密码: `woqx`

接着，下载喜马拉雅 FM 下载器，直接把下面的这个文件夹下载下来，放到桌面即可。

👉 喜马拉雅 FM 下载器链接: <https://pan.baidu.com/s/17hVasZ0UM7pfcZ6qPYqF4w>  密码: `apj5`

## 获取 token

OK，现在文件下载好了，打开 Chrome 浏览器，进入 <https://www.ximalaya.com/> 网站，**然后先把你的账号登陆上**。

紧接着点击浏览器右边竖着的**三个点**，选中**更多工具**，再选中**开发者工具**，将其打开

![](https://raw.githubusercontent.com/ripperhe/oss/master/2019/0331/Jietu20190331-124640.png)

打开之后

1. 选中 Network 按钮
2. 接着在输入框中输入 `album?` 用于过滤掉那些多余的信息
3. 然后点击一下清除按钮，清除掉乱七八糟的网络请求
4. 最后，在网址栏输入一下  <https://www.ximalaya.com/> ，重新进入喜马拉雅网站

如下图所示

![](https://raw.githubusercontent.com/ripperhe/oss/master/2019/0331/Jietu20190331-125709.png)

重新进入网站之后，应该会获取到一个网络请求。如下所示

![](https://raw.githubusercontent.com/ripperhe/oss/master/2019/0331/Jietu20190331-125823.png)

**如果没有看到任何一个请求，就随便选一个专辑，点进去，然后播放任意一个音频，应该就会有请求出现在列表中了。**

选中一个请求之后，查看该请求的详细信息，选中 `Headers` - `Request Headers` - `Cookie`，这个 Cookie 里面的内容就是我们需要的东西

![](https://raw.githubusercontent.com/ripperhe/oss/master/2019/0331/Jietu20190331-130042.png)

将 Cookie 中 `1&_token=` 开始的这部分拷贝出来，到最近的一个分号 `;` 为止，这就是我们要的 token 了。**注意，从 `1` 开始复制，到 `;` 结束，不需要最后这个分号。**形如这样:

```
1&_token=24789987&IJFD87WY757C4NdV9FB4226EJD343EBE3A8D9B21B8ECA1908EF06FBF690BE15A02D806032C02CE58
```

## 开始下载

现在打开刚才喜马拉雅下载器文件夹，打开 `config.ini` 文件，如下所示

```
[Settings]
max_thread_num=4
download_try_times=5
[settings]
cookies=将你获取到的1&_token的内容替换到这里
history=https://www.ximalaya.com/renwen/15801963/
```

⚠️ 顺便提醒一下，下载不了常常是因为 `config.ini` 文件没写对，所以：

* **请务必使用从我分享的链接下载的 `config.ini` 文件**
* **请务必使用从我分享的链接下载的 `config.ini` 文件**
* **请务必使用从我分享的链接下载的 `config.ini` 文件**

现在直接将 token 替换到文件中 `cookie=` 后面，**不要留空格**，**不要有任何其他多余操作**，形如这样：

```
[Settings]
max_thread_num=4
download_try_times=5
[settings]
cookies=1&_token=24789987&IJFD87WY757C4NdV9FB4226EJD343EBE3A8D9B21B8ECA1908EF06FBF690BE15A02D806032C02CE58
history=https://www.ximalaya.com/renwen/15801963/
```

![](https://raw.githubusercontent.com/ripperhe/oss/master/2019/0331/Jietu20190331-130334.png)

`history` 后面对应的就是打开软件默认填写的专辑 URL 地址，可以不用管。现在将文件保存好，然后打开喜马拉雅 FM 下载工具 `喜马拉雅FM下载工具.exe`

将专辑 URL 填写到顶部的输入框，然后点击 Info 按钮解析，直接显示 + 号就证明可以下载了，然后选中所有，点击 Download 按钮开始下载

![](https://raw.githubusercontent.com/ripperhe/oss/master/2019/0331/Jietu20190331-130620.png)

下载完成之后，文件会放到和自动下载器的同一文件夹，如同所示

![](https://raw.githubusercontent.com/ripperhe/oss/master/2019/0331/Jietu20190331-130718.png)

祝贺，现在已经下载成功了~ 🎉

## 下载失败？`2020.06.02更新`

根据群友的反应，按照以上方法大部分的电脑上是可以正常下载的，但是有少部分人仍旧下载失败，具体体现为付费专辑中仅有免费的部分显示 `+` 号，其余依旧显示 `-` 号

现在说下解决方案，所有步骤跟前面一样，然后将 `config.ini` 文件的内容改为如下所示，两个需要填写 token 的地方都填上自己刚才获取的 token，**不要留空格**，**不要有任何其他多余操作**

```
[Settings]
max_thread_num=4
download_try_times=5
cookies=将你获取到的1&_token的内容替换到这里
history=https://www.ximalaya.com/renwen/15801963/
[settings]
max_thread_num=4
download_try_times=5
cookies=将你获取到的1&_token的内容替换到这里
history=https://www.ximalaya.com/renwen/15801963/
```

形如这样

```
[Settings]
max_thread_num=4
download_try_times=5
cookies=1&_token=24789987&IJFD87WY757C4NdV9FB4226EJD343EBE3A8D9B21B8ECA1908EF06FBF690BE15A02D806032C02CE58
history=https://www.ximalaya.com/renwen/15801963/
[settings]
max_thread_num=4
download_try_times=5
cookies=1&_token=24789987&IJFD87WY757C4NdV9FB4226EJD343EBE3A8D9B21B8ECA1908EF06FBF690BE15A02D806032C02CE58
history=https://www.ximalaya.com/renwen/15801963/
```

如果确认操作步骤和文章完全一样，没有漏掉细节，并且使用后面这个 config 文件还是无法下载，那就放弃吧，放弃吧 😂

## 最后

喜马拉雅默认的音频格式就是 `m4a` 文件，是可以直接播放的，不需要做什么操作。如果不能播放，只能证明你的播放器不支持这种格式，可以下载一个格式工厂将音频转换成 `mp3` 格式进行播放。

截止目前 2019 年 3 月 31 日，该下载方法仍旧是可以用的，如有需要，尽快下载。

**最后，还请大家低调使用，不要私自兜售付费音频。**

如有任何问题，可以进 QQ 群讨论，群号：**763915098**。

<img src="https://gitee.com/ripperhe/oss/raw/master/2020/0602/fm2_qq_group.JPG" alt="QQ群二维码" width="400" />
