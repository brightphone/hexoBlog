---
layout: post
title: "Combine 中的 Subjects"
date: 2021-03-01 12:00:00
comments: true
catagories: swift
tags: [iOS][Combine][Swift]
---

Subject 是一类比较特殊的发布者，因为它同时也是订阅者。Combine 提供了两类 Subject ：PassthroughSubject 和 CurrentValueSubject。
<!--more-->
# PassthroughSubject
PassthroughSubject 可以向下游订阅者广播发送元素。使用 PassthroughSubject 可以很好地适应命令式编程场景。

如果没有订阅者，或者需求为0，PassthroughSubject 就会丢弃元素。
```
final class SubjectsDemo {
    private var cancellable: AnyCancellable?
    private let passThroughtSubject = PassthroughSubject<Int, Never>()
    func passThroughtSubjectDemo() {
        cancellable = passThroughtSubject
            .sink {
                print(#function, $0)
            }
        passThroughtSubject.send(1)
        passThroughtSubject.send(2)
        passThroughtSubject.send(3)
    }
}
```
输出内容：

passThroughtSubjectDemo() 1
passThroughtSubjectDemo() 2
passThroughtSubjectDemo() 3


# CurrentValueSubject
CurrentValueSubject 包装一个值，当这个值发生改变时，它会发布一个新的元素给下游订阅者。

CurrentValueSubject 需要在初始化时提供一个默认值，您可以通过 value 属性访问这个值。在调用 send(_:) 方法发送元素后，这个缓存值也会被更新。

示例代码：
```
final class SubjectsDemo {
    private var cancellable: AnyCancellable?
    private let currentValueSubject = CurrentValueSubject<Int, Never>(1)
    func currentValueSubjectDemo() {
        cancellable = currentValueSubject
            .sink { [unowned self] in
                print(#function, $0)
                print("Value of currentValueSubject:", self.currentValueSubject.value)
            }
        currentValueSubject.send(2)
        currentValueSubject.send(3)
    }
}
```
输出内容：

currentValueSubjectDemo() 1
Value of currentValueSubject: 1
currentValueSubjectDemo() 2
Value of currentValueSubject: 2
currentValueSubjectDemo() 3
Value of currentValueSubject: 3

# Subject 作为订阅者
示例代码：

```
final class SubjectsDemo {
    private var cancellable1: AnyCancellable?
    private var cancellable2: AnyCancellable?
    private let optionalCurrentValueSubject = CurrentValueSubject<Int?, Never>(nil)
    private func subjectSubscriber() {
        cancellable1 = optionalCurrentValueSubject
            .sink {
                print(#function, $0)
            }
        cancellable2 = [1, 2, 3].publisher
            .subscribe(optionalCurrentValueSubject)
    }
}
```
optionalCurrentValueSubject 可以作为一个订阅者去订阅序列发布者 [1, 2, 3].publisher 发送的元素。

输出内容：

subjectSubscriber() nil
subjectSubscriber() Optional(1)
subjectSubscriber() Optional(2)
subjectSubscriber() Optional(3)

# 常见用法
在使用 Subject 时，我们往往不会将其暴露给调用方。这时候，可以使用 eraseToAnyPublisher 操作符来隐藏内部的 Subject。

示例代码：
```
struct Model {
        let id: UUID
        let name: String
    }
    final class ViewModel {
        private let modelSubject = CurrentValueSubject<Model?, Never>(nil)
        var modelPublisher: AnyPublisher<Model?, Never> {
            return modelSubject.eraseToAnyPublisher()
        }
        func updateName(_ name: String) {
            modelSubject.send(.init(id: UUID(), name: name))
        }
    }
```
外部调用者无法直接操控 ViewModel 内部的 Subject，这样可以让 ViewModel 更好地面对将来可能的改动。
外部调用者只需要知道 modelPublisher 是 AnyPublisher<Model?, Never> 类型的发布者即可，无论内部采用了 CurrentValueSubject 还是 PassthroughSubject 甚至是其他的发布者。
总结

 

相比于其他的发布者来说， Subject 是比较容易理解的，而且也是最常用到的。
只可惜，对比 Rx 提供的 Subject，Combine 中的 Subject 无法设置缓冲的大小。也许某天苹果会对此做出调整吧~