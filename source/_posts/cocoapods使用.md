---
layout: post
title: "cocoapods基本使用"
date: 2019-04-20 12:00:00
comments: true
catagories: Tools
tags: [cocoapods]
---
 
CocoaPods通过集中式的管理，可以非常有效的管理第三方库，甚至可以用于大型项目的模块化管理，非常优雅高效的解决iOS项目中的依赖管理。

<!--more-->

# 安装CocoaPods:
CocoaPods是一个Ruby Gem，因为直接访问RubyGem速度非常慢，建议先替换成淘宝镜像

```ruby
$ gem sources --remove https://rubygems.org/ //移除系统 ruby 默认源
$ gem sources -a https://ruby.taobao.org/ //使用新的源或--add
$ gem sources -l  //验证是否替换成功
```
安装CocoaPods
```ruby
$ sudo gem install cocoapods //按照命令
$ gem uninstall cocoapods //卸载命令
```

# 管理第三方库
## 创建Podfile
在项目根目录下创建Podfile，下面是一个Podfile的例子 (详情可以参考http://guides.cocoapods.org/syntax/podfile.html#podfile)：

```ruby
platform :ios, '9.0'
use_frameworks!

target "MyApp" do
  pod 'ObjectiveSugar', '~> 0.5'
 
  target "MyAppTests" do
    pod 'OCMock', '~> 2.0.1'
  end
end
```

platform: 指定的版本是仓库兼容的最小版本

use_frameworks!则表明依赖的库编译生成.frameworkds的包，而不是.a的包

target: 指定的是作用于工程中的那个目标

pod: 用来指定相关的仓库及仓库版本。下方是相关仓库版本的几种常见的指定方式：

pod 'xxxx' : 后方没有指定版本，则表示使用仓库的最新版本。

pod 'xxxx', '2.3' : 使用xxxx仓库的2.3版本。

pod 'xxxx', '~>2.3': 则表示使用的版本范围是 2.3 <= 版本 < 3.0。如果后方指定版本是~>2.3.1, 那么则表示使用的版本范围是 2.3.1 <= 版本 < 2.4.0。

pod 'xxxx', '>2.3': 使用大于2.3的版本。

pod 'xxxx', '>=2.3': 使用2.3及以上的版本。

pod 'xxxx', '<2.3': 使用小于2.3的版本。

pod 'xxxx', '<=2.3': 使用小于等于2.3的版本。


pod 'AFNetworking', :path => '~/Documents/AFNetworking'  -- 使用本地路径引入

pod 'AFNetworking', :git => 'https://github.com/gowalla/AFNetworking.git', :tag => '0.7.0'  -- 使用git库引入

pod 'JSONKit', :podspec => 'https://example.com/JSONKit.podspec'  -- 使用外部的podspec来引入

## 安装Pods
安装pods
```ruby
$ pod install
```
更新pods
```ruby
$ pod update
```
install和update的区别：假如使用 pod 'SVProgressHUD'，没有指定版本。使用pod install，如果Pods中存在SVProgressHUD，则直接使用。使用pod update，则会保证更新SVProgressHUD到最新版本。

install或update速度通常很慢，因为每次执行的时候都需要同步一下CocoaPods Specs，这个有几百兆的大小，同步一次非常耗时。所以如果你使用的第三方库并不是经常更新，则不用经常更新那个Specs库。可以使用以下命令：
```ruby
$ pod install --verbose --no-repo-update
$ pod update --verbose --no-repo-update

执行完install或者update命令后，就可以使用.xcworkspace打开项目。
```
初始第三方库信息：
```
pod setup
```

初始第三方库信息：`pod setup`（此时会花费较长时间，另开一终端，cd ~/.cocoapods到该目录下 ，执行：du -sh *，可以查看下载进度）    
(`pod update` 更新第三方库信息) 
(`pod search`  搜索是否存在第三方框架)

# 使用CocoaPods管理私有库
## 大型项目模块化管理
随着iOS APP越来越复杂，功能越来越多，对于iOS项目的工程化要求也越来越高了，对于复杂的APP一般都需要对项目进行模块化管理。

模块化有几个方式：

1. 目录结构管理：这是最原始的方式，仅仅通过目录结构实现代码层次的清晰化。但本质上并没有解决代码之间的依赖混乱的情况，模块化划分也非常不清晰。

2. 子工程：通过子工程可以实现代码依赖管理和模块化，但是需要引入复杂的设置，不利于管理。

3. 静态库：将依赖代码打包成为静态库.a，不过由于不能看到源码，调试不方便。

自从有了CocoaPods，可以使用它来管理私有库，从而实现了代码模块化管理。例如下图所示：  
![image](/res/images/article/cocoapods/1.png)
# CocoaPods私有库
## 创建私有的Specs git库
例如在github上面创建一个空的git库：https://github.com/xxx/MySpecs
将这个git库加入到CocoaPods库的列表中：
```ruby
$ pod repo add MySpecs git@github.com:xxx/MySpecs.git
```
此时可以检查下本地的pod repo
```ruby
<br class="Apple-interchange-newline">$ pod repo list<br><br>MySpecs
```

- Type: git (master)- URL: git@github.com:xxx/MySpecs.git
- Path: /Users/xxx/.cocoapods/repos/mySpecs
 
master
- Type: git (master)
- URL:  git@github.com:CocoaPods/Specs.git
- Path: /Users/xxx/.cocoapods/repos/master
　　确定私有库的Specs已经加到本地pod repo中。
## 在私有库项目中创建podspec文件
在私有库项目中的根目录，创建对应的podspec文件，里面会描述这个库的基本信息。

PodSpec规范可以查看：https://guides.cocoapods.org/syntax/podspec.html
```ruby
#
#  Be sure to run `pod spec lint PodName.podspec' to ensure this is a
#  valid spec and to remove all comments including this before submitting the spec.
#  To learn more about Podspec attributes see http://docs.cocoapods.org/specification.html
#  To see working Podspecs in the CocoaPods repo see https://github.com/CocoaPods/Specs/
#
Pod::Spec.new do |s|
  s.name         = "PodName"
  s.version      = "0.0.1"
  s.summary      = "A short description of PodName."
  s.homepage     = "http://github.com/xxx/PodName"
  s.license      = { :type => "MIT", :text => <<-LICENSE
    Copyright © 2016年 xxx. All rights reserved.
    LICENSE
     }
  s.author       = { "" => "" }
  s.source       = { :git => "git@github.com:xxx/PodName.git", :tag => "0.0.1" }
  s.source_files = "**/*.{h,m,mm,c}"
  s.frameworks   = "Foundation", "QuartzCore", "UIKit", "WebKit"
  s.libraries    = "z"
  
  s.dependency 'AFNetworking'
  s.ios.deployment_target = '6.0'
end
```
resource: 可以指定资源文件，建议使用bundle以避免资源文件产生冲突。

frameworks: 指定这个pod依赖的系统framework

libraries: 指定这个pod依赖的系统动态库。注意使用的名字：比如需要引用"libz.dylib", 那么这里只需要写"z"

 

无论原始项目的目录结构或者group结构，默认的pod里面的代码都会平铺在根目录里面

如果需要增加目录层次结构，则需要使用subspec，详细使用规范：https://guides.cocoapods.org/syntax/podspec.html#subspec

注意：SubSpecs之间不能存在相互依赖关系，只能单向依赖
## 验证私有库的合法性
```ruby
$ pod lib lint --sources='git@github.com:xxx/MySpecs.git' --verbose --use-libraries --allow-warnings
```
sources参数可以指定私有库的Pod Specs库的地址。如果能够通过，说明代码编译没有问题。

## 提交私有库的版本信息
```ruby
$ git tag -m "first release" "0.0.1"
$ git push --tags     #推送tag到远端仓库
```
## 向Spec Repo提交podspec
```ruby
$ pod repo push MySpecs PodName.podspec --sources='git@github.com:xxx/MySpecs.git' --use-libraries --allow-warnings
```
这样就完成了一个CocoaPods的私有库的提交了，别人就可以在Podfile里面使用这个私有库了。
# cocoapods 更多使用
```ruby
platform :ios, '9.0'
inhibit_all_warnings!

target 'MyApp' do
    pod 'ObjectiveSugar', '~> 0.5' 

    target "MyAppTests" do
       inherit! :search_paths 
       pod 'OCMock', '~> 2.0.1'
     end
end
post_install do |installer|     
    installer.pods_project.targets.each do |target| 
        puts "#{target.name}" 
    end
end
```
## 根选项（Root Options
```ruby
install! 'cocoapods' , 
          :deterministic_uuids => false , 
          :integrate_targets => false
```
所支持的关键词:
:clean
:deduplicate_targets
:deterministic_uuids
:integrate_targets
:lock_pod_sources
:share_schemes_for_development_pods
