---
layout: post
title: Combine 中执行异步代码
date: 2021-03-05 12:00:00
comments: true
catagories: swift
tags: [iOS][Combine][Swift]
---

你的应用可能会使用一些常见的模式来处理异步事件，比如：
完成处理器(Completion handlers)。它其实是调用方提供的一个闭包，当一个耗时任务完成后，这个闭包会被调用一次；
闭包属性(Closure properties)。它其实也是调用方提供的一个闭包，这个闭包会在每一次异步事件发生时被调用；
[参考](https://developer.apple.com/documentation/combine/using-combine-for-your-app-s-asynchronous-code)
<!--more-->
Combine 为这些模式提供了强大的替换选项。它可以让你消除这种样板代码，并且充分利用 Combine 中的操作符。当你在应用的其他地方采用 Combine 时，将异步调用点转换为 Combine 可以提高代码的一致性和可读性。

# 用 Future 取代回调闭包
一个完成处理器其实就是一个传给某个方法的闭包参数，当这个方法完成任务时，它就会调用这个闭包。

比如下面的代码，这个函数接收一个闭包，然后在延时2秒之后执行这个闭包：
```
func performAsyncAction(completionHandler: @escaping () -> Void) {
    DispatchQueue.main.asyncAfter(deadline:.now() + 2) {
        completionHandler()
    }
}
```
你可以用 Combine 中的 Future 去替换这种模式的代码。用 Future 创建一个发布者去执行一些工作，然后异步发送成功或者失败的信号。如果执行成功，Future 就会执行一个 Future.Promise，这是一个用来接收 Future 生成的元素的闭包。

你可以像这样重写上面的代码：
```
func performAsyncActionAsFuture() -> Future <Void, Never> {
    return Future() { promise in
        DispatchQueue.main.asyncAfter(deadline:.now() + 2) {
            promise(Result.success(()))
        }
    }
}
```
当任务完成时，Future 会调用传入的 promise， 并把代表执行成功或者失败的结果 Result 传给 promise，而不是像之前的代码那样调用一个显式传入的闭包。然后，调用方会通过 Future 异步接收这个结果。由于 Future 是一个 Combine 发布者，调用方会把自己连接到一个可选的操作符链上，链的尾部是一个订阅者，比如：sink(receiveValue:)。

调用方的代码如下所示：
```
cancellable = performAsyncActionAsFuture()
    .sink() { _ in print("Future succeeded.") }
```

# 用输出类型(Output Types)代表 Future 的参数
有时，一个耗时任务生成的结果需要作为一个参数传递给完成处理器闭包。要在 Combine 中实现同样的功能，就需要为这个参数声明 Future 发布的输出类型。

下方的示例可以生成一个随机整型数。它将 Future 发布的输出类型声明为 Int类型， 并将生成的整型值传递给 promise：
```
func performAsyncActionAsFutureWithParameter() -> Future <Int, Never> {
    return Future() { promise in
        DispatchQueue.main.asyncAfter(deadline:.now() + 2) {
            let rn = Int.random(in: 1...10)
            promise(Result.success(rn))
        }
    }
}
```
请注意，performAsyncActionAsFuture 方法的返回值是 Future <Void, Never> 类型，而 performAsyncActionAsFutureWithParameter 方法的返回值是 Future <Int, Never> 类型。
通过声明 Future 产生的元素为 Int 类型，Future 可以使用 Result 给 promise 传入整型值。当 promise 执行结束时，Future 会发布这个值，然后调用方就可以通过订阅者(如：sink(receiveValue:))接收到这个值：
```
cancellable = performAsyncActionAsFutureWithParameter()
    .sink() { rn in print("Got random number \(rn).") }
```
# 用 Subject 取代重复执行的闭包

你的应用也可能会采用这种常见的模式：将一个闭包作为一个属性，然后当某个事件发生时执行这个闭包属性。这种属性的命名通常以 on 开头，然后调用点看起来就像这样：
```
vc.onDoSomething = { print("Did something.") }
```
有了 Combine，你可以使用 Subject 替代这种模式。一个 Subject 实例允许你在任何时候通过调用 send() 方法来命令式地发布一个新元素。你可以通过使用私有的 PassthroughSubject 或者 CurrentValueSubject 来采用这种模式，然后将其作为一个 AnyPublisher 暴露给外部以便外部调用方使用它：
```
private lazy var myDoSomethingSubject = PassthroughSubject<Void, Never>()
lazy var doSomethingSubject = myDoSomethingSubject.eraseToAnyPublisher()
// 发布一个空元组元素
myDoSomethingSubject.send(())
```
然后，调用方只需要在订阅者中执行对应的操作即可，不需要配置一个闭包属性：
```
cancellable = vc.doSomethingSubject
    .sink() { print("Did something with Combine.") }
```
这种基于 Combine 的方法还有一个优势，subject 可以调用 send(completion:) 来告知订阅者：不会再有后续的事件发生，或者发生了一个错误。
总结

 

通过学习上述内容，我们可以感觉到：迁移现有的异步代码到 Combine 是比较容易的。而且，因为 Combine 提供了很多常用的操作符，它可以极大地提升我们的开发效率！

可以想象一下，以前写的很多方法/函数，现在只需要使用 Combine 就可以写成非常易读而且优雅的链式调用代码。

