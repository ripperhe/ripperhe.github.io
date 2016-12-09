---
layout: wiki
title: Mac Terminal
categorys: Mac
description: Mac 终端命令
keywords: Mac, command, terminal
---

### 关于控制台的术语

| 名词 | 词义 |
| :--- | :--- |
| Console | 控制台，包括当前命令行以及前面输出的命令。 |
| Command Line | 控制台中你输入命令的实际行。 |  
| Prompt | 提示，这是命令行的开始。它通常提供一些上下文信息，例如你是谁，你在哪里和其他一些有用的信息。它通常显示为一个 美元符号`$`，其后是您将键入命令的地方。 |
| Terminal | 终端，这是控制台的实际接口，让我们可以键入并执行命令。 |

### 终端快捷键

| 快捷键 | 功能 |
| :---  | :--- |
| ctrl + a | 将光标移动到行首 |
| ctrl + e | 将光标移动到行尾 |
| option + 左/右 | 将光标以单词为单位左右移动 |
| ctrl + l | 清除当前屏幕 |
| ctrl + u | 清除当前行 |
| ctrl + k | 清除从光标位置到行尾 |
| ctrl + r | 搜索使用过的命令并填入当前行 |
| ctrl + z | 将当前命令暂停在后台，用fg恢复 |
| ctrl + c | 终止当前命令 |


* 备注：其中`option + 左/右`在mac原生终端中可以使用，iTerm中不行

### 常用命令

#### 查询命令

```
$ man 命令名
```
* 输入命令名字，可查询其具体使用方法
* 举例

```
$ man -t cal | open -a Preview -f
# 查询cal命令，并用Preview打开
```

####  显示文件列表

```
$ ls 参数 文件目录
```

* 参数：`-a` 显示所有文件		
* 文件夹：不输入则表示当前文件夹

#### 切换文件目录

```
$ cd 文件目录
```

* 文件目录：

	* `/` 到根目录 
	* `~` 到用户目录 
	* `..` 到上一个目录

#### 显示当前目录

```
$ pwd
```

#### 新建文件夹

```
$ mkdir 目录名
```

#### 新建文件

```
$ touch 文件名
```

#### 拷贝

```
$ cp 参数 源文件 目标文件
```

* 参数：`-R` 拷贝文件夹的时候需要用
* 源文件：如果是当前文件夹的文件，直接写文件名，否则要带上整个文件路径
* 目标文件：如果写的是一个文件夹路径，则会将文件拷贝到那个文件路径；如果最后加了文件名，将会重命名源文件

#### 删除

```
$ rm 参数 文件
```

* 参数：`-rf` 强制删除

#### 移动

```
$ mv 源文件 目标文件
```

#### 查找

```
$ find 文件目录 -name "匹配符"
```

* 举例

```
$ find . -name "*ripper*"
# 找出当前目录及其子目录中所有名字包含"ripper"的文件

$ find . -name "*.md"
# 找出当前目录及其子目录中所有名字后缀为".md"的文件
```





<!--
```
chmod 参数 权限 文件

chown 参数 用户：组 文件

nano 文件名
Ctrl + O   Ctrl + X

sh 脚本文件名

cat 文件名

pg 文件名

more 文件名

ln -s 文件1 文件2

find . -name "*.c" -print

file 文件名

head -20 文件名

tail -15 文件名

diff 文件1 文件2

passwd

make 


kill

date

cal

telnet

echo

printf

```
-->

### Other

#### 开启显示隐藏文件

```
$ defaults write com.apple.finder AppleShowAllFiles -bool true
```

#### 关闭显示隐藏文件

```
$ defaults write com.apple.finder AppleShowAllFiles -bool false
```






