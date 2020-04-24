---
layout: post
title: "Ruby环境搭建"
date: 2019-07-20 12:00:00
comments: true
catagories: language
tags: [Ruby]
---

rbenv和rvm 是用来管理 ruby 的，ruby 的其中一个“程序”叫 rubygems ，简称 gem，而用来管理项目的 gem 的，叫 bundle ，他俩完全是不同的东西，相同的只是都可以管理gem。bundler 用来管理 fastlane 自身版本和 fastlane 运行时的相关依赖版本, 相当于 iOS 开发中的 CocoaPods 框架,CocoaPods就是模仿Bundle的工作原理开发出来的，bundle中用于管理gem的文件叫gemfile

<!--more-->


 # 安装 rbenv | rvm



## 安装 rbenv

### rbenv installer & doctor scripts

rbenv-installer脚本可以在系统上安装或更新rbenv。 如果检测到Homebrew，将使用brew install / upgrade进行安装。 否则，将rbenv安装在`〜/.rbenv`下。

另外，如果rbenv install还不可用，那么还将安装ruby-build

安装后，将运行单独的rbenv-doctor脚本以验证安装是否成功并检测常见问题。 您可以在计算机上分别运行rbenv-doctor来验证安装状态：

```
# with curl
curl -fsSL https://github.com/rbenv/rbenv-installer/raw/master/bin/rbenv-installer | bash

# alternatively, with wget
wget -q https://github.com/rbenv/rbenv-installer/raw/master/bin/rbenv-installer -O- | bash
# with curl
curl -fsSL https://github.com/rbenv/rbenv-installer/raw/master/bin/rbenv-doctor | bash

# alternatively, with wget
wget -q https://github.com/rbenv/rbenv-installer/raw/master/bin/rbenv-doctor -O- | bash
```
 ```
$ brew install rbenv #会自动安装ruby-build,升级使用这个脚本brew upgrade rbenv ruby-build
$ rbenv init #可以把此命令放在.bash_profile/.zshrc文件中
 ```
装rvm 执行以下代码
```
$ \curl -sSL https://get.rvm.io | bash -s stable
```
注意rbenv和rvm不兼容的，不可以两个都装,可以使用下面的命令检查安装情况
```
curl -fsSL https://github.com/rbenv/rbenv-installer/raw/master/bin/rbenv-doctor | bash
```

 ## rbenv使用
 ### 初始化
 ```
 echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc

# 如果使用的是 Zsh  
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(rbenv init -)"' >> ~/.zshrc

```

 常用的只有四个命令，其他命令的用法可以通过rbenv --help来查看

 ### 设置本地版本
 ```
 $ rbenv local 2.4.1
 $ rbenv local --unset #取消设置
 ```
 执行rbenv local显示当前工作目录下的 ruby 版本，local后面带上版本号2.4.1则是设置的效果。

### 设置全局版本
```
rbenv local jruby-1.7.3      # 当前目录使用 jruby-1.7.3, 会生成一个 `.rbenv-version` 文件
rbenv global system #使用系统版本
```
跟上述用法类似，只不过global指的是当前终端的 ruby 版本。

### 设置当前终端版本
```
rbenv shell 1.9.3-p392       # 当前的 shell 使用 1.9.3-p392, 会设置一个 `RBENV_VERSION` 环境变量
rbenv shell --unset  #取消设置
```

### 安装ruby版本
```
$ rbenv install -l 或 rbenv install --list  # 列出所有 ruby 版本
$ rbenv install 1.9.3-p392     # 安装 1.9.3-p392
$ rbenv install jruby-1.7.3    # 安装 jruby-1.7.3
```
执行上面的命令会输出目前有效可安装的版本，找到你想要的把-l替换成版本号。例如：rbenv install 2.4.2，使用该命令会安装在
`~/.rbenv/versions`中

### rehash
每当切换ruby版本和执行bundle install之后必须执行这个命令
```
rbenv rehash
```

### 查看版本

```
rbenv versions               # 列出安装的版本
rbenv version                # 列出正在使用的版本
```

### 其他命令

```
rbenv which irb              # 列出 irb 这个命令的完整路径
rbenv whence irb             # 列出包含 irb 这个命令的版本
```
### 安装gem 包
```
$ gem install bundler
```

可以使用下面命令查看gem 被安装的路径

```
$ gem env home
# => ~/.rbenv/versions/<ruby-version>/lib/ruby/gems/...
# =>/Library/Ruby/Gems/2.6.0
```



### rbenv 下使用 gemset   
rvm 中最方便的就是 gemset。实际上，rbenv 通过插件也可以使用 gemset
MacOS 下使用 brew 的话，一个命令就搞定

brew install rbenv-gemset
创建一个 gemset
```
rbenv gemset create 1.9.3-p392 ruby-china
                       参数 1       参数 2
```
以上命令中，参数 1 是已安装的 ruby 版本，参数 2 是 gemset 的名字
具体使用方法

在项目的根目录下，把想要使用的 gemset 名字放到 .rbenv-gemsets 文件中即可。有 .rbenv-gemsets 文件的情况下执行 bundle 命令就是对设置好的 gemset 进行操作
```
echo ruby-china > .rbenv-gemsets
```
当前目录下没有 .rbenv-gemsets 文件的情况下，执行 bundle 命令（没有指定 --path 参数的情况）时，是对当前版本的 ruby 版本的 gemset 。也就相当于 rvm 中 global gemset 的作用了
[rbenv 官方文档](https://github.com/rbenv/rbenv) 


### 手动安装rbenv
```
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
# 用来编译安装 ruby
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
# 用来管理 gemset, 可选, 因为有 bundler 也没什么必要
git clone git://github.com/jamis/rbenv-gemset.git  ~/.rbenv/plugins/rbenv-gemset
# 通过 rbenv update 命令来更新 rbenv 以及所有插件, 推荐
git clone git://github.com/rkh/rbenv-update.git ~/.rbenv/plugins/rbenv-update
# 使用 Ruby China 的镜像安装 Ruby, 国内用户推荐
git clone git://github.com/AndorChen/rbenv-china-mirror.git ~/.rbenv/plugins/rbenv-china-mirror
```
### 卸载Ruby

直接删除 `~/.rbenv/versions`中对应的ruby版本，或者使用 ruby-build 这个插件提供的`rbenv uninstall `

### 卸载rbenv


## rvm 简介
```
$ rvm list known
```
跟rbenv install -l的效果一样，输出有效可安装的版本。

```
$ rvm install 2.4.1
```
安装指定版本号的 ruby 环境
```
$ rvm use “ruby version”@“gemset name” --create
```
在执行上面这行命令之前，先到你的工作目录下，手动创建.ruby-gemset和.ruby-version，这两个都是文本内容。.ruby-gemset里要写的可以是一个跟项目相关的名字，会在你指定的版本号环境下创建一个目录，存放工作目录下的gem依赖包。.ruby-version里写的时候要注意，按照ruby-2.4.1这个样子，ruby-加版本号。
