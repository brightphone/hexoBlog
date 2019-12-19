---
title: "carthage基本使用"
catalog: true
toc_nav_num: true
date: 2019-04-18 12:00:00
header-img: "/img/article_header/article_header.png"
tags:
- Carthage
catagories:
- [Carthage]

---
# 简介  
Carthage与CocoaPods类似，都是用于在iOS/OS X环境下管理第三方的工具,Carthage不会像CocoaPods那样创建一个workspace。而是直接提供了一种去中心化的依赖管理系统，不提供中心化的项目列表，使用者可以自行添加类库，对项目的侵入性也较少。可以理解为只帮你下载和更新第三方依赖，怎么用看你自己.

安装Carthage:
推荐使用HomeBrew进行安装

`$ brew install carthage`

使用Carthage
1. 先进入到项目所在文件夹

`$ cd 项目路径`

创建一个空的Carthage文件

`$ touch Cartfile`

3. 编辑cartfile文件，添加依赖框架
```
github "Alamofire/AlamofireImage"
github "https://github.com/AFNetworking/AFNetworking"
git "xxx"
```
4. 保存并关闭cartfile文件，进行安装

`$ carthage update --no-use-binaries --platform ios`

安装完之后根目录会出现一个叫Carthage的文件夹，里面包含Build和Checkouts两个文件夹。

Build中/iOS路径下的就是framework包，需要自行引用进来。

Checkouts是从Github上获取来的源码，所以理论上来说你在这个文件夹里对源码进行任何的修改，再次执行 `carthage build` 就会根据这里的源码打包出相应的framework出来。

但需要注意的是当每次执行carthage update后这里的源码又被覆盖了。所以你有特别需要修改的地方可以加例外防止覆盖!!!! 重要

5. 项目Target -> Build Setting -> Search Paths -> Framework Search Paths添加
$(PROJECT_DIR)/Carthage/Build/iOS

6. 项目Target -> Build Phases -> '+' -> New Run Script Phase,
添加脚本 `/usr/local/bin/Carthage copy-frameworks`
添加"Input Files" `$(SRCROOT)/Carthage/Build/iOS/AFNetworking.framework`

