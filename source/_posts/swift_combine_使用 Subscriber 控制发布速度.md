---
layout: post
title: Combine 使用 Subscriber 控制发布速度
date: 2021-03-04 12:00:00
comments: true
catagories: swift
tags: [iOS][Combine][Swift]
---

对于大多数响应式编程场景而言，订阅者不需要对发布过程进行过多的控制。当发布者发布元素时，订阅者只需要无条件地接收即可。但是，如果发布者发布的速度过快，而订阅者接收的速度又太慢，我们该怎么解决这个问题呢？Combine 已经为我们制定了稳健的解决方案！现在，让我们来了解如何施加背压(back pressure，也可以叫反压)以精确控制发布者何时生成元素。
<!--more-->
[参考]
(https://developer.apple.com/documentation/combine/processing-published-elements-with-subscribers)
在 Combine 中，发布者生成元素，而订阅者对其接收的元素进行操作。不过，发布者会在订阅者连接和获取元素时才发送元素。订阅者通过 Subscribers.Demand 类型来表明自己可以接收多少个元素，以此来控制发布者发送元素的速率。

订阅者可以通过两种方式来表明需求(Demand)：

调用 Subscription 实例（由发布者在订阅者进行第一次订阅时提供）的 request(_:) 方法；
在发布者调用订阅者的 receive(_:) 方法来发送元素时，返回一个新的 Subscribers.Demand 实例；
Demand 是可以累加的。如果订阅者已经请求了两个元素，然后请求 Subscribers.Demand(.max(3))，则现在发布者不满足的需求是五个元素。如果发布者随后发送元素，则未满足的需求将减少到四个。
发布元素是减少未满足需求的数量的唯一方法，订阅者不能请求负需求。
 

很多应用会使用 sink(receiveValue:) 和 assign(to:on:) 来创建便捷的订阅者类型，分别为：Subscribers.Sink 和 Subscribers.Assign。这两种订阅者在第一次连接到发布者时，会发送一个 unlimited 的 Demand，这时候订阅者会一直不停地接收发布者发来的内容。

# 在发布者生产元素时消耗它们
当发布者的需求很高或不受限制时，它发送元素的速度可能比订阅者处理元素的速度快很多。这种情况可能导致元素丢失，或者在元素等待被缓存时迅速增加内存的压力。

如果您使用便捷的订阅者，则会发生这种情况，因为它们的需求(Demand) 是无限数量 (unlimited) 的元素。确保您提供给 sink(receiveValue:) 的闭包和 assign(to:on:) 的副作用(执行效果)遵循以下特征：

不会阻塞发布者；
不会因为缓存元素而消耗过多的内存；
不会不知所措并且不能处理元素;
庆幸的是，许多常用的发布者（例如与用户界面元素相关联的发布者）都会以可控的速度进行发布。其他常见的发布者仅仅生成一个元素，例如：URL 加载系统的 URLSession.DataTaskPublisher。配合这些发布者，使用 sink(receiveValue:) 和 assign(to:on:) 订阅者是绝对安全的。

# 使用自定义的订阅者施加背压(back pressure)

想要控制发布者向订阅者发送元素的速率，可以创建订阅者协议的自定义实现。使用你的自定义实现来指定你的订阅者可以适应的需求。当订阅者接收元素时，它可以通过返回新的需求值给 receive(_:) 方法，或通过在订阅上调用 request(_:) 来请求更多内容。无论使用哪种方法，你自定义的订阅者都可以在任何给定时间微调发布者可以发送的元素数量。

 

通过发信号来表明订阅者已准备好接收元素来控制流量的概念称为背压
每个发布者都跟踪其当前未满足的需求，也就是：订阅者已请求多少个元素。甚至，像 Foundation 框架中的 Timer.TimerPublisher 这样的自动化资源，也只会在有未满足的需求时才产生元素。

下面的示例代码说明了这个行为：
```
// 发布者: 使用一个定时器来每秒发送一个日期对象
let timerPub = Timer.publish(every: 1, on: .main, in: .default)
    .autoconnect()
// 订阅者: 在订阅以后，等待5秒，然后请求最多3个值
class MySubscriber: Subscriber {
    typealias Input = Date
    typealias Failure = Never
    var subscription: Subscription?
    func receive(subscription: Subscription) {
        print("published                             received")
        self.subscription = subscription
        DispatchQueue.main.asyncAfter(deadline: .now() + 5) {
            subscription.request(.max(3))
        }
    }
    func receive(_ input: Date) -> Subscribers.Demand {
        print("\(input)             \(Date())")
        return Subscribers.Demand.none
    }
    func receive(completion: Subscribers.Completion<Never>) {
        print ("--done--")
    }
}
// 订阅 timerPub
let mySub = MySubscriber()
print ("Subscribing at \(Date())")
timerPub.subscribe(mySub)
```

订阅者的 receive(subscription:) 实现在请求发布者的任何元素之前执行了五秒钟的延迟。在此期间，发布者存在并具有有效的订阅者，但需求为零，因此不会产生任何元素。它仅在延迟到期且订阅者给它一个非零需求 subscription.request(.max(3)) 之后才开始发布元素，如以下输出所示：
```
Subscribing at 2019-12-09 18:57:06 +0000
published                             received
2019-12-09 18:57:11 +0000             2019-12-09 18:57:11 +0000
2019-12-09 18:57:12 +0000             2019-12-09 18:57:12 +0000
2019-12-09 18:57:13 +0000             2019-12-09 18:57:13 +0000
```
这个示例只请求了三个元素，在五秒钟的延迟到期后发出需求。最后，发布者在第三个元素之后不再发送其他元素，但是也不会通过发送完成(.finished) 的值来完成发布，因为发布者只是在等待更多需求。为了继续接收元素，订阅者可以存储订阅并定期请求更多元素。它还可以在 receive(_:) 方法中返回新需求的值。

这个示例只请求了三个元素，在五秒钟的延迟到期后发出需求。最后，发布者在第三个元素之后不再发送其他元素，但是也不会通过发送完成(.finished) 的值来完成发布，因为发布者只是在等待更多需求。为了继续接收元素，订阅者可以存储订阅并定期请求更多元素。它还可以在 receive(_:) 方法中返回新需求的值。
即使没有自定义的订阅者，你也可以通过一些操作符来实施背压：

buffer(size:prefetch:whenFull:)，保留来自上游发布者的固定数量的项目。缓冲满了之后，缓冲区会丢弃元素或抛出错误；
debounce(for:scheduler:options:)，只在上游发布者在指定的时间间隔内停止发布时才发布；
throttle(for:scheduler:latest:)，以给定的最大速率生成元素。如果在一个间隔内接收到多个元素，则仅发送最新的或最早的元素;
collect(_:) 和 collect(_:options:) 聚集元素，直到它们超过给定的数量或时间间隔，然后向订阅者发送元素数组。如果订阅者可以同时处理多个元素，这个操作符将是很好的选择。
由于这些操作符可以控制订阅者接收的元素数量，因此可以放心地连接无限需求的订阅者，例如：sink(receiveValue:) 和 assign(to:on:)。
总结

通过实施背压，我们可以灵活地调控发布过程。背压操作符可以帮助我们应对大多数场景，这些操作符可以大幅提升我们的开发效率。

比如这种常见的场景：当搜索输入框的内容发生变动时，应用需要去查找用户输入内容对应的结果，但是这个查找操作的频率需要有一定的控制。如果用户按住一个键不放开，输入框的内容就会一直变化，此时就会触发多次查找操作。这时候，我们可以从容地使用背压操作符解决这种问题。

如果你需要处理的场景非常复杂，通过自定义订阅者来实施精确的背压将会是一个更好的选择。
