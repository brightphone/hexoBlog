---
layout: post
title: "Combine 中的 Scheduler"
date: 2021-03-01 12:00:00
comments: true
catagories: swift
tags: [iOS][Combine][Swift]
---

# receive(on:options:)
receive(on:options:) 操作符可以指定下游在指定的调度器上接收发布者发布的元素。其中，on 标签处需要传递的 scheduler 参数是 Scheduler 协议类型，发布者会使用这个 scheduler 向下游发送元素。
<!--more-->

苹果已经提供了很多预定义的 Scheduler，如：RunLoop, DispatchQueue 和 OperationQueue，常用的 Scheduler 有：RunLoop.main, DispatchQueue.main, DispatchQueue.global(), OperationQueue.main 等
```
 let queue = DispatchQueue(label: "my_queue")
        let queueKey = DispatchSpecificKey<String>()
        queue.setSpecific(key: queueKey, value: queue.label)
        DispatchQueue.main.setSpecific(key: queueKey, value: DispatchQueue.main.label)
        queue.async {
            Future<Int, Never> { promise in
                print("Future queue:", DispatchQueue.getSpecific(key: queueKey) ?? "")
                promise(.success(1))
            }
            .map { value -> Int in
                print("map queue:", DispatchQueue.getSpecific(key: queueKey) ?? "")
                return value
            }
            .receive(on: DispatchQueue.main)
            .sink { value in
                print("sink queue:", DispatchQueue.getSpecific(key: queueKey) ?? "")
            }
            .store(in: &self.cancellables)
        }

输出内容：

Future queue: my_queue
map queue: my_queue
sink queue: com.apple.main-thread

如果去掉 .receive(on: queue) 这一行代码，输出内容将变为：

Future queue: my_queue
map queue: my_queue
sink queue: my_queue
```
根据输出内容可以看出，receive(on: ) 确实可以指定执行下游代码的线程。

 

receive(on:options:) 最常见的使用场景：

在后台线程执行耗时任务，完成后将结果汇总到 UI 主线程上。
```
Future<Int, Never> { promise in
        DispatchQueue.global().async {
            // 耗时的后台任务
            // ...
            let result = 1
            promise(.success(result))
        }
    }
    .receive(on: DispatchQueue.main) // 切换到主线程
    .sink { value in
        // 在主线程处理耗时任务得到的结果
    }
    .store(in: &self.cancellables)
```

# subscribe(on:options:)

subscribe(on:options:) 操作符可以指定上游执行订阅、取消和请求操作的线程。

其中，on 标签处需要传递的 scheduler 参数是 Scheduler 协议类型，下游会使用这个 scheduler 向上游发送消息。

```
let queue = DispatchQueue(label: "my_queue")
        let queueKey = DispatchSpecificKey<String>()
        queue.setSpecific(key: queueKey, value: queue.label)
        DispatchQueue.main.setSpecific(key: queueKey, value: DispatchQueue.main.label)
        queue.async {
            Future<Int, Never> { promise in
                print("Future queue:", DispatchQueue.getSpecific(key: queueKey) ?? "")
                promise(.success(1))
            }
            .map { value -> Int in
                print("map queue:", DispatchQueue.getSpecific(key: queueKey) ?? "")
                return value
            }
            .subscribe(on: DispatchQueue.main) // 对比上面的例子，只增加了这一行代码
            .receive(on: DispatchQueue.main)
            .sink { value in
                print("sink queue:", DispatchQueue.getSpecific(key: queueKey) ?? "")
            }
            .store(in: &self.cancellables)
        }
输出内容:
Future queue: my_queue
map queue: com.apple.main-thread
sink queue: com.apple.main-thread

如果没有 .subscribe(on: DispatchQueue.main) 这一行代码，输出内容为：

Future queue: my_queue
map queue: my_queue
sink queue: com.apple.main-thread
```
根据输出内容可以看出，subscribe(on: ) 确实改变了 map 中运行的线程，但是却没有改变 Future 中运行的线程！为什么？？？

让我们观察另一个示例：

```
 let queue = DispatchQueue(label: "my_queue")
        let queueKey = DispatchSpecificKey<String>()
        queue.setSpecific(key: queueKey, value: queue.label)
        DispatchQueue.main.setSpecific(key: queueKey, value: DispatchQueue.main.label)
        queue.async {
            Deferred<Future<Int, Never>> {
                print("Deferred queue:", DispatchQueue.getSpecific(key: queueKey) ?? "")
                return Future<Int, Never> { promise in
                    print("Future queue:", DispatchQueue.getSpecific(key: queueKey) ?? "")
                    promise(.success(1))
                }
            }
            .map { value -> Int in
                print("map queue:", DispatchQueue.getSpecific(key: queueKey) ?? "")
                return value
            }
            .subscribe(on: DispatchQueue.main)
            .receive(on: DispatchQueue.main)
            .sink { value in
                print("sink queue:", DispatchQueue.getSpecific(key: queueKey) ?? "")
            }
            .store(in: &self.cancellables)
        }

输出内容：
Deferred queue: com.apple.main-thread
Future queue: com.apple.main-thread
map queue: com.apple.main-thread
sink queue: com.apple.main-thread
```
咦，全部都是 com.apple.main-thread，也就是说 subscribe(on: ) 改变了 Deferred, Future 和 map 中运行的线程！为什么？

因为，Future 会在创建时就执行传入的闭包，而 Deferred 会等到订阅者发送需求时才执行传入的闭包。

创建 Future 的操作是在 queue 中进行的，所以此时执行传入的闭包的线程就是 queue 提供的线程。

Deferred 在下游订阅者发送需求时就已经被指定了 subscribe(on: DispatchQueue.main)，所以创建 Deferred 时传入的闭包会在 DispatchQueue.main 提供的线程中执行。