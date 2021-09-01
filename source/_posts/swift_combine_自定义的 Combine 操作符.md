---
layout: post
title: Combine 使用 KVO
date: 2021-03-08 12:00:00
comments: true
catagories: swift
tags: [iOS][Combine][Swift]
---

Combine 中已经内置了很多操作符(发布者)，比如：map, filter, zip, merge, combineLatest 等。但是，Combine 并没有提供某些在 Rx 中较为常用的操作符，比如：withLatestFrom, concatMap 等。如果您感兴趣，可以参考这份详细的对照表：[RxSwift to Combine Cheatsheet](https://github.com/CombineCommunity/rxswift-to-combine-cheatsheet)。
<!--more-->

既然 Combine 没有提供，那么我们就自己造轮子吧！Ficow 将由浅入深地逐步打造自定义的操作符，希望能够带给您些许启发~

后文中的示例代码存储在这个 [Github 仓库](https://github.com/FicowShen/Ficow-Combine-SwiftUI/tree/master/CombineDemo) 中，仅供您参考。

# 创建 map 操作符
map 操作符的实现较为简单，Ficow 就以重写它作为例子。

首先，最简单的方式就是在 Publisher 协议类型的扩展中定义操作符并返回 Combine 提供的 Publishers.Map：
```
extension Publisher {
    func myMap1<Result>(transform: @escaping (Output) -> Result) -> Publishers.Map<Self, Result> {
        return Publishers.Map(upstream: self, transform: transform)
    }
}
```
然后，我们就可以使用这个自定义的操作符了：
```
        Just(1)
            .myMap1 { $0.description }
            .sink { value in
                print(value)
            }
```
接下来，我们不用 Publishers.Map 提供的实现，自行构建一个更为灵活的版本：

```
extension Publisher {
    func myMap2<Result>(transform: @escaping (Output) -> Result) -> Publishers.MyMap<Self, Result> {
        return Publishers.MyMap(upstream: self, transform: transform)
    }
}
extension Publishers {
    struct MyMap<Upstream: Publisher, Output>: Publisher {
        // 让自身的失败类型和上游保持一致
        typealias Failure = Upstream.Failure
        private let upstream: Upstream
        private let transform: (Upstream.Output) -> Output
        init(upstream: Upstream, transform: @escaping (Upstream.Output) -> Output) {
            // 引用上游发布者，在订阅后绑定下游和上游
            self.upstream = upstream
            self.transform = transform
        }
        /// 下游执行订阅操作时会调用这个方法，传入的参数就是下游订阅者
        func receive<S>(subscriber: S) where S : Subscriber, Self.Failure == S.Failure, Self.Output == S.Input {
            upstream
                // 每次上游发送值时，都首先执行这个转换操作
                .map(self.transform)
                // 让下游订阅这个 map 
                .receive(subscriber: subscriber)
        }
    }
}
```
在以上示例代码中，我们构建了一个自定义的 Publisher 类型：Publishers.MyMap。

由于 Publisher 协议中关联类型的约束，Publishers.MyMap 需要提供实际的 Output 类型和 Failure 类型。

因为 map 操作是需要对上游发送的值进行转换，然后将转换完成的值发送给下游。所以 Publishers.MyMap 中定义了 Upstream 类型遵守 Publisher 协议，同时自身的 Failure 类型等于 Upstream 的 Failure 类型（不对错误进行转换）。

到这一步，我们确实就已经实现了自定义的操作符。而且，只需要更改 .map(self.transform) 处的实现，这个 myMap2 操作符的效果就可以被更改。

 

myMap2 操作符的用法也是一样的：
```
Just(1)
    .myMap2 { $0.description }
    .sink { value in
        print(value)
    }
```
然而，这只是对一个流进行操作。如果我们需要对多个流进行操作，该怎么做呢？

# 创建 withLatestFrom 操作符
接下来，Ficow 就以创建 withLatestFrom 操作符为例，演示如何对多个流进行转换操作。
![image](/res/images/article/swift/combine8.png)
在开始之前，我们需要先理解 withLatestFrom 操作符的执行效果。withLatestFrom 顾名思义就是结合来自另一个流的最新值的意思。

请看上面的宝石图，最上面的箭头线是主流，主流的下面那条箭头线是辅助流。withLatestFrom 操作符向下游发送主流和辅助流的最新值的组合的前提条件是辅助流必须有值（比如 A）。但是这个发送操作只能由主流的最新值触发，而不是辅助流（比如 2A，而不是 1A）。

和 Ficow 一起想象用户输入帐号和密码然后进行登录的场景，辅助流是用户输入的帐号密码，主流是用户点击登录按钮的操作。只有登录按钮按下的操作才可以触发登录请求，无论帐号密码的值怎么更新，我们都不应该发起登录请求。

如果您对 withLatestFrom 操作符的实际使用场景感兴趣，您可以参考 Ficow 的文章： Combine + MVVM 如何构建易测试的 ViewModel？。

 

接下来，和 Ficow 一起开始构建操作符吧！

首先，这里和前文一样，在 Publisher 扩展中添加操作符方法：
```
extension Publisher {
    func myWithLatestFrom<Other: Publisher, Result>(_ other: Other, transform: @escaping (Output, Other.Output) -> Result) -> Publishers.MyWithLatestFrom<Self, Other, Result> {
        return Publishers.MyWithLatestFrom(upstream: self,
                                           second: other,
                                           transform: transform)
    }
}
```
然后，在 Publishers 类型的扩展中增加 MyWithLatestFrom 类型：
```
extension Publishers {
    struct MyWithLatestFrom<Upstream: Publisher, Other: Publisher, Output>: Publisher {
        typealias Failure = Upstream.Failure
        public typealias Transform = (Upstream.Output, Other.Output) -> Output
        private let upstream: Upstream
        private let second: Other
        private let transform: Transform
        init(upstream: Upstream, second: Other, transform: @escaping Transform) {
            self.upstream = upstream
            self.second = second
            self.transform = transform
        }
        func receive<S: Subscriber>(subscriber: S) where Failure == S.Failure, Output == S.Input {
            // 通过订阅来连接上下游
            subscriber.receive(subscription: Subscription(upstream: upstream,
                                                          downstream: subscriber,
                                                          second: second,
                                                          transform: transform))
        }
    }
}
```
MyWithLatestFrom 与前文中的 MyMap 有几个重要的不同点：

MyMap 与 1 个发布者进行交互，而 MyWithLatestFrom 与 2 个发布者进行交互；
MyMap 使用了 receive(subscriber:) 来绑定上下游，而 MyWithLatestFrom 使用了 receive(subscription:) 来绑定上下游；
其中，receive(subscription:) 方法接受的参数为 Subscription 协议类型。Subscription 代表的是订阅者与发布者之间的联系。

仔细看一下 Subscription 协议类型的定义：
```
@available(OSX 10.15, iOS 13.0, tvOS 13.0, watchOS 6.0, *)
public protocol Subscription : Cancellable, CustomCombineIdentifierConvertible {
    /// Tells a publisher that it may send more values to the subscriber.
    func request(_ demand: Subscribers.Demand)
}
```
请注意，Subscription 类型同时还是 Cancellable 类型，也就是说遵循 Subscription 协议的类型需要同时实现 request(_:) 和 cancel() 方法。然后，下游就可以向上游请求值，也可以向上游发送取消订阅的消息。

 

接下来，Ficow 将扩展 Publishers.MyWithLatestFrom 类型，构建一个实际的订阅类型 Subscription：
```
extension Publishers.MyWithLatestFrom {
    final class Subscription<Downstream: Subscriber>: Combine.Subscription where Downstream.Input == Output, Downstream.Failure == Failure {
        private let upstream: Upstream
        private let second: Other
        private let downstream: Downstream
        private let transform: Transform
        private var initialDemandOfDownstream = Subscribers.Demand.none
        private var otherSubscription: Combine.Subscription?
        private var latestValueFromOther: Other.Output?
        private var backPressureSubscriber: BackPressureSubscriber<Upstream, Downstream>?
        init(upstream: Upstream,
             downstream: Downstream,
             second: Other,
             transform: @escaping Transform) {
            self.upstream = upstream
            self.second = second
            self.downstream = downstream
            self.transform = transform
            // 创建订阅时就开始订阅第二个流
            trackLatestFromSecondStream()
        }
        func request(_ demand: Subscribers.Demand) {
            guard latestValueFromOther != nil else {
                // 暂存下游的需求，在订阅后再向上游请求
                initialDemandOfDownstream += demand
                return
            }
            // 当第二个流有值的时候，直接将下游订阅者的需求发给第一个流的上游发布者
            backPressureSubscriber?.requestDemand(demand)
        }
        func cancel() {
            // 取消两个流的订阅
            otherSubscription?.cancel()
            backPressureSubscriber = nil
        }
        private func trackLatestFromSecondStream() {
            var isTrackingMainUpstream = false
            let subcriber = AnySubscriber<Other.Output, Other.Failure>(
                receiveSubscription: { [weak self] (subscription) in
                    self?.otherSubscription = subscription
                    subscription.request(.unlimited)
                },
                receiveValue: { [weak self] (value) -> Subscribers.Demand in
                    self?.latestValueFromOther = value
                    if !isTrackingMainUpstream {
                        isTrackingMainUpstream = true
                        // 在第二个流有值时才开始订阅第一个流
                        self?.trackMainUpstream()
                    }
                    return .unlimited
                },
                receiveCompletion: { (completion) in
                })
            second.subscribe(subcriber)
        }
        private func trackMainUpstream() {
            backPressureSubscriber = BackPressureSubscriber(upstream: upstream,
                                                            downstream: downstream,
                                                            transformOutput: { [weak self] value in
                                                                guard let self = self,
                                                                    let other = self.latestValueFromOther
                                                                    else { return nil }
                                                                return self.transform(value, other) },
                                                            transformFailure: { $0 })
            // 向上游请求之前已暂存的下游订阅者的需求
            request(initialDemandOfDownstream)
            initialDemandOfDownstream = .none
        }
    }
}
```
上面的示例代码中用到了 BackPressureSubscriber 类型，这是一个自定义的订阅者类型。因为 Combine 提供了背压(back pressure)支持，所以当我们在定义操作符时也要尽可能支持背压。

如果您对背压感兴趣，可以参考 Ficow 的文章： Combine 框架，从0到1 —— 3.使用 Subscriber 控制发布速度。

接下来，Ficow 将定义一个支持背压的订阅者，它的作用就是持有上游发布者发送的订阅对象，之后根据下游订阅者的需求通过订阅对象来向上游发布者请求值，并在收到上游发布者发送的值或者完成时转发给下游订阅者。

BackPressureSubscriber 类型的源码如下：

```
@available(OSX 10.15, iOS 13.0, tvOS 13.0, watchOS 6.0, *)
final class BackPressureSubscriber<Upstream: Publisher, Downstream: Subscriber>: Subscriber {
    typealias TransformFailure = (Upstream.Failure) -> Downstream.Failure
    typealias TransformOutput = (Upstream.Output) -> Downstream.Input?
    private let upstream: Upstream
    private let downstream: Downstream
    private let transformOutput: TransformOutput
    private let transformFailure: TransformFailure
    private var subscription: Subscription?
    init(upstream: Upstream,
         downstream: Downstream,
         transformOutput: @escaping TransformOutput,
         transformFailure: @escaping TransformFailure) {
        self.upstream = upstream
        self.downstream = downstream
        self.transformOutput = transformOutput
        self.transformFailure = transformFailure
        upstream.subscribe(self)
    }
    deinit {
        // 销毁时，自动取消订阅
        cancelSubscription()
    }
    func requestDemand(_ demand: Subscribers.Demand) {
        guard demand > .none else { return }
        subscription?.request(demand)
    }
    func cancelSubscription() {
        subscription?.cancel()
        subscription = nil
    }
    func receive(subscription: Subscription) {
        self.subscription = subscription
    }
    func receive(_ input: Upstream.Output) -> Subscribers.Demand {
        guard let transformedInput = transformOutput(input) else {
            return .none
        }
        return downstream.receive(transformedInput)
    }
    func receive(completion: Subscribers.Completion<Upstream.Failure>) {
        switch completion {
        case .finished:
            downstream.receive(completion: .finished)
        case .failure(let error):
            downstream.receive(completion: .failure(transformFailure(error)))
        }
        cancelSubscription()
    }
}
extension BackPressureSubscriber: Cancellable {
    func cancel() {
        receive(completion: .finished)
    }
}
```
最后，在示例中应用 myWithLatestFrom 操作符并检查运行结果：

```
let subject1 = PassthroughSubject<Int, Never>()
        let subject2 = PassthroughSubject<String, Never>()
        var results = [String]()
        subject1
            .myWithLatestFrom(subject2, transform: { (intValue, stringValue) -> String in
                return intValue.description + stringValue
            })
            .sink { v in
                results.append(v)
            }
        subject2.send("2") // 辅助流发送的值，不会触发下游的接收操作
        subject2.send("3") // 辅助流的最新值为 3
        subject1.send(1)  // 主流发送的最新值，结合辅助流的最新值3，下游会执行接收操作并收到 "13"
        subject2.send("4")
        subject1.send(5) // 辅助流的最新值为 4，下游会收到 "54"
        subject1.send(6) // 辅助流的最新值为 4，下游会收到 "64"
        subject2.send("7")
        subject2.send("8")
        subject1.send(9) // 辅助流的最新值为 8，下游会收到 "98"
        print(results) // ["13", "54", "64", "98"]

```
通过自定义操作符，我们可以将一些常用的算法封装起来。在有效提升开发效率的同时，业务逻辑代码也会变得更简洁、易读。

但是，仅仅这样做是不够的，因为我们自定义的操作符很可能在某些应用场景下会有 bug。因此，为自定义操作符构建测试是非常有必要的。
# 为自定义操作符构建测试

没有足够多的测试用例覆盖的代码是非常不可靠的！每次自定义一个操作符之后，强烈建议您为该操作符构建单元测试，这样就可以保证该操作符的正确性。

接下来，Ficow 将为之前创建的自定义操作符构建测试代码：
```
final class TestCustomOperatorTests: XCTestCase {
    private var cancellables: Set<AnyCancellable>!
    override func setUp() {
        super.setUp()
        cancellables = Set<AnyCancellable>()
    }
    func testMyMap1() throws {
        let expect = expectation(description: #function)
        Just(1)
            .myMap1 { $0.description }
            .sink { v in
                XCTAssertEqual(v, "1")
                expect.fulfill()
            }
            .store(in: &cancellables)
        wait(for: [expect], timeout: 5)
    }
    func testMyMap2() throws {
        let expect = expectation(description: #function)
        Just(1)
            .myMap2 { $0.description }
            .sink { v in
                XCTAssertEqual(v, "1")
                expect.fulfill()
            }
            .store(in: &cancellables)
        wait(for: [expect], timeout: 5)
    }
    func testMyWithLatestFrom() throws {
        let expect = expectation(description: #function)
        expect.expectedFulfillmentCount = 4
        let subject1 = PassthroughSubject<Int, Never>()
        let subject2 = PassthroughSubject<String, Never>()
        var results = [String]()
        subject1
            .myWithLatestFrom(subject2, transform: { (intValue, stringValue) -> String in
                return intValue.description + stringValue
            })
            .sink { v in
                results.append(v)
                expect.fulfill()
            }
            .store(in: &cancellables)
        subject2.send("2")
        subject2.send("3")
        subject1.send(1)
        subject2.send("4")
        subject1.send(5)
        subject1.send(6)
        subject2.send("7")
        subject2.send("8")
        subject1.send(9)
        wait(for: [expect], timeout: 3)
        XCTAssertEqual(["13", "54", "64", "98"], results)
    }
}
```
上面的测试代码仅仅覆盖了最常见的使用场景。如果您需要测试自定义操作符的可靠性，您可能还需要考虑以下这些场景：

是否可以成功取消订阅；
发送完成或错误后能否得到预期的执行结果；
在多线程环境中运行是否安全、稳定；
# 总结
以上就是本文的所有内容，您已经学会了自定义 Combine 操作符。而且在学习自定义操作符的过程中，您对于 Combine 中常见的各种类型(发布者、订阅、订阅者)之间的协作机制也有了更深入的理解。除此之外，您还学会了如何为自定义操作符构建测试代码以保证操作符的代码质量。

未来已来，只是尚未流行。Combine 必将大幅提升苹果开发者的开发效率，而这离不开开源社区的贡献。现在已经有一些开源项目开发出了许多自定义操作符，如果您不想重复造轮子，您也可以使用这些项目为您提供的开源代码。

[CombineExt](https://github.com/CombineCommunity/CombineExt) 就是一个比较好的例子！如果您感兴趣，您可以使用、学习其中的自定义操作符，您也可以为开源社区贡献自己的一份力量~