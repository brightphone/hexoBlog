---
layout: post
title: "Combine的核心概念"
date: 2021-03-01 12:00:00
comments: true
catagories: swift
tags: [iOS][Combine][Swift]
---

响应式编程（Reactive Programming）是一种编程思想，相对应的也有面向过程编程、面向对象编程、函数式编程等等。不同的是，响应式编程的核心是面向异步数据流和变化的.
<!--more-->

在现在的前端世界中，我们需要处理大量的事件，既有用户的交互，也有不断的网络请求，还有来自系统或者框架的各种通知，因此也无可避免产生纷繁复杂的状态。使用响应式编程后，所有事件将成为异步的数据流，更加方便的是可以对这些数据流可以进行组合变换，最终我们只需要监听所关心的数据流的变化并做出响应即可。

类型定义

如图所示，这三个协议就是 Combine 的核心。Publisher 负责发布内容，Subscriber 负责接收内容，Subscription 作为中介，协调生产端和消费端的需求。
![image](/res/images/article/swift/combine.png)
Publisher(发布者) 的源码：
```
@available(OSX 10.15, iOS 13.0, tvOS 13.0, watchOS 6.0, *)
public protocol Publisher {
    /// 这个发布者发布的值的类型
    associatedtype Output
    /// 这个发布者可能发布的错误的类型
    ///
    /// 如果这个发布者不发布错误，就用 `Never`
    associatedtype Failure : Error
    /// 在调用 `subscribe(_:)` 方法时，这个方法会被触发，并连接指定的 `Subscriber 到这个发布者
    ///
    /// - SeeAlso: `subscribe(_:)`
    /// - Parameters:
    ///     - subscriber: 被连接到这个发布者上的订阅者。连接后，订阅者就可以开始接收值
    func receive<S>(subscriber: S) where S : Subscriber, Self.Failure == S.Failure, Self.Output == S.Input
}
```
Subscriber(订阅者) 的源码：
```
@available(OSX 10.15, iOS 13.0, tvOS 13.0, watchOS 6.0, *)
public protocol Subscriber : CustomCombineIdentifierConvertible {
    /// 这个订阅者要接收的值的类型
    associatedtype Input
    /// 这个订阅者可能接收到的错误的类型
    ///
    /// 如果这个订阅者不会接收到错误，使用 `Never`
    associatedtype Failure : Error
    /// 告知订阅者成功订阅了发布者，并且可以获取发布项
    ///
    /// 使用收到的 `Subscription` 来向发布者请求内容
    /// - Parameter subscription: 订阅，代表发布者和订阅者之间的连接
    func receive(subscription: Subscription)
    /// 告知订阅者，发布者已经发布了一个元素
    ///
    /// - Parameter input: 发布了的元素
    /// - Returns: 命令，指明订阅者还期望接收多少元素
    func receive(_ input: Self.Input) -> Subscribers.Demand
    /// 告知订阅者，发布者已经结束了发布，可能是正常结束，也可能是因为发生了错误
    ///
    /// - Parameter completion: 完成，指明发布结束是正常结束还是由于错误而结束
    func receive(completion: Subscribers.Completion<Self.Failure>)
}
```
Subscription(订阅) 的源码：
```
public protocol Subscription : Cancellable, CustomCombineIdentifierConvertible {
    /// 通知发布者，它可以向订阅者发送一个或多个值
    func request(_ demand: Subscribers.Demand)
}
```
交互流程
![image](/res/images/article/swift/combine2.png)
1.Subscriber 被绑定到 Publisher 上；
2.Publisher 创建订阅对象(subscription)，并将订阅对象发送给 Subscriber；
3.Subscriber 通过订阅对象将需求发送给 Publisher；
4.Publisher 根据需求，将内容发送给 Subscriber；
5.Subscriber 通过订阅对象来向 Publisher 请求取消订阅；
6.Publisher 向 Subscriber 发送完成内容；
现在，让我们使用 Combine 来写一个发起网络请求的示例：
```
class CombineDemo {
    var cancellable: AnyCancellable?
    func makeRequest() {
        let url = URL(string: "https://ficow.cn")!
        let dataTaskPublisher = URLSession.shared.dataTaskPublisher(for: url)
        cancellable = dataTaskPublisher.sink(
            receiveCompletion: { completion in
                // 发布结束的时候会被调用一次
                switch completion {
                case .failure(let error):
                    print(error)
                case .finished:
                    print("success")
                }
            }, receiveValue: { value in
                // 每次接收到发布者发送的值都会被调用一次
                // 因为发起的是网络请求，所以这里只会被调用一次
                print(value.data)
                print(value.response)
            })
    }
}
```
dataTaskPublisher 是系统提供的方法，它会返回一个发布者（Publisher）。然后我们可以对这个 publisher 调用 sink（此处应翻译为：接收）方法，以此创建一个基于闭包的订阅者（Subscriber）。

sink 方法也有一个返回值，类型为 AnyCancellable，我们可以用这个值来取消订阅。当这个值在内存中被销毁时，订阅也会被自动取消。所以，如果我们不希望这个订阅在 makeRequest() 方法执行结束时停止，就要在实例中强引用这个 cancellable。
如果你想提前结束订阅，可以对这个 cancellable 调用 cancel 方法：

cancellable?.cancel()
此时，我们还没有用到操作符。现在，对上面的示例稍作调整：

```
 cancellable = dataTaskPublisher
            .delay(for: .seconds(2), scheduler: DispatchQueue.global()) // 在后台线程中去延时执行
            .receive(on: RunLoop.main) // 在主线程上接收发布的内容
            .sink(receiveCompletion: { completion in
```
如果使用传统的GCD，这里的代码就会变成两个闭包：

```
DispatchQueue.global().asyncAfter(deadline: .now() + 2) {
            // 发起网络请求操作
            dataTask() { (response) in
                DispatchQueue.main.async {
                    // 切换到主线程
                }
            }
        }
```
接下来，我们研究一下官方提供的一个基于 AppKit 的示例，我稍微做了一些处理以适应UIKit。如下所示：
```
class OfficialDemo {
    class MyViewModel {
        var filterString = ""
    }
    private let filterField = UITextField()
    private let myViewModel = MyViewModel()
    private var subscription: AnyCancellable?
    func bind() {
        subscription = NotificationCenter.default
            .publisher(for: UITextField.textDidChangeNotification, object: filterField)
            .map( { (($0.object as! UITextField).text ?? "") } )
            .filter( { $0.unicodeScalars.allSatisfy({CharacterSet.alphanumerics.contains($0)}) } )
            .debounce(for: .milliseconds(500), scheduler: RunLoop.main)
            .receive(on: RunLoop.main)
            .assign(to:\MyViewModel.filterString, on: myViewModel)
    }
}
```
bind 方法中的代码完成了很多任务：

通过通知中心来订阅输入框 filterField 的 textDidChangeNotification 通知；
通过 map 将接收到的内容转换为输入框中的文本；
通过 filter 来过滤掉无效的文本内容，阻止内容继续沿着订阅链往后传递；
通过 debounce(for:scheduler:) 来控制内容往后传递的频率（收到内容之后的500毫秒后执行后续操作，如果在这个时间段内收到了新内容，则重新计时500毫秒再执行后续操作）；
通过 receive(on:) 来指定执行后续操作的调度器（线程）；
通过 assign(to:on:) 来使用 keypath 为指定的对象赋值（每次收到内容就会执行一次）；
如果不使用 Combine，上面这一系列的任务可能需要写非常多的代码才能完成。而且代码的组织结构将会变得很庞大，你可能需要写很多方法来封装这些操作。


# AnyPublisher、AnySubscriber、AnySubject
通用类型,任意的 Publisher、Subscriber、Subject 都可以通过 eraseToAnyPublisher()、eraseToAnySubscriber()、eraceToAnySubject() 转化为对应的通用类型。
```
class StudentManager {

    let namePublisher: AnyPublisher<[String, Never]>

    func updateStudentsFromLocal() {
        let namePublisher: AnyPublisher<[String, Never]> = Publishers.Sequence<[Student], Never>(sequence: students).map { $0.name }.eraseToAnyPublisher()
        self.namePublisher = namePublisher
    }

    func updateStudentsFromNetwork() {
        let namePublisher: AnyPublisher<[String, Never]> = Publishers.Future { promise in
            promise(.success([names]))
        }.eraseToAnyPublisher()
        self.namePublisher = namePublisher
    }
}
```
AnyPublisher、AnySubscriber、AnySubject 的另外一个实用场景是创建自定义的 Publisher/Subscriber/Subject，因为在框架中它们已经是实现好相应的协议了。


