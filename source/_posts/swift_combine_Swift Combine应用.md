---
layout: post
title: "Swift Combine 简介"
date: 2021-03-29 12:00:00
comments: true
catagories: swift
tags: [iOS][Combine][Swift]
---

在这篇文章中，着重介绍一些 Combine 的实际应用。
<!--more-->

# 发布者（Publisher）

假设有一个 MagicTrick 类型的 JSON 数据，这个数据的来源是 NotificationCenter，数据会以 Data 的形式放在 Notification 的 UserInfo 中。让我们使用 Combine 的发布者来改造这个数据的发送。
```
extension Notification.Name{
    static var newTrickDownloaded:Notification.Name {
        return Notification.Name("aa")
    }
}

class MagicTrick:Codable {
    var name:String = ""
}

let trickNamePublisher = NotificationCenter.Publisher(center: .default, name: .newTrickDownloaded)
```
由于 NotificationCenter 发布者的 Output 类型是 Notification，需要类型转换。

## map
这个操作符可以改变发布者的类型。
```
let trickNamePublisher = NotificationCenter.Publisher(center: .default, name: .newTrickDownloaded)
    .map{ notification -> Data in
        let userInfo = notification.userInfo
        return userInfo?["data"] as! Data
    }
```
这时候 Output 变成了 Data 类型，还不能实际使用。要转换成实际类型，可以用 Codable + JSONDecoder 进行解析，解析时可能会抛出错误，就需要使用 try 关键字。

## tryMap
这个操作符允许在转换值的闭包内抛出异常。
```
let trickNamePublisher = NotificationCenter.Publisher(center: .default, name: .newTrickDownloaded)
    .map{ notification -> Data in
        let userInfo = notification.userInfo
        return userInfo?["data"] as! Data
    }.tryMap { data -> MagicTrick in
        let decoder = JSONDecoder()
        return try decoder.decode(MagicTrick.self, from: data)
    }
```
## decode
而 Combine 为 Codable，还提供了便捷的 decode 方法。这个操作符允许传入 Decodable 类型和解码器，将解码器支持的上游数据类型解码，发送给下游订阅者。
```
let trickNamePublisher = NotificationCenter.Publisher(center: .default, name: .newTrickDownloaded)
    .map{ notification -> Data in
        let userInfo = notification.userInfo
        return userInfo?["data"] as! Data
    }
    .decode(type: MagicTrick.self, decoder: JSONDecoder())
```
接下来应该处理解码过程出现的错误了，因为每一个发布者需要描述了他们产生或者允许的错误类型，所以 Combine 中提供了各种各样的错误处理的操作符，对错误做出反应或是从错误中恢复并做一些兜底处理。

## assertNoFailure
这个操作符可以在你确认上游发布者不会产生错误时使用，会将错误类型转为 Never，但当错误发生时，将会崩溃。

```
let trickNamePublisher = NotificationCenter.Publisher(center: .default, name: .newTrickDownloaded)
    .map{ notification -> Data in
        let userInfo = notification.userInfo
        return userInfo?["data"] as! Data
    }
    .decode(type: MagicTrick.self, decoder: JSONDecoder())
    .assertNoFailure()
```

## catch
这个操作符允许在上游发布者发生错误时，提供一个默认的发布者替换上游的发布者，发送值给下游的订阅者，以便做默认兜底方案。
```
let trickNamePublisher = NotificationCenter.Publisher(center: .default, name: .newTrickDownloaded)
    .map{ notification -> Data in
        let userInfo = notification.userInfo
        return userInfo?["data"] as! Data
    }
    .decode(type: MagicTrick.self, decoder: JSONDecoder())
    .catch{ _ in
        return Publishers.Just(MagicTrick())
    }
```
Just：是一个很简单的发布者，用需要产生的值进行初始化，就会将该值发送一次给下游订阅者并结束。

## flatMap
上面的例子在错误发生后，生成一个 Just 的发布者作为上游发布者的替代品，但 Just 发布者只会产生一个值就结束了，整个事件流就会结束。但我们需要的是当错误发生时，catch 只处理这次错误，但不替换上游的发布者，也就是上游可以继续产生值。为了不影响上游，我们需要一个新的发布者，能将上游的值用新的发布者发送给下游，catch 只影响这个新的发布者，这个时候就需要 flatMap 了。

Combine 里的 flatMap 和函数式编程高阶函数里的 flatMap 一样，可以将包装的类型进行转换，在这里包装就是发布者，类型就是发布者的 Input，也就是说在 flatMap 里面可以返回一个新 Input 类型的发布者。
