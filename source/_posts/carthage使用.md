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
Carthage与CocoaPods类似，都是用于在iOS/OS X环境下管理第三方的工具,Carthage不会像CocoaPods那样创建一个workspace。而是直接提供了一种去中心化的依赖管理系统，不提供中心化的项目列表，使用者可以自行添加类库，对项目的侵入性也较少。可以理解为只帮你下载和更新第三方依赖，怎么用看你自己.Carthage使用xcodebuild去编译依赖，而不是将依赖集成到一个单一的工作区间，它没有类似的规范文件(例如CocoaPods 的 podspec)，但你的依赖必须包括它们自己的Xcode工程文件来描述是如何编译它们的项目。


# 安装Carthage:
推荐使用HomeBrew进行安装
如果没有安装homebrew，打开终端，输入命令

`/usr/bin/ruby -e"$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`

更新brew到最新版本

`brew update`

安装Carthage命令

`$ brew install carthage`

查看版本

`carthage version //(当前最新版本 0.34.0)`


# 使用Carthage

## 先进入到项目所在文件夹
   
   `$ cd 项目路径`

   创建一个空的Carthage文件

   `$ touch Cartfile`

## 使用xcode打开Cartfile文件
   
   `open -a Xcode Cartfile`

   编辑cartfile文件，添加依赖框架
   ```
github "Alamofire/AlamofireImage"
github "https://github.com/AFNetworking/AFNetworking
github "SVProgressHUD/SVProgressHUD" ~> 1.0
git "xxx"

#  支持 >= , ~> , == 不写默认取最新
github "ReactiveCocoa/ReactiveCocoa" >= 2.3.1

# 用分支
github "jspahrsummers/xcconfigs" "branch"

# 用自己的git服务器
git "https://enterprise.local/desktop/git-error-translations2.git" "development"

# 用本地git项目 
git "file:///directory/to/project" "branch"

# A binary only framework
binary "https://my.domain.com/release/MyFramework.json" ~> 2.3

# A binary only framework via file: url
binary "file:///some/local/path/MyFramework.json" ~> 2.3

# A binary only framework via local relative path from Current Working Directory to binary project specification
binary "relative/path/MyFramework.json" ~> 2.3

# A binary only framework via absolute path to binary project specification
binary "/absolute/path/MyFramework.json" ~> 2.3
   ```
   Carfile文件格式
   依赖源 Dependency origin
   Carthage支持两种类型的源，一个是 github ，另一个是 git 。
   1. github表示依赖源，告诉Carthage去哪里下载文件。依赖源之后跟上要下载的库，格式为Username/ProjectName
   2. git关键字后面跟的是资料库的地址，可以是远程的URL地址，使用git://, http://, ssh://，或者是本地资料库地址。
   3. binary json 文件格式示例
   ```
    Example binary project specification
    {
      "1.0": "https://my.domain.com/release/1.0.0/framework.zip",
      "1.0.1": "https://my.domain.com/release/1.0.1/framework.zip"
    }
```
   依赖版本号 Dependency Version
   
  告诉Carthage使用哪个版本，这是可选的，不写默认使用最新版本
   1.  == 1.0 表示使用1.0版本
   2.  = 1.0 表示使用1.0或更高的版本
   3.  ~> 1.0 表示使用版本1.0以上但是低于2.0的最新版本，如1.2，1.6
   4. branch名称 / tag名称 / commit名称，意思是使用特定的分支/标签/提交，比如可以是分支名master，也可以是提交5c8a74a。
   
## 保存并关闭cartfile文件，进行安装
   
   `$ carthage update --no-use-binaries --platform ios(或者 carthage update --platform iOS 这个命令有时候会有问题 ，建议加上 --no-use-binaries)`
   
   安装完之后根目录会出现一个叫Carthage的文件夹，里面包含Build和Checkouts两个文件夹。
   carthage会clone文件中对应的git第三方库，把每一个第三方库编译成二进制文件的framework文件。
### --platform
  其中 --platform iOS 命令是可选的，作用是保证只为iOS编译framework，如果不指定平台，会为全平台编译framework文件。如果想要了解更多的命令，可以运行 carthage help update 查看。
### -no-user-binary
  有些项目中已经存在打包好的 framework
  如果直接用 carthage update 会直接下载提供的 framework 到 carthage/build 而用
  carthage update -no-user-binary 的时候 会下载源码到 carthage/checkout 然后自己本地打包到 carthage/build 

  当命令执行完毕，在Cartfile文件同级别的文件夹中生成一个名为“Carthage”文件夹和“Cartfile.resolved”文件。打开Carthage文件夹，可以看到两个文件夹 Build 和 Checkouts 。
  ### Cartfile.resolved
  Cartfile.resolved ：这个文件是辅助 Cartfile 的，需要被提交到版本库中，它有助于其他开发者使用和你相同版本的第三方库。Build ：包含每一个第三方库创建生成的framework，可以被集成到项目中，每一个framework都是依赖于源文件或者GitHub上的 Releases 版本。它列出了为每个framework编译的实际版本。确保提交你的Cartfile.resolved，因为任何使用该项目的人将通过该文件来编译相同的framework版本.

  Checkouts ：这里包含的是转换成framework之前的源文件，Carthage有自己的缓存机制，所以不需要在不同的项目中对同一个的第三方库clone多次。

  对于是否把 Build 和 Checkouts 文件夹提交到版本库取决于你，这不是必须的。如果提交的话，其他人clone了你的资料库就可以使用这两个文件中的内容。

  不要改变 Checkouts 文件夹中的内容，因为如果使用 carthage update 或者 carthage checkout 命令的话，这个文件夹中的内容可以随时被复写，那么改动工作就白费了。如果一定要改动的话，在使用 carthage update 命令时，可以使用 --use-submodules 选项。如果加上这个选项的话，Carthage在添加每个依赖库的时候就会作为一个字模块。

  如果其他人想要使用你的工程，你不需要在你的代码中提交已经编译好的framework，他们需要在check out你的工程之后执行carthage bootstrap命令。

  bootstrap 命令会根据 Cartfile.resolved 文件下载和编译依赖库的精确版本。另一方面， carthage update 命令会更新项目中的第三方库的最新的编译版本，这是不可取的。

  在"Carthage/Build/iOS"文件夹中会生成 .framework 文件。

## 使用carthage build本地工程
`carthage build --platform ios --project-directory ${project} --no-skip-current
`
如果你在执行carthage build --no-skip-current
时编译失败，尝试执行xcodebuild -scheme SCHEME -workspace WORKSPACE build 或 xcodebuild -scheme SCHEME -project PROJECT build（将其中的大写单词换成你项目的对应名称），然后观察是否有相同的失败发生，这应该能生成足够的失败信息来解决问题。

# 添加FrameWorks到项目中
## Add Build Setting
点击"项目名称"-> "target" -> "Gerneral"，在最底部找到"Linked Frameworks and Libraries"。
打开Carthage文件夹，进入Build\iOS，拖拽*.framework到Xcode的 Linked Frameworks and Libraries中。

## Add New Run Script Phase 
项目Target -> Build Phases -> '+' -> New Run Script Phase,
添加脚本   
`/usr/local/bin/Carthage copy-frameworks`   
添加"Input Files"   
`$(SRCROOT)/Carthage/Build/iOS/AFNetworking.framework`   
添加"Output Files"    
`$(BUILT_PRODUCTS_DIR)/$(FRAMEWORKS_FOLDER_PATH)/Alamofire.framework`   
此脚本处理由通用二进制文件触发的App Store提交错误，并确保在归档时复制必需的bitcode-related文件和dSYM。   
With output files specified alongside the input files, Xcode only needs to run the script when the input files have changed or the output files are missing. This means dirty builds will be faster when you haven't rebuilt frameworks with Carthage.

通过将调试信息复制到已经编译的工程的目录中，只要在断点处停止，Xcode就能够对堆栈跟踪进行符号化。它也使你在调试器中通过第三方代码。

当打包程序提交到App Store或TestFlight时，Xcode还会将这些文件复制到应用程序的.xcarchive包的dSYMs子目录中。

![add](/img/article/res/carthage/1.webp)
   

## 在需要使用的地方import "xxx"

![add](/img/article/res/carthage/2.webp)

# 使用过程中遇到的问题

## unable to find utility 'xcodebuild, not a developer tool or in PATH'
在执行Carthage update后，控制台可能会打印这样的错误。  
![add](/img/article/res/carthage/3.webp)
原因是当git源码被checkout后，carthage会进行build。此时若是执行xcodebuild发生错误多半是因为在xcode中没有设置相应的编译工具选项。需要进到xcode的Preference中去设置Command Line Tools.

## 单独更新某一个框架
例如我新加了这两个框架，只需要更新它们，其他不需要更新。
github "Alamofire/Alamofire" github "ReactiveX/RxSwift"
则可以只执行这句
carthage update Alamofire
或者指定更新多个框架，空格隔开即可。
carthage update Alamofire RxSwift
当执行完后在命令行log中会发现依旧去fetch其他框架，不用担心，并不会重新CheckOut。只会CheckOut和build指定的依赖。
## Swift二进制框架下载兼容性
Carthage将检查以确保下载的Swift（和混合的Objective-C / Swift）框架是使用本地使用的相同版本的Swift构建的。 如果有版本不匹配，Carthage将继续从源代码构建框架。 如果框架不能从源代码构建，Carthage将失败。

因为Carthage使用xcrun swift --version的输出来确定本地Swift版本，所以请确保运行Carthage命令，使用你打算使用的Swift工具链。对于大多数情况，不需要额外的去注意整个问题。但是，举例来说，如果你使用Xcode8.x 去编译一个Swift2.3的项目，一种为carthage bootstrap指定默认swift的方法是使用以下命令    
`TOOLCHAINS=com.apple.dt.toolchain.Swift_2_3 carthage bootstrap
`
## 向单元测试或框架添加框架
对任何任意target,使用Carthage非常类似于前面提到的给应用添加frameworks。 主要的区别在于frameworks如何在Xcode中设置和链接。

因为单元测试target在其“General”设置选项卡中缺少“Linked Frameworks and Libraries”部分，所以必须将构建的frameworks拖动到“Link Binaries With Libraries”构建阶段。

在“Build Settings”选项卡下的测试目标中，将@ loader_path/Frameworks添加到“Runpath Search Paths”（如果尚未存在）。

在极少数情况下，你可能想将每个依赖复制到你构建的产品中（例如，在外部框架中嵌入依赖项，或确保测试包中存在依赖性）。 为此，使用“Framework”目标创建一个新的“Copy Files”构建阶段，然后在那里添加框架引用。


