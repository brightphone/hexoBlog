---
layout: post
title: "Combine 提供的发布者(Publishers)"
date: 2021-03-01 12:00:00
comments: true
catagories: swift
tags: [iOS][Combine][Swift]
---

正所谓，工欲善其事，必先利其器。在开始使用 Combine 进行响应式编程之前，建议您先了解 Combine 为您提供的各种发布者(Publishers)、操作符(Operators)、订阅者(Subscribers)。合理地选择符合需求的 Combine 发布者，可以大幅度地提升您的开发效率！
[参考这里](https://blog.ficowshen.com/page/post/21)
<!--more-->
这些都是 Combine 为我们提供的发布者：
Just，Future，Deferred，Empty，Publishers.Sequence，Fail，Record，Share，Multicast，ObservableObject，@Published。

接下来的几分钟，让我们把它们各个击破！
请注意，后续内容中出现的 cancellables 全部由这个类的实例提供 ：
```
final class CombinePublishersDemo {
    private var cancellables = Set<AnyCancellable>()
}
```
# Just
Just 向每个订阅者只发送单个值，然后结束。它的失败类型为 Never，也就是不能失败。 示例代码：

```
func just() {
        Just(1) // 直接发送1
            .sink { value in
                // 输出：just() 1
                print(#function, value)
            }
            .store(in: &cancellables)
}

输出内容：

just() 1

```
Just 常被用在错误处理中，在捕获异常后发送备用值。 示例代码：

```
func just2() {
        // 使用 Fail 发送失败
        Fail(error: NSError(domain: "", code: 0, userInfo: nil))
            .catch { _ in
                // 捕获错误，返回 Just(3)
                return Just(3)
            }
            .sink { value in
                // 输出：just2() 3
                print(#function, value)
            }
            .store(in: &cancellables)
    }

输出内容：

just2() 3
```

# Future
Future 使用一个闭包来进行初始化，最终这个闭包将执行传入的一个闭包参数(promise)来发送单个值或者失败。请不要使用 Future 发送多个值。PassthroughSubject, CurrentValueSubject 或者 Deferred 会是更好的选择。

请注意，Future 不会等待订阅者发送需求，它会在被创建时就立刻异步执行这个初始化时传入的闭包！如果你需要等待订阅者发送需求时才执行这个闭包，请使用 Deferred。如果你需要重复执行这个闭包，也请使用 Deferred。

示例代码：
```
func future() {
        Future<Int, Never> { promise in
            // 延时1秒
            DispatchQueue.global().asyncAfter(deadline: .now() + 1) {
                promise(.success(2))
            }
        }
        .sink { value in
            // 输出：future() 2
            print(#function, value)
        }
        .store(in: &cancellables)
}
输出内容：

future() 2
```
更常见的用法是将 Future 作为一个任务函数的返回值，让具体任务的执行代码与订阅代码分离：
```
private func bigTask() -> Future<Int, Error> {
        return Future() { promise in
            // 模拟耗时操作
            sleep(1)
            guard Bool.random() else {
                promise(.failure(NSError(domain: "com.ficowshen.blog", code: -1, userInfo: [NSLocalizedDescriptionKey: "task failed"])))
                return
            }
            promise(.success(3))
        }
    }
func future2() {
        bigTask()
            .subscribe(on: DispatchQueue.global())
            .receive(on: DispatchQueue.main)
            .sink(receiveCompletion: { completion in
                switch completion {
                case .finished:
                    // 输出：future2() finished
                    print(#function, "finished")
                case .failure(let error):
                    // 输出：future2() Error Domain=com.ficowshen.blog Code=-1 "task failed" UserInfo={NSLocalizedDescription=task failed}
                    print(#function, error)
                }
            }, receiveValue: { value in
                // 输出：future2() 3
                print(#function, value)
            })
            .store(in: &cancellables)
    }

```
输出内容由 Bool.random() 决定，可能是：

future2() Error Domain=com.ficowshen.blog Code=-1 “task failed” UserInfo={NSLocalizedDescription=task failed}
也可能是：

future2() 3
future2() finished

# Deferred
Deferred 使用一个生成发布者的闭包来完成初始化，这个闭包会在订阅者执行订阅操作时才执行。

示例代码
```
func deferred() {
        let deferredPublisher = Deferred<AnyPublisher<Bool, Error>> {
            // 在订阅之后才会执行
            print(Date(), "Future inside Deferred created")
            return Future<Bool, Error> { promise in
                promise(.success(true))
            }.eraseToAnyPublisher()
        }.eraseToAnyPublisher()
        print(Date(), "Deferred created")
        DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
            // 延迟1秒后进行订阅
            deferredPublisher
                .sink(receiveCompletion: { completion in
                    print(Date(), "Deferred receiveCompletion:", completion)
                }, receiveValue: { value in
                    print(Date(), "Deferred receiveValue:", value)
                })
                .store(in: &self.cancellables)
        }
    }
```
执行上面的函数，输出内容如下(请注意观察输出的时间，延时1秒)：

2020-09-08 23:44:35 +0000 Deferred created
2020-09-08 23:44:36 +0000 Future inside Deferred created
2020-09-08 23:44:36 +0000 Deferred receiveValue: true
2020-09-08 23:44:36 +0000 Deferred receiveCompletion: finished

# Empty

Empty 是一个从不发布任何值的发布者，可以选择立即完成(Empty() 或者 Empty(completeImmediately: true))。

可以使用 Empty(completeImmediately: false) 创建一个从不发布者（一个从不发送值，也从不完成或失败的发布者）。
Empty 常用于错误处理。当错误发生时，如果你不想发送错误，可以用 Empty 来发送完成。
```
func empty() {
        Empty<Never, Error>() // 或者 Empty<Never, Error>(completeImmediately: true)
            .sink(receiveCompletion: { completion in
                // 输出：empty() finished
                print(#function, completion)
            }, receiveValue: { _ in
            })
            .store(in: &self.cancellables)
    }

输出内容：

empty() finished
```
# Publishers.Sequence

Publishers.Sequence 是发送一个元素序列的发布者，元素发送完毕时会自动发送完成。

```
func sequence() {
        [1, 2, 3].publisher
            .sink(receiveCompletion: { completion in
                // 输出：sequence() finished
                print(#function, completion)
            }, receiveValue: { value in
                // 输出：sequence() 1
                // 输出：sequence() 2
                // 输出：sequence() 3
                print(#function, value)
            })
            .store(in: &self.cancellables)
    }
```
输出内容：

sequence() 1
sequence() 2
sequence() 3
sequence() finished

# Fail
Fail 是一个以指定的错误终止序列的发布者。通常用于返回错误，比如：在校验参数缺失或错误等场景中，返回一个 Fail。

```
func fail() {
        Fail<Never, NSError>(error: NSError(domain: "", code: 0, userInfo: nil))
            .sink(receiveCompletion: { completion in
                // 输出：fail() failure(Error Domain= Code=0 "(null)")
                print(#function, completion)
            }, receiveValue: { _ in
            })
            .store(in: &cancellables)
    }
```

# Record
Record 发布者允许录制一系列的输入和一个完成，录制之后再发送给每一个订阅者。

```
func record() {
        Record<Int, Never> { record in
            record.receive(1)
            record.receive(2)
            record.receive(3)
            record.receive(completion: .finished)
        }
        .sink(receiveCompletion: { completion in
            // 输出：record() finished
            print(#function, completion)
        }, receiveValue: { value in
            // 输出：record() 1
            // 输出：record() 2
            // 输出：record() 3
            print(#function, value)
        })
        .store(in: &cancellables)
    }
```
输出内容：

record() 1
record() 2
record() 3
record() finished

# Share
Share 发布者可以和多个订阅者共享上游发布者的输出。请注意，它和其他值类型的发布者不一样，这是一个引用类型的发布者！

当您需要使用引用语义的发布者时，可以考虑使用这个类型。

为了更好地理解 Share 的意义和用途, 让我们先来观察没有 Share 会出现什么问题：
```
func withoutShare() {
        let deferred = Deferred<Future<Int, Never>> {
            print("creating Future")
            return Future<Int, Never> { promise in
                print("promise(.success(1))")
                promise(.success(1))
            }
        }
        deferred
            .print("1_")
            .sink(receiveCompletion: { completion in
                print("receiveCompletion1", completion)
            }, receiveValue: { value in
                print("receiveValue1", value)
            })
            .store(in: &cancellables)
        deferred
            .print("2_")
            .sink(receiveCompletion: { completion in
                print("receiveCompletion2", completion)
            }, receiveValue: { value in
                print("receiveValue2", value)
            })
            .store(in: &cancellables)
    }
```
输出内容：

creating Future
promise(.success(1))
1: receive subscription: (Future)
1: request unlimited
1: receive value: (1)
receiveValue1 1
1: receive finished
receiveCompletion1 finished
creating Future
promise(.success(1))
2: receive subscription: (Future)
2: request unlimited
2: receive value: (1)
receiveValue2 1
2: receive finished
receiveCompletion2 finished
通过观察输出内容，我们可以发现 Deferred 和 Future 部分的代码执行了两次！

接下来，我们使用 Share 来尝试解决这个问题：
```
func withShare() {
        let deferred = Deferred<Future<Int, Never>> {
            print("creating Future")
            return Future<Int, Never> { promise in
                print("promise(.success(1))")
                promise(.success(1))
            }
        }
        let sharedPublisher = deferred
            .print("0_")
            .share()
        sharedPublisher
            .print("1_")
            .sink(receiveCompletion: { completion in
                print("receiveCompletion1", completion)
            }, receiveValue: { value in
                print("receiveValue1", value)
            })
            .store(in: &cancellables)
        sharedPublisher
            .print("2_")
            .sink(receiveCompletion: { completion in
                print("receiveCompletion2", completion)
            }, receiveValue: { value in
                print("receiveValue2", value)
            })
            .store(in: &cancellables)
    }
```
输出内容：

1: receive subscription: (Multicast)
1: request unlimited
creating Future
promise(.success(1))
0: receive subscription: (Future)
0: request unlimited
0: receive value: (1)
1: receive value: (1)
receiveValue1 1
0: receive finished
1: receive finished
receiveCompletion1 finished
2: receive subscription: (Multicast)
2: request unlimited
2_: receive finished
receiveCompletion2 finished
咦，Deferred 和 Future 部分执行了两次的问题解决了，但是出现了另一个问题！第二个订阅者没有收到值，只收到了完成！！？？

而且，仔细观察输出的内容，Multicast 十分引人注目！

原来，根据官方文档的解释，Share 其实是 Multicast 发布者和 PassthroughSubject 发布者的结合，而且它会隐式调用 autoconnect()。
也就是说，在订阅操作发生后，Share 就会开始发送内容。这样也就导致了后续的订阅者无法收到之前就已经发布的值。

怎么解决这个问题？

回顾 Combine 框架，从0到1 —— 2.通过 ConnectablePublisher 控制何时发布 的内容，我们可以通过自行调用 connect() 来解决这个问题。

这是调整后的代码：
```
func withShareAndConnectable() {
        let deferred = Deferred<Future<Int, Never>> {
            print("creating Future")
            return Future<Int, Never> { promise in
                print("promise(.success(1))")
                promise(.success(1))
            }
        }
        let sharedPublisher = deferred
            .print("0_")
            .share()
            .makeConnectable() // 自行决定发布者何时开始发送订阅元素给订阅者
        sharedPublisher
            .print("1_")
            .sink(receiveCompletion: { completion in
                print("receiveCompletion1", completion)
            }, receiveValue: { value in
                print("receiveValue1", value)
            })
            .store(in: &cancellables)
        sharedPublisher
            .print("2_")
            .sink(receiveCompletion: { completion in
                print("receiveCompletion2", completion)
            }, receiveValue: { value in
                print("receiveValue2", value)
            })
            .store(in: &cancellables)
        sharedPublisher
            .connect() // 让发布者开始发送内容
            .store(in: &cancellables)
    }
```
只需要在 share() 之后调用 makeConnectable()，我们即可夺回控制权！在所有订阅者准备就绪之后，通过调用 connect() 让发布者开始发送内容。

输出内容：

1: receive subscription: (Multicast)
1: request unlimited
2: receive subscription: (Multicast)
2: request unlimited
creating Future
promise(.success(1))
0: receive subscription: (Future)
0: request unlimited
0: receive value: (1)
1: receive value: (1)
receiveValue1 1
2: receive value: (1)
receiveValue2 1
0: receive finished
1: receive finished
receiveCompletion1 finished
2: receive finished
receiveCompletion2 finished
现在，Deferred 和 Future 部分的代码只执行一次，两个订阅者也都收到了值和完成。

除此之外，我们也可以使用 Multicast 解决这个问题。

# Multicast
Multicast 发布者使用一个 Subject 向多个订阅者发送元素。和 Share 一样，这也是一个引用类型的发布者。在使用多个订阅者进行订阅时，它们可以有效地保证上游发布者不重复执行繁重的耗时操作。

而且 Multicast 是一个 ConnectablePublisher，所以我们需要在订阅者准备就绪之后去手动调用 connect() 方法，然后订阅者才能收到上游发布者发送的元素。

示例代码：
```
func multicast() {
        let multicastSubject = PassthroughSubject<Int, Never>()
        let deferred = Deferred<Future<Int, Never>> {
            print("creating Future")
            return Future<Int, Never> { promise in
                print("promise(.success(1))")
                promise(.success(1))
            }
        }
        let sharedPublisher = deferred
            .print("0_")
            .multicast(subject: multicastSubject)
        sharedPublisher
            .print("1_")
            .sink(receiveCompletion: { completion in
                print("receiveCompletion1", completion)
            }, receiveValue: { value in
                print("receiveValue1", value)
            })
            .store(in: &cancellables)
        sharedPublisher
            .print("2_")
            .sink(receiveCompletion: { completion in
                print("receiveCompletion2", completion)
            }, receiveValue: { value in
                print("receiveValue2", value)
            })
            .store(in: &cancellables)
        sharedPublisher
            .connect()
            .store(in: &cancellables)
    }

```
输出内容：

1: receive subscription: (Multicast)
1: request unlimited
2: receive subscription: (Multicast)
2: request unlimited
creating Future
promise(.success(1))
0: receive subscription: (Future)
0: request unlimited
0: receive value: (1)
1: receive value: (1)
receiveValue1 1
2: receive value: (1)
receiveValue2 1
0: receive finished
1: receive finished
receiveCompletion1 finished
2: receive finished
receiveCompletion2 finished

# ObservableObject
ObservableObject 是具有发布者的一种对象，该对象在更改对象之前发出变动元素。常用在 SwiftUI 中。

遵循 ObservableObject 协议的对象会自动生成一个 objectWillChange 发布者，这个发布者会在这个对象的 @Published 属性发生变动时发送 变动之前的旧值。

来自官网文档的示例代码：
```
class Contact: ObservableObject {
    @Published var name: String
    @Published var age: Int
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
    func haveBirthday() -> Int {
        age += 1
        return age
    }
}
let john = Contact(name: "John Appleseed", age: 24)
john.objectWillChange
    .sink { _ in
        print("\(john.age) will change")
    }
    .store(in: &cancellables)
print(john.haveBirthday())
```
输出内容：

24 will change
25

# @Published
@Published 是一个属性包装器(@propertyWrapper)，它可以为任何属性添加一个 Combine 发布者。常用在 SwiftUI 中。

请注意，@Published 发布的是属性观察器 willSet 中接收到的新值，但是这个属性当前的值还是旧值！观察下面的例子，可以帮助您理解这个重点。

来自官网文档的示例代码：
```
class Weather {
    @Published var temperature: Double
    init(temperature: Double) {
        self.temperature = temperature
    }
}
func published() {
        let weather = Weather(temperature: 20)
        weather
            .$temperature // 请注意这里的 $ 符号，通过 $ 操作符来访问发布者
            .sink() { value in
                print("Temperature before: \(weather.temperature)") // 属性中的值尚未改变
                print("Temperature now: \(value)") // 发布者发布的是新值
            }
            .store(in: &cancellables)
        weather.temperature = 25 // 请注意这里没有 $ 符号，访问的是被属性包装器包装起来的值
    }
```
输出内容：

Temperature before: 20.0
Temperature now: 20.0
Temperature before: 20.0
Temperature now: 25.0
当 sink 中收到新值 25.0 时，weather.temperature 的值依然为 20.0。
总结

 

感谢 Combine 为我们提供了这些发布者：
Just，Future，Deferred，Empty，Publishers.Sequence，Fail，Record，Share，Multicast，ObservableObject，@Published

虽然看起来有很多不同的发布者，而且使用起来也有颇多的注意事项，但是这些发布者无疑是大幅度地提升了我们进行响应式编程的效率。

如果将 Combine 与 SwiftUI 结合在一起，我们就可以充分地享受声明式编程带来的易读、便利、高效以及优雅。
不过，这就需要我们充分掌握 Combine 和 SwiftUI 中的基础知识和重难点。否则，一定会有很多坑在等着我们~

最后，除了这些普通的 Publishers，Combine 还为我们提供了特殊的发布者 —— Subjects。
