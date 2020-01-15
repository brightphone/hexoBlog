---
layout: post
title: "libimobiledevice的安装和常用命令"
date: 2019-02-25 12:00:00
comments: true
catagories: Tools
tags: [libimobiledevice]
---


libimobiledevice 是一个跨平台的软件库，【类似android adb 方便获取iOS设备信息】,支持 iPhone®, iPod Touch®, iPad® and Apple TV® 等设备的通讯协议。不依赖任何已有的私有库，不需要break jail。应用软件可以通过这个开发包轻松访问设备的文件系统、获取设备信息，备份和恢复设备，管理 SpringBoard 图标，管理已安装应用，获取通讯录、日程、备注和书签等信息，使用 libgpod 同步音乐和视频。

<!--more-->

[官方网站](http://www.libimobiledevice.org)

[github地址](https://github.com/libimobiledevice)

# 安装方式

## Install For MacOS

```
brew install -HEAD libimobiledevice #安装最新的更新，支持 iOS 10
brew install ideviceinstaller  # 仅在 iOS9工作,ipa安装命令
```
如果安装过程报下面错误：
"configure: error: Package requirements (libusbmuxd >= 1.1.0) were not met:
Requested 'libusbmuxd >= 1.1.0' but version of libusbmuxd is 1.0.10"
试试下面的方法：
```
brew update
brew uninstall --ignore-dependencies libimobiledevice
brew uninstall --ignore-dependencies usbmuxd
brew install --HEAD usbmuxd
brew unlink usbmuxd
brew link usbmuxd
brew install --HEAD libimobiledevice
brew install ideviceinstaller
```
## Install For Ubuntu

```
$ sudo add-apt-repository ppa:pmcenery/ppa
$ sudo apt-get update
$ apt-get install libimobiledevice-utils
$ sudo apt-get install ideviceinstaller
```
# 常用命令

## 查看当前所连接的设备

```
MacBookPro:~ lemon$ idevice_id -l # 显示当前所连接的设备[udid]，包括 usb、WiFi 连接
********c06e788b2d8dc60004a7015ce5dad782
********9a816a4089bd28f4f2e63c57a8138c63

instruments -s devices      # 列出设备包括模拟器、真机及 mac 电脑本身
```

## 安装应用

```
ideviceinstaller -u [udid] -i [xxx.ipa] # 给指定连接的设备安装应用
```

## 卸载应用

ideviceinstaller -u [udid] -U [bundleId] # 给指定连接的设备卸载应用

## 查看设备已安装的应用

```
ideviceinstaller -u [udid] -l                   # 指定设备，查看安装的第三方应用
ideviceinstaller -u [udid] -l -o list_user      # 指定设备，查看安装的第三方应用
ideviceinstaller -u [udid] -l -o list_system    # 指定设备，查看安装的系统应用
ideviceinstaller -u [udid] -l -o list_all       # 指定设备，查看安装的系统应用和第三方应用
```

## 获取设备信息

```
ideviceinfo -u [udid]                       # 指定设备，获取设备信息
ideviceinfo -u [udid] -k DeviceName         # 指定设备，获取设备名称：iPhone6s
idevicename -u [udid]                       # 指定设备，获取设备名称：iPhone6s
ideviceinfo -u [udid] -k ProductVersion     # 指定设备，获取设备版本：10.3.1
ideviceinfo -u [udid] -k ProductType        # 指定设备，获取设备类型：iPhone8,1
ideviceinfo -u [udid] -k ProductName        # 指定设备，获取设备系统名称：iPhone OS
```
## 查看系统日志
```
idevicesyslog
```
使用的这个命令时遇到下面问题
```
ERROR: Could not start service com.apple.syslog_relay.
Could not start logger for udid bceb000b4abae4f122ff2222d73ca6a85b8186da
```

## 截图
```
idevicescreenshot
//如果在使用截图的时候出现报错信息，那么就去把相应版本的DeveloperDiskImage的两个文件复制到libimobiledevice文件下面。

路径：
/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport/对应版本/

获取版本号命令：
ideviceinfo -k ProductVersion

安装DeveloperDiskImage命令：
ideviceimagemounter DeveloperDiskImage.dmg
//然后就可以正常截图了
```

## 获取设备时间
```
idevicedate
```

## 重启设备
```
idevicediagnostics restart
```

## 关机
```
idevicediagnostics shutdown
```

## 休眠
```
idevicediagnostics sleep
```
# 出错处理
使用的时候可能会报
"Could not connect to lockdownd. Exiting."使用下面的试试
```
brew update
brew uninstall --ignore-dependencies libimobiledevice
brew uninstall --ignore-dependencies usbmuxd
brew install --HEAD usbmuxd
brew unlink usbmuxd
brew link usbmuxd
brew install --HEAD libimobiledevice
brew install ideviceinstaller
```