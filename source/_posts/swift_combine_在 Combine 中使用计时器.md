---
layout: post
title: Combine 使用计时器
date: 2021-03-05 12:00:00
comments: true
catagories: swift
tags: [iOS][Combine][Swift]
---
计时器是苹果开发者常用的功能。如果你的应用使用 Foundation 框架中的计时器 Timer 来周期性地执行某些操作，你可以用 Combine 简化这些代码。

现在，让我们来学习如何使用 Combine 处理计时器，并将已有的计时器处理代码迁移到 Combine。
[参考](https://developer.apple.com/documentation/combine/replacing-foundation-timers-with-timer-publishers)
<!--more-->

# 使用计时器执行周期性的工作
对于 iOS 开发人员而言，以下代码一定非常眼熟：
```
var timer: Timer?
override func viewDidLoad() {
    super.viewDidLoad()
    timer = Timer.scheduledTimer(withTimeInterval: 1, repeats: true) { _ in
        self.myDispatchQueue.async() {
            self.myDataModel.lastUpdated = Date()
        }
    }
}
```
以上代码使用 scheduledTimer(withTimeInterval:repeats:block:) 来实现每秒钟在 myDispatchQueue 中更新 self.myDataModel.lastUpdated 的功能。

# 将计时器转换为计时器发布者(Timer.TimerPublisher)
要将以上代码迁移到 Combine，只需将 Timer（scheduledTimer(withTimeInterval:repeats:block:)的返回值） 替换为 Timer.TimerPublisher。调用 Timer. publish(every:tolerance:on:in:options:) 方法即可创建一个发布者。

每次底层的计时器(Timer)触发时，发布者都会发出一个新的日期(Date)实例，该日期代表计时器触发的瞬间。然后，你可以将 Combine 操作符应用到这个日期实例上，最终将这个发布者和一个订阅者(如：sink(receiveValue:) 或 assign(to:on:))连接。
由于 Timer.TimerPublisher 遵从 ConnectablePublisher 协议，因此在您显式地连接之前，它不会产生任何元素。为此，可以通过手动调用 connect() 或使用 autoconnect() 运算符在订阅者连接时自动连接来实现。关于 ConnectablePublisher 的用法，可以参考 [Combine 通过 ConnectablePublisher 控制何时发布]。
下一个示例将展示如何使用 Timer.TimerPublisher 替换上一个示例。它使用 Combine 的操作符来完成上一个示例中的闭包中的操作：

```
var cancellable: Cancellable?
override func viewDidLoad() {
    super.viewDidLoad()
    cancellable = Timer.publish(every: 1, on: .main, in: .default)
        .autoconnect()
        .receive(on: myDispatchQueue)
        .assign(to: \.lastUpdated, on: myDataModel)
}
```
在这个例子中，Combine 操作符替换了上一个示例的闭包中的所有行为：

receive(on:options:) 操作符确保了后续操作在指定的调度队列中执行，它替代了前面用到的 async() 调用；
assign(to:on:) 操作符通过键路径来更新数据模型的 lastUpdated 属性；
 

使用 Combine 来简化你的代码时，你会发现 Timer.TimerPublisher 会产生新的 Date 实例作为其输出类型。而第一个示例的闭包是将 Timer 本身作为其参数，因此它必须手动创建新的 Date 实例。

使用 Combine 来简化你的计时器代码时，你会发现：

代码易读性明显提升；
线程切换变得更简单；
数据模型的更新可以通过键路径(key path)来简化；
