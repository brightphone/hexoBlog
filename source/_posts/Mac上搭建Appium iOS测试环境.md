---
layout: post
title: "Mac 上搭建Appium iOS测试环境"
date: 2019-02-13 12:00:00
comments: true
catagories: Appium
tags: [Appium]
---


# Mac 上通过Appium实现自动化测试

## [安装Homebrew](https://brew.sh)

Homebrew是一款Mac OS平台下的软件包管理工具，拥有安装、卸载、更新、查看、搜索等很多实用的功能。简单的一条指令，就可以实现包管理，而不用你关心各种依赖和文件路径的情况，十分方便快捷.

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

<!--more-->

## 安装NodeJS

  ```
  brew install node
  ```
（安装nodeJS时会默认安装上icu4c）

## 安装Carthage
carthage在后面编译WebDriverAgent的时候会使用到
```
brew install carthage //install
brew uninstall carthage //uninstall
```

## 安装Appium
```
npm install -g appium // install
npm uninstall -g appium //uninstall
```


## 安装appium-doctor
使用appium-doctor校验Appium的依赖环境是否正确配置
```
npm install -g appium-doctor
```
校验环境
```
appium-doctor --ios
```
在使用上面命令时遇见一个奇怪的问题是提示`WARN AppiumDoctor  ✖ Xcode is NOT installed!`
通过`sudo xcode-select -s /Applications/Xcode.app/Contents/Developer`命令解决了这个问题
```
Options:
  -h, --help                  print this help message and exit
  -p, --print-path            print the path of the active developer directory
  -s <path>, --switch <path>  set the path for the active developer directory
  --install                   open a dialog for installation of the command line developer tools
  -v, --version               print the xcode-select version
  -r, --reset                 reset to the default command line tools path
```
# 真机测试
## 下载安装[WebDriverAgent](https://github.com/facebook/WebDriverAgent/)

cd 到WebDriverAgent目录执行命令：
```
./Scripts/bootstrap.sh
```
命令执行完成后用xcode打开WebDriverAgent工程
修改工程中的Bundle ID和证书，这里使用personal 证书
如果运行后log如下说明WebDriverAgent运行成功了
```
2019-02-14 10:25:49.709807+0800 WebDriverAgentRunner-Runner[7231:729750] +[CATransaction synchronize] called within transaction
2019-02-14 10:25:49.927258+0800 WebDriverAgentRunner-Runner[7231:729750] Running tests...
Test Suite 'All tests' started at 2019-02-14 10:25:53.993
Test Suite 'WebDriverAgentRunner.xctest' started at 2019-02-14 10:25:53.998
Test Suite 'UITestingUITests' started at 2019-02-14 10:25:54.000
Test Case '-[UITestingUITests testRunner]' started.
    t =     0.01s Start Test at 2019-02-14 10:25:54.012
    t =     0.05s Set Up
2019-02-14 10:25:54.118209+0800 WebDriverAgentRunner-Runner[7231:729750] Built at Feb 14 2019 10:25:07
2019-02-14 10:25:54.368368+0800 WebDriverAgentRunner-Runner[7231:729750] ServerURLHere->http://10.60.22.70:8100<-ServerURLHere
```
如果build 成功但没有提示ServerURLHere，尝试把手机重启一下。

## 安装libimobiledevice
libimobiledevice又称libiphone，是一个开源包，可以让Linux支持连接iPhone/iPod Touch等iOS设备。开源项目的地址：https://github.com/libimobiledevice/libimobiledevice ，感兴趣的同学可以下载研究一下,安装libmobiledevice时会自动安装libplist, libtasn1, openssl, libusb and usbmuxd。如果没有安装 libimobiledevice，会导致Appium无法连接到iOS的设备，所以必须要安装，如果要在iOS10+的系统上使用appium，则需要安装ios-deploy
```
brew install -HEAD libimobiledevice #安装最新的更新，支持 iOS 10
brew install ideviceinstaller  # 仅在 iOS9工作,ipa安装命令
brew install ios-deploy
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
成功运行WebDriverAgent后，通过浏览器输入 `http://10.60.22.70:8100`，发现没有反应，这是因为：有些国产的iPhone机器通过手机的IP和端口还不能访问，此时需要将手机的端口转发到Mac上。关于这个问题，通过端口转发才看到效果，所以你也应该会遇到同样的问题，[可以参考这里](https://github.com/facebook/WebDriverAgent/wiki/USB-support)。

运行以下命令：
```
iproxy 8100 8100
```
这时在浏览器中输入：http://localhost:8100/status 即可看到response.(即使不使用iproxy，在浏览器中输入`http://10.60.22.70:8100`没反应也不会影响后面的测试)

输入http://localhost:8100/inspector 可以查看手机屏幕，不过很慢很慢



# 使用Appium client写测试用例

## 使用Java_client
这里使用IntelliJ创建Maven工程   
![image](/res/images/article/AppiumiOS/Snip20190227_1.png)   
![image](/res/images/article/AppiumiOS/Snip20190227_2.png)

pom.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.Iight</groupId>
    <artifactId>appiumTestDemo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-java</artifactId>
            <version>3.14.0</version>
        </dependency>
        <dependency>
            <groupId>com.github.appium</groupId>
            <artifactId>java-client</artifactId>
            <version>7.0.0</version>
        </dependency>
    </dependencies>


</project>
```
在test>java下建立一个class，名为testDemo
添加代码如下
```
import org.openqa.selenium.remote.DesiredCapabilities;
import org.openqa.selenium.remote.RemoteWebDriver;
import io.appium.java_client.remote.*;

import java.net.MalformedURLException;
import java.net.URL;


public class testDemo {
    public static void main(String[] args) throws MalformedURLException, InterruptedException {

        DesiredCapabilities desiredCapabilities = new DesiredCapabilities();
        desiredCapabilities.setCapability(MobileCapabilityType.DEVICE_NAME, "light");
        desiredCapabilities.setCapability(MobileCapabilityType.AUTOMATION_NAME, "XCUITest");
        desiredCapabilities.setCapability(MobileCapabilityType.PLATFORM_NAME, "iOS");
        desiredCapabilities.setCapability(MobileCapabilityType.PLATFORM_VERSION, "11.4.1");
        desiredCapabilities.setCapability(MobileCapabilityType.APP, "com.baidu.map");
        desiredCapabilities.setCapability(MobileCapabilityType.UDID, "yourdevice");
        URL url = new URL("http://10.60.22.78:4723/wd/hub");
        RemoteWebDriver iosDriver = new RemoteWebDriver(url, desiredCapabilities);
    }
}
```
开启appium server
打开终端，输入
>appium -a 10.60.22.78 -p 4723
这样就开启了一个appium server，通过xcode，coomand + u 运行之前的webDriverAgent

运行java testDemo即可打开百度地图.