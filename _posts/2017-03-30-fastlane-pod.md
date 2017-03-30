---
layout: post
title:  一行命令发布 Pod 框架
categories: iOS
description: 快速发布 Pod 框架
keywords: iOS, fastlane, CocoaPods
---

## 前言

目前比较流行的组件化开发，针对多个 app 要用同一套代码，将其做成 pod 仓库是比较好的解决方案。代码只有一份放在组件仓库，需要集成的 app 只需要将其 pod 到工程内部即可。

如果很多组件都需要做成 pod 库，每一步都手动去做，显得繁琐而且容易出错。本文主要讲一下，怎么自动化去实现这些事情。不过，在此之前，先概述一下，发布框架具体需要做哪些事情。如果对发 pod 的流程比较熟悉，直接跳过看 [自动化实现](#fastlane) 部分。

先上 **GitHub 仓库地址**：

* [fastlane-files → 快车道文件](https://github.com/ripperhe/fastlane-files)
* [pod-template → ruby 脚本及模板文件](https://github.com/ripperhe/pod-template)

## 发布框架到官方库

![](https://raw.githubusercontent.com/ripperhe/Resource/master/20170330/public_pod.png)

> 注：图中标号并非和发布流程标号对应，图中表述的是从框架发布到使用的**大致**流程。

想要发布框架到 CocoaPods 官方库，需要有以下流程：

> 1. [安装 CocoaPods](#a-安装 CocoaPods)
> 2. [注册 CocoaPods 账号](#a-注册 CocoaPods 账号)
> 3. [创建 spec 文件](#a-创建 spec 文件)
> 4. [提交代码并为框架打 tag](#a-提交代码并为框架打 tag)
> 5. [验证 spec 文件](#a-验证 spec 文件)
> 6. [通过 pod trunk push 推送 spec 文件](#a-推送 spec 文件)

### <a name="a-安装 CocoaPods">安装 CocoaPods</a>
	
网上有很多教程，这里不再赘述。需要注意的是，`RubyGems` 的 [淘宝镜像](https://ruby.taobao.org/) 已经不再维护，已迁移至 [Ruby China](https://gems.ruby-china.org/)，安装 CocoaPods 前记得替换 `gem sources`。
	
### <a name="a-注册 CocoaPods 账号">注册 CocoaPods 账号</a>

首先需要注册一个 CocoaPods 账号，用于发布 pod 仓库。

```ruby
 $ pod trunk register EMAIL [NAME]
```

`trunk` 命令用于创建一个账号或是一个 seession。

* 如果你还没有账号，则需要将邮箱和名字一起填入，创建账号的同时会自动创建 session
* 如果已经有了账号，但是没有在当前这台电脑上发布过 pod 仓库，这个时候就需要创建一个 session，仅需要填邮箱即可，除非你想改名字，否则不用填写名字。

你可以对你的 session 进行描述，这样在利用 `pod trunk me` 命令查看所有 session 信息的时候，可以很清晰地看到在哪些电脑上创建过 session。

```ruby
$ pod trunk register orta@cocoapods.org 'Orta Therox' --description='macbook air'
```

命令成功之后，你的邮箱应该会收到一份邮件。查看邮件信息，访问对应网址即可验证你的账号。当然，这个事情做一次就好了，以后基本上不用再操作。

### <a name="a-创建 spec 文件">创建 spec 文件</a>

spec 文件即一个 pod 仓库的描述信息。所有通过 `pod search` 搜索出来的框架，都是根据这个文件中的描述信息进行匹配的。以下为部分描述信息：

* 仓库名字
* 作者
* 证书
* 版本
* 描述
* 代码源路径
* 使用哪些文件
* 依赖哪些其他框架
* ...

一个简单的 spec 文件如下：

```ruby
Pod::Spec.new do |s|
  s.name             = 'ZYTemplateName'
  s.version          = '0.1.0'
  s.summary          = 'A short description of ZYTemplateName.'
  s.homepage         = 'https://github.com/ripperhe/ZYTemplateName'
  s.license          = { :type => 'MIT', :file => 'LICENSE' }
  s.author           = { 'ripperhe' => 'ripperhe@qq.com' }
  s.source           = { :git => 'https://github.com/ripperhe/ZYTemplateName.git', :tag => s.version.to_s }
  s.ios.deployment_target = '8.0'
  s.source_files = 'ZYTemplateName/Classes/**/*'
end
```

先进入需要发布的框架的根目录，执行

```ruby
$ pod spec create [NAME]
```

这里的 `NAME` 即为框架名字，不需要手动加 `.podspec` 后缀。执行完成之后，会生成 `ZYTemplateName.podspec` 文件（这里以 `ZYTemplateName` 为例），进入该文件，将对应信息改为你具体的信息即可。

如果你发现你的 spec 文件默认填写的用户名和邮箱不是你刚才利用 trunk 命令注册账号时候填写的不用惊讶，因为默认取的是 git 的用户名和邮箱，所以想修改的话，去修改 `~/.gitconfig` 文件中的用户名和邮箱。以下为相关源码：

```ruby
def default_data_for_template(name)
	data = {}
	data[:name]          = name
	data[:version]       = '0.0.1'
	data[:summary]       = "A short description of #{name}."
	data[:homepage]      = "http://EXAMPLE/#{name}"
	data[:author_name]   = `git config --get user.name`.strip
	data[:author_email]  = `git config --get user.email`.strip
	data[:source_url]    = "http://EXAMPLE/#{name}.git"
	data[:ref_type]      = ':tag'
	data[:ref]           = '#{s.version}'
	data
end
```
spec 文件的这些描述，基本上看命名是可以看明白的。实在不明白可以查看 [官方文档](https://guides.cocoapods.org/syntax/podspec.html#specification)。需要注意的是，`s.version` 是你将要发布的 pod 仓库的版本，`s.source` 是你的仓库源地址和对应的 `tag` 信息。

`version` 和 `tag` 可以是不同的，但是为了方便管理和维护，我们一般将其设置为一样的，所以 `tag` 可以 直接使用 `s.version.to_s`。既然这里用到了 `tag`，那么接下来，就需要为仓库代码打上 `tag`。

### <a name="a-提交代码并为框架打 tag">提交代码并为框架打 tag</a>

首先，需要将本仓库的所有修改提交到远程仓库。

```ruby
$ git add .
$ git ci -m 'release pod'
$ git push
```

上一步在仓库的根目录创建了 spec 文件，随着代码的提交，spec 文件也会提交到自己的远程代码仓库。这个 spec 文件和需要发布的框架放在一起并不是必须，只是这样可以方便我们以后维护这个框架。

发布 pod 仓库，需要和自己框架的远程仓库代码版本对应，所以这里需要为当前代码打上 tag，这个 tag 是和前面的 spec 文件中填写的 tag 对应的。框架发布成功之后，CocoaPods 会根据 tag 信息去获取相应代码。

```ruby
$ git push origin master
$ git tag '0.1.0' 
$ git push --tags 
```

### <a name="a-验证 spec 文件">验证 spec 文件</a>

用于验证 spec 文件是否正确，可以及早发现问题。同样，也是在需要发布的框架的根目录，执行

```ruby
$ pod spec lint ZYTemplateName.podspec
```

这里需要加上 `.podspec` 后缀来验证这个文件。如果有报错，仔细查看报错信息，一般可以定位问题，可以加上 `--verbose` 查看详细的验证过程，方便定位问题。

```ruby
$ pod spec lint ZYTemplateName.podspec --verbose
```

验证 spec 文件也可以用另外一个命令

```
$ pod lib lint ZYTemplateName.podspec
```

`pod spec lint` 和 `pod lib lint` 最主要的区别是，前者会根据 spec 文件 tag 信息去验证远程仓库代码是否存在，后者不会。简单理解就是，`pod spec lint` 联网检查，`pod lib lint` 不联网检查。

验证的时候，可能会报错，或是报警告。报错的话，必须解决，详细看具体信息，一般能找到问题，如果不能，自行 [google](https://www.google.com) 😁。警告的话，可以忽略，不过能解决的话，最好解决。

```
$ pod lib lint ZYTemplateName.podspec --allow-warnings
```

### <a name="a-推送 spec 文件">推送 spec 文件</a>

验证文件通过之后，需要将文件推送到 CocoaPods 描述文件仓库，在框架根目录，执行以下命令：

```ruby
$ pod trunk push ZYTemplateName.podspec
```

这个命令，可以不在后面加描述文件名称，如果不加的话，会在终端当前工作目录下查找 `.podspec` 文件，执行 `trunk push` 命令。

这一步稍微会久一点，需要等待一会儿，因为 CocoaPods 需要先将 spec 文件上传到 `CocoaPods/Specs` 仓库，再 `git pull` 到本地 `CocoaPods/Specs` 仓库。等待显示成功之后，可以验证以下自己的框架是否真的发布成功了。搜索一下：

```ruby
$ pod search [NAME]
```

如果搜索不到，可能是仓库的索引库出现问题，将其删除，重新生成索引。索引缓存地址：

```
~/Library/Caches/CocoaPods/search_index.json
```

如果这样还不行，有可能是官方的 spec 仓库 出现问题。官方 spec 仓库路径：

```
~/.cocoapods/repos/master
```

将其删除，再重新克隆官方的 spec 仓库：

```
$ pod setup
```

这个时候再试试 `pod search` 应该就有了。

## 发布框架到私有库

![](https://raw.githubusercontent.com/ripperhe/Resource/master/20170330/private_pod.png)

> 注：图中标号并非和发布流程标号对应，图中表述的是从框架发布到使用的**大致**流程。

对于公司内部使用的组件模块，一般是不公开的，所以就需要用到私有 sepc 仓库。

将代码发布到私有仓库和 CocoaPods 官方库最大的区别就是，不需要利用 trunk 命令创建 session。既然整个私有库都是你的，所以也没有必要再有账号机制了，自己想怎样就怎样。

少做了一件事情，但也有额外的事情需要做。前面 spec 文件都是 CocoaPods 帮你管理的，所以做私有 pod，就需要创建私有 spec 仓库。

整体流程如下：

> 1. [安装 CocoaPods](#b-安装 CocoaPods) 
> 2. [创建远程私有 spec 仓库](#b-创建远程私有 spec 仓库)
> 3. [克隆远程私有 spec 仓库到本地](#b-克隆远程私有 spec 仓库到本地)
> 4. [创建 spec 文件](#b-创建 spec 文件)
> 5. [提交代码并为框架打 tag](#b-提交代码并为框架打 tag)
> 6. [验证 spec 文件](#b-验证 spec 文件)
> 7. [通过 pod repo push 推送 spec 文件](#b-推送 spec 文件)

很多流程是相同的，相同的流程一笔带过。

### <a name="b-安装 CocoaPods">安装 CocoaPods</a>

[同上。👆](#a-安装 CocoaPods)

### <a name="b-创建远程私有 spec 仓库">创建远程私有 spec 仓库</a>

所谓的远程私有 spec 仓库，其实也就是一个普通远程代码仓库，到一个 git 仓库托管网站创建一个空仓库即可。[GitHub](https://github.com/) 创建私有仓库是需要付费的，所以可以在 [Bitbucket](https://bitbucket.org) 和 [码市](https://coding.net) 创建免费的私有仓库。

这里以码市为例。创建账号并点击创建仓库

![](https://raw.githubusercontent.com/ripperhe/Resource/master/20170330/create_step1.png)

选中私有属性，点击创建即可

![](https://raw.githubusercontent.com/ripperhe/Resource/master/20170330/create_step2.png)


### <a name="b-克隆远程私有 spec 仓库到本地">克隆远程私有 spec 仓库到本地</a>

首先，复制远程仓库地址。

![](https://raw.githubusercontent.com/ripperhe/Resource/master/20170330/create_step3.png)

利用以下命令，克隆远程仓库，并为自己的私有仓库命名

```ruby
$ pod repo add NAME URL
```

克隆完成之后，你会发现，在 `~/.cocoapods/repos/` 文件夹下，除了 CocoaPods 官方 spec 仓库 `master` 之外，还多了自己的仓库。

### <a name="b-创建 spec 文件">创建 spec 文件</a>

[同上。👆](#a-创建 spec 文件)****

### <a name="b-提交代码并为框架打 tag">提交代码并为框架打 tag</a>

[同上。👆](#a-提交代码并为框架打 tag)

### <a name="b-验证 spec 文件">验证 spec 文件</a>

[同上。👆](#a-验证 spec 文件)

### <a name="b-推送 spec 文件"> 推送 spec 文件</a>

这一步同发布到官方库不同，利用以下命令：

```ruby
$ pod repo push REPO [NAME.podspec]
```

以我自己的仓库为例：

```ruby
$ pod repo push ZYSpec ZYTemplateName.podspec
```

这个命令实质上是先将该 spec 文件提交到本地的私有仓库，然后再 push 到远程仓库。而发布到官方库时，是通过 `pod trunk push` 通过请求，将 spec 文件加入到了官方库远程仓库，再从远程仓库拉到了本地，不过最终的效果是相同的。当命令显示成功，框架就发布完毕了。

因为是推送到了自己的仓库，所以在使用的时候需要注意。在 podfile 中需要添加私有 spec 仓库地址：


```ruby
source 'https://github.com/CocoaPods/Specs.git'   # 官方库地址
source 'https://git.coding.net/ripper/Specs.git'  # 私有仓库地址

platform :ios, '8.0'

target 'PodDemo' do
  # 官方库中的框架
  pod 'MJExtension', '~> 3.0.13'
  # 私有库中的框架
  pod 'ZYLibXXX'
end
```

如果需要官方库中的框架，也需要将官方库地址写上，如果 source 都不写，默认是从官方库中查找框架信息。

可能这里会存在一个问题，同一个 spec 仓库不能够有重名的框架，但是多个 spec 仓库之间，是可以有重名的框架的。所以，可能会存在名字相同的问题，一般采取自己特有的前缀的办法来避免这个问题。

## <a name="fastlane">利用 fastlane 实现自动化</a>

从前面可以看到，发布 pod 的流程虽然不算复杂，但是每次一步步地做这些事情，非常的没有必要，所以这里考虑用自动化来实现。自动化实现，就需要用到 fastlane。

### fastlane 简介

Fastlane 是用 Ruby 编写的一套自动化工具框架，最开始主要针对 iOS 开发，后来因为社区呼声太高，于是也支持了 Android。Fastlane 内置了许多任务脚本，使用者可以自行组合，涵盖了打包、签名、测试、发布等移动开发常用功能。

使用 Fastlane 首先理解两个概念，action 和 lane。一个 action 其实就是一个 ruby 脚本，每个 action 的粒度大小不一，现在 Fastlane 官方库中有 100 多个 action，其中有很多 action 都是 github 社区成员贡献的。一个 lane 就是一条自动化命令流程，lane 是由一个或多个 action 组成。

### 安装 fastlane

Fastlane 的安装仅需一条命令即可，[官网](https://docs.fastlane.tools/getting-started/ios/setup/) 有更详细的解释。

```ruby
$ sudo gem install fastlane
```

### 书写 Fastfile

如果使用 Fastlane 进行提包等操作，就在工程根目录下执行以下命令：

```ruby
fastlane init
```

这样就会询问你的 Apple ID 以及其他的一些配置，并且生成一大堆东西，而我们这里只是想要发布 pod，与 Apple ID 这些没用关系，所以我们手动创建 Fastfile。同样，在工程根目录：

```ruby
$ mkdir fastlane
$ touch fastlane/Fastfile
```

我们可以在 Fastfile 开始书写我们的 lane，不过，写之前，我们需要分析一下哪些步骤需要自动化。就发布官方库而言：

| 步骤 | 是否需要自动 |
|:--:|:--:|
| 安装 CocoaPods | ❌ |
| 注册 CocoaPods 账号 | ❌ |
| 创建（更新） spec 文件 | ❌ - ✅ |
| 提交代码并为框架打 tag | ✅ |
| 验证 spec 文件 | ✅ |
| 推送 spec 文件 | ✅ |

首先，安装 CocoaPods 肯定是手动安装，第二次也不需要再安装。利用 trunk 注册账号也是同样的，如果不换电脑仅需调用一次 pod trunk 命令。创建 spec 文件，第一的时候肯定是需要手动创建的，需要填写很多自己的信息。不过在今后框架升级，如果 sepc 文件中不需要修改多余的信息，仅仅只需要修改 s.version 的话，可以通过自动化实现。提交代码并打 tag、验证 spec 文件以及推送文件都可以利用自动化实现。

那我们再看一下发布私有库：

| 步骤 | 是否需要自动 |
|:--:|:--:|
| 安装 CocoaPods | ❌ |
| 创建远程私有 spec 仓库 | ❌ |
| 克隆远程私有 spec 仓库到本地 | ❌ |
| 创建（更新） spec 文件 | ❌ - ✅ |
| 提交代码并为框架打 tag | ✅ |
| 验证 spec 文件 | ✅ |
| 推送 spec 文件 | ✅ |

创建远程仓库和克隆远程仓库到本地一般不需要自动实现，毕竟如果是公司使用，公司的所有框架放到一个私有 spec 仓库就好了，只需要创建一次、克隆一次，不需要每次都创建私有库，所以不需要自动化，其他基本和上面一致。

接下来，在 Fastlane 中写我们 lane，兼容这两种情况

```ruby
lane :release_pod do |options|
  target_repo    = options[:repo]
  target_project = options[:project]
  target_version = options[:version]
  spec_path = "#{target_project}.podspec"
  
  # git pull
  git_pull 
  # 确认是 master 分支
  ensure_git_branch
  # 修改 spec 为即将发布的版本
  version_bump_podspec(path: spec_path, version_number: target_version)
  # 提交代码到远程仓库
  git_add(path: '.')
  git_commit(path: '.', message: 'release')
  push_to_git_remote
  # 检查对于 tag 是否已经存在
  if git_tag_exists(tag: target_version)
      # 删除对应 tag
      remove_git_tag(tag: target_version)
  end
  # 添加 tag
  add_git_tag(tag: target_version)
  # 提交 tag
  push_git_tags
  # 验证 spec 文件
  pod_lib_lint(allow_warnings: true)
  # 检查是否传了 repo 参数
  if target_repo
  	# pod repo push 'target_repo' 'spec_path'
    pod_push(path: spec_path, repo: target_repo, allow_warnings: true)
  else
  	# pod trunk push 'spec_path'
    pod_push(path: spec_path, allow_warnings: true)
  end
end
```

代码并不多，从头开始就是我们 lane 的名字为 release_pod，接下来就是三个参数。

* repo：指定的 specs 仓库，如果发布私有框架，就需要传自己的 spec 仓库名，发布到官方库不需要传。
* project：想要发布的框架名字
* version：想要发布的框架版本

为了安全起见，先拉取代码，并确认是 master 分支。之后，`version_bump_podspec` 可以自动修改 spec 文件的版本信息为我们想要发布的版本。紧接着提交代码，之所以判断 tag 是否存在，是因为我们可能在上一次走到了验证 spec 文件，但是没通过，所以我们修改了之后重新发布，但是刚才已经打了相应 tag，这一次就打不上相同的 tag，所以先判断是否存在，存在将错误的 tag 删除。当然，也可能会因为自己输入错误了想要发布的 verison 而导致误删 tag，所以可以自行斟酌这一步是否需要。

后面就是验证 spec 文件，前面说过 `pod spec lint` 和 `pod lib lint` 主要区别是是否联网检查对于 tag 的代码是否存在，所以用两个检查都可以，但是由于 Fastlane 官方工具库中只有 `pod_lib_lint` 这个 action，所以这里也不折腾了，直接使用 `pod_lib_lint`。

最后就是推送 spec 文件。我们这里的设定是，如果指定仓库，那就发布到自己的私有仓库，不指定仓库，就发布到官方库。你也许会看到，我们这里是同一个 action，都是 `pod_push` 只是参数不同，其实这是因为这个 action 内部做了判断。查看这个 Fastlane 的官方 action 源码即可看到内部的判断，这里截取部分源码如下：

```ruby
if params[:repo]
  repo = params[:repo]
  command = "pod repo push #{repo}"
else
  command = 'pod trunk push'
end

if params[:path]
  command << " '#{params[:path]}'"
end
```

我们这条 lane 中，`git_pull`、`ensure_git_branch`...每一句都是一个 action。这里面除了 `remove_git_tag` 之外，都是官方自带的 action，由于找遍了官方的 [actions](https://docs.fastlane.tools/actions/) 并没有找到删除 git tag 的，所以自定义了一个 action。

自定义 action，在工程根目录执行

```ruby
$ fastlane new_action --name remove_git_tag
```

之后 fastlane/actions 文件下，就会生成对应的 remove_git_tag.rb 文件，我们修改该文件为我们想要的样子就好。这里代码不全贴了，就贴上核心代码，如果感兴趣，可以在 github 仓库中下载查看。

```ruby
command = []

target_tag = params[:tag]
remove_local = params[:remove_local]
remove_remote = params[:remove_remote]

command << "git tag -d #{target_tag}" if remove_local
command << "git push origin :#{target_tag}" if remove_remote

if command.empty?
  UI.message('👉 If you really want to delete a tag, you should to set up remove_local and remove_remote at least one for true!')
else
  result = command.join(' & ')
  Actions.sh(result)
  UI.message('Remove git tag Successfully! 🎉')
end
```

### 执行 Fastlane

说了这么多，终于到了使用我们的 fastlane 的环节。对于我们发布官方库而言，假设我们 CocoaPods 安装，注册账号，创建 spec 文件都已经准备完毕，我们直接到根目录执行

```ruby
fastlane release_pod project:'框架名' version:'版本'
```

接下来只需要静静等待一步步自动执行。

发布私有库也同样简单，假设私有库已经创建好，克隆到本地并命名为 ZYSpec，那么，到根目录执行

```ruby
fastlane release_pod repo:ZYSpec project:'框架名' version:'版本'
```

![](https://raw.githubusercontent.com/ripperhe/Resource/master/20170330/success_release.png)

以后我们升级框架，假如没有依赖新的三方库之类的，我们同样直接执行一行命令即可。麻烦一点的情况，也不过就是手动修改一下 spec 文件，然后再执行命令即可。

### 复用 Fastfile

上面已经写好 Fastfile，但是每次发布新的 pod，我们都需要将 fastlane 整个文件夹拷贝到新的工程根目录中去，这显得非常的麻烦。另一个问题是，如果某一天我们觉得我们这个发布 pod 的流程不是很完善，我们也需要到每次仓库去修改 Fastfile，不方便统一管理。

不过 Fastlane 有一个 import 机制。也就是说，一个 Fastfile 可以导入另一个 Fastfile，并且还可以重写 lane。有两个命令用于 import：

> `import`

从本地路径导入 Fastfile

```ruby
import "../GeneralFastfile"

override_lane :from_general do
  # ...
end
```

> `import_from_git`

从一个 git 仓库导入 Fastfile，并且这个命令会导入所有自定义的 action

```ruby
import_from_git(url: 'https://github.com/fastlane/fastlane/tree/master/fastlane')
# or
import_from_git(url: 'git@github.com:MyAwesomeRepo/MyAwesomeFastlaneStandardSetup.git',
               path: 'fastlane/Fastfile')

lane :new_main_lane do
  # ...
end
```

对于着两种方式，我这里选择后者，虽然前者在本地执行比较快，但是我换了电脑就不行了。所以，我把当前写好的 Fastfile 放到了 GitHub 上，仓库地址 → [fastlane-files](https://github.com/ripperhe/fastlane-files)。

这样，我在每次用的时候，仅需在新的工程根目录创建 fastlane 文件夹，并在 fastlane 文件夹创建 Fastfile 文件，并在文件中写入

```ruby
import_from_git(url: 'https://github.com/ripperhe/fastlane-files', branch: 'master')
```

这里可以不指定 branch，默认就是 master 分支。这样，我们就可以使用同一份 Fastfile，便于维护。

## 利用 pod 模板仓库

如果做成 pod 的仓库比较多，每次都有去创建 spec 文件等，也显得比较麻烦，但是 CocoaPods 为我们提供了一个命令，直接创建一个仓库。

```ruby
$ pod lib create ZYLib
```

执行该命令之后会问几个问题，最终生成文件目录

```ruby
$ tree ZYLib -L 2

ZYLib
├── Example
│   ├── Podfile
│   ├── Podfile.lock
│   ├── Pods
│   ├── Tests
│   ├── ZYLib
│   ├── ZYLib.xcodeproj
│   └── ZYLib.xcworkspace
├── LICENSE
├── README.md
├── ZYLib
│   ├── Assets
│   └── Classes
├── ZYLib.podspec
└── _Pods.xcodeproj -> Example/Pods/Pods.xcodeproj

10 directories, 5 files
```

包含了 LICENSE、README.md、.podspec 等文件，并且创建好 git 管理，自己仅需将仓库与自己的远程仓库关联即可，非常方便，推荐使用。看更多信息，可以查看 [官网的解释](https://guides.cocoapods.org/making/using-pod-lib-create.html)。

其实 CocoaPods 是先克隆 [pod-template](https://github.com/CocoaPods/pod-template) 仓库到本地，再根据参数进行一些配置。可以将 `pod-template` fork 到自己账户下，并进行一些修改，每次使用 `pod lib create` 命令时拼上` --template-url=URL`，其中 URL 填写自己的模板地址。

## 利用 ruby 脚本创建模板

虽然有 `pod lib create` 这么好用的命令，但是其实我经常会写一些东西，在开始写的时候并没有想做成 pod，又或者不想要 `pod lib create` 创建出来的这么多文件，只想要其中一些。虽然可以 fork 一份过来修改，但觉得稍显麻烦。

所以这里准备自己写一个小脚本，创建几项必要的文件。以下为利用脚本创建的文件：

| 文件 | 说明 |
| :--: | :--: |
| README.md |写一个模板，以后填空就好 |
| LICENSE | 一般都是 MIT 证书 |
| fastlane/Fastfile | 虽然这个 Fastfile 只有一句代码，但是我还是记不住 |
| xxx.podspec | spec 文件必填的就那么几项，写个模板可以把大部分都填好，不用改了 |

到工程根目录下执行脚本：

```ruby
$ ruby ~/Desktop/pod-template/prepare_release.rb

[20:56:35]: Please input the lib name:👇
```

执行脚本，这里做了一个逻辑是询问 lib 名字，如果直接回车，就默认为当前文件夹名字。假设创建一个空文件夹 DemoLib，则结果如下：

```ruby
$ tree DemoLib

DemoLib
├── DemoLib.podspec
├── LICENSE
├── README.md
└── fastlane
    └── Fastfile

1 directory, 4 files
```

也存在一种情况，有的文件已经存在了，为了不覆盖当前存在的文件，所以前缀加一个 `template.`，结果如下:

```ruby
$ tree DemoLib

DemoLib
├── DemoLib.podspec
├── LICENSE
├── README.md
├── fastlane
│   ├── Fastfile
│   └── template.Fastfile
├── template.DemoLib.podspec
├── template.LICENSE
└── template.README.md
```

因为需要在工程目录执行脚本，但是脚本一般是放在其他文件夹下，所以每次执行的时候也不是很方便。所以这里建议设置一个命令别名，我是用的 [Oh My Zsh](http://ohmyz.sh/)，所以我直接在 `~/.zshrc` 文件末尾加上别名设置

```ruby
alias pre='ruby ~/Desktop/pod-template/prepare_release.rb'
```

这里等于符号两边不要留空格，之后，我要发布某一个组件的时候，只需要到工程根目录输入：

```ruby
$ pre
[21:06:38]: Please input the lib name:👇
```

![](https://raw.githubusercontent.com/ripperhe/Resource/master/20170330/pre.png)

现在终于不会忘记了...（当然，要是移动了模板文件的路径又得改... 😂 ）如果你也想使用这个脚本，可以 fork [这个仓库](https://github.com/ripperhe/pod-template) 到你的 GitHub 账号下，修改模板文件即可。以下为该仓库的文件：

```ruby
$ tree pod-template

├── LICENSE
├── README.md
├── prepare_release.rb
└── template-files
    ├── %ZYTemplateName%.podspec
    ├── Fastfile
    ├── LICENSE
    └── README.md
```

其中 `template-files` 文件夹下面的均为模板文件，不要去删除文件，脚本内部没有做什么容错处理。所以想要写框架名字的地方，全部用 `%ZYTemplateName%` 代替。

## 总结

整理后的发布框架流程

1. 利用 `pod lib create` 创建工程并与远程代码仓库关联
2. 修改各种文件信息为当前要发布的框架信息
2. 利用 `fastlane release_pod` 命令发布或更新框架

如果已经有工程，直接在工程根目录

1. 利用 ruby 脚本创建相应文件并修改需要发布的框架信息
2. 利用 `fastlane release_pod` 命令发布或更新框架

综上，虽然不能所以情况都一行命令发布框架，但是更新框架的时候基本上一行命令没问题。

文章到这里就结束了，如有疏漏，还望指正。如果对您有帮助，请不要吝啬你的 **star**🌹 哦！

## 参考

* [https://docs.fastlane.tools/](https://docs.fastlane.tools/)
* [https://guides.cocoapods.org/](https://guides.cocoapods.org/)
* [https://www.zybuluo.com/pockry/note/524101](https://www.zybuluo.com/pockry/note/524101)