---
layout: post
title: Combine 使用通知
date: 2021-03-05 12:00:00
comments: true
catagories: swift
tags: [iOS][Combine][Swift]
---

[参考]
(https://developer.apple.com/documentation/combine/routing-notifications-to-combine-subscribers)
通知中心是苹果开发者常用的功能，很多框架都会使用通知中心来向外部发送异步事件。对于iOS开发人员而言，以下代码一定非常眼熟：
<!--more-->

```
var notificationToken: NSObjectProtocol?
override func viewDidLoad() {
    super.viewDidLoad()
    notificationToken = NotificationCenter.default
        .addObserver(forName: UIDevice.orientationDidChangeNotification,
                     object: nil,
                     queue: nil) { _ in
                        if UIDevice.current.orientation == .portrait {
                            print ("Orientation changed to portrait.")
                        }
    }
}
```
现在，让我们来学习如何使用 Combine 处理通知，并将已有的通知处理代码迁移到 Combine。

```
var cancellable: Cancellable?
override func viewDidLoad() {
    super.viewDidLoad()
    cancellable = NotificationCenter.default
        .publisher(for: UIDevice.orientationDidChangeNotification)
        .filter() { _ in UIDevice.current.orientation == .portrait }
        .sink() { _ in print ("Orientation changed to portrait.") }
}
```
如上面的代码所示，在 Combine 中重写了最上面的代码。此代码使用了默认的通知中心来为orientationDidChangeNotification 通知创建发布者。当代码从该发布者接收到通知时，它使用过滤器操作符 filter(_:) 来实现只处理纵向屏幕通知的需求，然后打印一条消息。

需要注意的是，orientationDidChangeNotification 通知的 userInfo 字典中不包含新的屏幕方向，因此 filter(_:) 操作符直接查询了 UIDevice。

总结

 

虽然上面的示例无法突显 Combine 的优势，但是我们可以自行想象。使用 Combine 之后，如果需求变得很复杂，我们要做的可能只是增加操作符而已，而且不破坏链式调用代码的易读性。

朋友，行动起来吧！把现有项目中的旧代码重构成使用 Combine 的代码~
