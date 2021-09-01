---
layout: post
title: Combine 通过 ConnectablePublisher 控制何时发布
date: 2021-03-04 12:00:00
comments: true
catagories: swift
tags: [iOS][Combine][Swift]
---

使用 Connectable Publisher， 你可以决定发布者何时开始发送订阅元素给订阅者。那么，为什么我们需要这么做？
<!--more-->
使用 sink(receiveValue:) 可以立刻开始接收订阅元素，但是这可能不是你想要的结果。当多个订阅者订阅了同一个发布者时，有可能会出现其中一个订阅者收到订阅内容，而另外一个订阅者收不到的情况。

比如，当你发起一个网络请求，并为这个请求创建了一个发布者以及连接了这个发布者的订阅者。

![image](/res/images/article/swift/combine3.png)
# 使用 makeConnectable() 和 connect() 控制发布
ConnectablePublisher 是一个协议类型，它可以在你准备好之前阻止发布者发布元素。
```
/// 可连接的发布者，它提供了显式的连接、取消订阅的方式
///
/// 使用 `makeConnectable()` 来从任何一个失败类型是 `Never` 的发布者创建一个 `ConnectablePublisher`
@available(OSX 10.15, iOS 13.0, tvOS 13.0, watchOS 6.0, *)
public protocol ConnectablePublisher : Publisher {
    /// 连接到发布者并返回一个用于取消发布的 `Cancellable` 实例
    ///
    /// - 返回值: 一个用于取消发布的 `Cancellable` 实例
    func connect() -> Cancellable
}
```
在你显式地调用 connect() 方法之前，一个 ConnectablePublisher 不会发送任何元素。

现在，就让我们用 ConnectablePublisher 来解决上面提到的网络请求示例中的问题吧！
![image](/res/images/article/swift/combine4.png)
在两个订阅者都连接到发布者之后，调用 connect()，然后网络请求才被触发。这样就可以避免竞争(race condition)，保证两个订阅者都收到数据。

为了在你的 Combine 代码中使用 ConnectablePublisher，你可以使用 makeConnectable() 操作符将当前的发布者包装到一个 Publishers.MakeConnectable 结构体实例中。

如下方的代码所示：

```
class ConnectablePublisherDemo {
    private var cancellable1: AnyCancellable?
    private var cancellable2: AnyCancellable?
    private var connection: Cancellable?
    func run() {
        let url = URL(string: "https://ficow.cn")!
        let connectable = URLSession.shared
            .dataTaskPublisher(for: url)
            .map(\.data)
            .catch() { _ in Just(Data()) }
            .share()
            .makeConnectable() // 阻止发布者发布内容
        cancellable1 = connectable
            .sink(receiveCompletion: { print("Received completion 1: \($0).") },
                  receiveValue: { print("Received data 1: \($0.count) bytes.") })
        DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
            self.cancellable2 = connectable.sink(receiveCompletion: { log("Received completion 2: \($0).") },
                                                 receiveValue: { log("Received data 2: \($0.count) bytes.") })
        }
        DispatchQueue.main.asyncAfter(deadline: .now() + 2) {
            // 显式地启动发布。返回值需要被强引用，可用于取消发布（主动调用cancel方法或返回值被析构）
            self.connection = connectable.connect() 
        }
    }
}
```
请注意，在 makeConnectable() 操作符前面有一个 share() 操作符！请问，这个操作符有什么作用呢？

# 使用 autoconnect() 操作符进行自动连接
某些 Combine 发布者已经实现了 ConnectablePublisher 协议，如：Publishers.Multicast 和 Timer.TimerPublisher。使用这些发布者时，如果你不需要配置发布者或者不需要连接多个订阅者，你就需要显式地调用 connect() 方法。

对于这种情况，ConnectablePublisher 提供了 autoconnect() 操作符。当一个订阅者通过 subscribe(_:) 方法连接到发布者时，connect() 方法会被马上调用。
```
let cancellable = Timer.publish(every: 1, on: .main, in: .default)
    .autoconnect()
    .sink() { date in
        print ("Date now: \(date)")
     }
```
上面的代码示例中使用了 autoconnect()，所以订阅者可以马上接收到定时器发送的元素。如果没有 autoconnect()，我们就需要在某个时刻手动地调用 connect() 方法。

Combine 为我们提供了很强大的异步编程功能，不过这也是有代价的，我们需要深知使用 Combine 过程中可能会遭遇的问题。如果不了解这些“坑”就开始上路，犯错的概率会非常高，犯错的成本也会非常高。