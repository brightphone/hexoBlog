---
layout: post
title: Combine 使用 KVO
date: 2021-03-05 12:00:00
comments: true
catagories: swift
tags: [iOS][Combine][Swift]
---
KVO(Key-Value Observing) 是苹果开发者常用的功能，很多框架都会使用 KVO 来发送异步改动。将基于回调和闭包的 KVO 代码迁移到 Combine，可以使你的代码更优雅、更易维护。
[参考](https://developer.apple.com/documentation/combine/performing-key-value-observing-with-combine)
<!--more-->

# 用 KVO 监控改动
在下面的示例中， UserInfo 类型为它的 lastLogin 属性提供了 KVO 支持。示例代码在 viewDidLoad() 方法中调用了 observe(_:options:changeHandler:) 方法来创建了一个处理这个属性的变动的闭包。这个闭包接收一个描述了改动事件的 NSKeyValueObservedChange 对象，并从这个对象取出了 newValue 属性的值，然后打印。

接下来，示例代码在 viewDidAppear(_:) 方法中改了 lastLogin 属性的值，最终触发了闭包并打印了消息。
```
class UserInfo: NSObject {
    @objc dynamic var lastLogin: Date = Date(timeIntervalSince1970: 0)
}
@objc var userInfo = UserInfo()
var observation: NSKeyValueObservation?
override func viewDidLoad() {
    super.viewDidLoad()
    observation = observe(\.userInfo.lastLogin, options: [.new]) { object, change in
        print ("lastLogin now \(change.newValue!).")
    }
}
override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)
    userInfo.lastLogin = Date()
}
```
# 将 KVO 代码迁移到 Combine
想要迁移 KVO 代码到 Combine，只需替换 observe(_:options:changeHandler:) 方法为 NSObject.KeyValueObservingPublisher。您可以通过在父对象(parent object)上调用 publisher(for:) 方法来获得此发布者的实例，如以下示例的 viewDidLoad() 方法所示：

```
class UserInfo: NSObject {
    @objc dynamic var lastLogin: Date = Date(timeIntervalSince1970: 0)
}
@objc var userInfo = UserInfo()
var cancellable: Cancellable?
override func viewDidLoad() {
    super.viewDidLoad()
    cancellable = userInfo.publisher(for: \.lastLogin)
        .sink() { date in print ("lastLogin now \(date).") }
}
override func viewDidAppear(_ animated: Bool) {
    super.viewDidAppear(animated)
    userInfo.lastLogin = Date()
}
```

KVO发布者 会生成观察类型的元素（在本例中为 Date），而不是 NSKeyValueObservedChange。这样就节省了一步操作，因为你不必像前一个示例中那样从 change 对象中获取 newValue。

如果上面的示例变得更加复杂，传统的 KVO 代码可能会变得非常臃肿，而基于 Combine 的 KVO 代码可以利用各种操作符进行链式调用。这样，就可以让代码更优雅，同时保持代码的易读性。