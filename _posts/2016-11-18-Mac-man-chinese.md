---
layout: post
title:  Mac 安装man命令中文文档
categories: Mac
description: 安装man命令中文文档
keywords: Mac, man
---

|----+----|
|本机环境|版本|
|:---|:---|
|macOS|Sierra 10.12|
|----+----|

备注：系统版本不同，可能会有一些不同的问题。

### man是什么？
man，是类unix系统最重要的手册工具，mac预装了man，所以我们可以通过man查询各种命令的使用方法。不过在使用的时候，全都是中文，如果英文不太好，阅读起来就比较困难。

### 安装中文文档

#### 下载安装包
利用wget下载安装包

```
$ wget https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/manpages-zh/manpages-zh-1.5.1.tar.gz --no-check-certificate
```
没有安装wget的话，可以查看我这篇文章[安装wget](http://www.jianshu.com/p/f6b290710262)，也可以直接到 [google code](https://code.google.com/archive/p/manpages-zh/downloads) ，选中**manpages-zh-1.5.1.tar.gz**进行下载。

#### 解压
```
$ tar zxvf manpages-zh-1.5.1.tar.gz
```
直接双击解压也可以

#### 安装
先进入解压出来的**manpages-zh-1.5.1**文件夹

```
$ cd manpages-zh-1.5.1
```
然后执行

```
$ ./configure --prefix=/usr/local/zhman --disable-zhtw
```
其中，`--disable-zhtw`代表不安装繁体中文，如果要安装繁体中文，还需要配置一些东西

```
$ make && make install
```

#### 配置别名
现在已经将中文文档安装完成，想调用中文文档的话，我们可以对`/etc/man.conf`文件进行修改，将其内容中所有`/usr/local/share/man`全部替换为`/usr/local/share/man/zh`即可，这样利于man命令的时候，则会显示中文，按`q`退出。不过这样的方式多少有些问题，利用设置别名的方法可以更好地处理问题。

进入到主用户文件夹

```
$ cd ~
```
显示所有文件

```
$ ls -a
```
文件列表中有`.bashrc`文件，最好不要用文本编辑，我个人比较喜欢使用Atmo，如果没有，可以从github上下载（直接在终端打开这个文件进行编辑也可以）。

```
$ open -a atom .bashrc
```
在文件最后添加以下语句，然后保存、退出
> alias cman='man -M /usr/local/zhman/share/man/zh_CN'

最后重载该文件

```
$ source .bashrc
```
这样别名配置就生效了，可以通过以下命令查看所有的别名配置

```
$ alias
```
现在可以调用`cman + 命令名`查询命令，不过现在可能会出现中文乱码问题

#### 解决中文乱码
主要原因是mac的groff版本比较老，可以利用以下命令查看版本

```
$ groff -v
```
先到网站 [groff.git](http://git.savannah.gnu.org/cgit/groff.git) 下载groff新版本，一般1.22版本即可，选中**groff-1.22.tar.gz**进行下载。下载完成之后，解压。然后

```
$ cd groff-1.22
$ ./configure
$ sudo make
$ sudo make install
```
可能会有一些报错，不过不太影响，这个时候进入到`/etc/man.conf`文件。同样，这里也可以利用Atom打开

```
$ open -a atom /etc/man.conf
```
在文件末尾加上如下语句，然后保存、退出
>NROFF preconv -e UTF8 | /usr/local/bin/nroff -Tutf8 -mandoc -c

重启终端，再尝试前面定义的`cman`命令，基本可以正常显示了。在显示上还是有些小问题，不过都还能接受。以下语句即可查询`ls`命令的用法

```
$ cman ls
```
当然，能看英文文档是最好的，不过在终端看文档，我始终觉得有些不方便，可以利用以下命令让文档输出在preview(预览)中进行查看

```
$ man -t ls | open -a Preview -f
```
以上命令，将ls换成想要查询的命令即可（`cman`暂时没找到方法中文输出到Preview，如果找到方法还望大神多多指教）。

#### 重启终端命令失效的问题
重启之后`.bashrc`文件没有自动加载，会导致自定义的命令alias失效，这里就需要加载手动将其加载一下。进入到用户文件夹下，找到`.bash_profile`文件

```
$ cd ~ && ls -a
```
如果没有`.bash_profile`文件,那需要手动创建一个

```
$ touch .bash_profile
```
然后利用Atom打开`.bash_profile`文件

```
$ open -a atom .bash_profile
```
在文件最后加上以下命令加载`.bashrc`

> source ~/.bashrc

现在再重启终端应该就可以了。如果这样还不行，那应该是装了`oh-my-zsh`，zsh覆盖了一些系统的shell变量，导致打开终端没有自动调用`.bash_profile`文件,所以我们再到zsh中调用一下`.bash_profile`即可。

```
$ open -a atom ~/.zshrc
```
在文件末尾加上以下，命令即可

> source ~/.bash_profile

然后，打完收工。