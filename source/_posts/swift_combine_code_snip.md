---
layout: post
title: "Combine的核心概念"
date: 2021-03-16 12:00:00
comments: true
catagories: swift
tags: [iOS][Combine][Swift]
---

Combine Code sample
<!--more-->

```
    @Published var isSubmitAllowed: Bool = true {
        didSet {
            print("isSubmitAllowed was set to: \(isSubmitAllowed)")
        }
    }
    

let blogPostPublisher = NotificationCenter.Publisher(center: .default, name: .newBlogPost,
object: nil)
    .map { (notification) -> String? in
        return (notification.object as? BlogPost)?.title ?? ""
    }
print("test")
let lastPostLabel = UILabel()
let lastPostLabelSubscriber = Subscribers.Assign(object: lastPostLabel, keyPath: \.text)
blogPostPublisher.subscribe(lastPostLabelSubscriber)
let blogPost = BlogPost(title: "Getting started with the Combine framework in Swift", url:
URL(string: "https://www.avanderlee.com/swift/combine/")!)
NotificationCenter.default.post(name: .newBlogPost, object: blogPost)
print("Last post is: \(lastPostLabel.text!)")


let queue = DispatchQueue(label: "my_queue")
let queueKey = DispatchSpecificKey<String>()
queue.setSpecific(key: queueKey, value: queue.label)
DispatchQueue.main.setSpecific(key: queueKey, value: DispatchQueue.main.label)
queue.async {
    Future<Int, Never> { (promise: (Result<Int, Never>) -> Void) -> Void in
        print("Future queue:",DispatchQueue.getSpecific(key: queueKey) ?? "")
        promise(.success(1))
    }.map { (value) -> Int in
        print("map queue:", DispatchQueue.getSpecific(key: queueKey) ?? "")
        return value
    }.receive(on: DispatchQueue.main)
    .sink { value in
        print("sink queue:", DispatchQueue.getSpecific(key: queueKey) ?? "")
    }
    .store(in: &self.cancellables)
}
```
在􏰑􏰒写一个􏱁议􏴕􏳨􏱢加􏱔
性或方法时，如果它们不是􏱁议所需的􏱔性或方法的􏲌认实现，那就得小心了。如果符合􏱁议的 类型也实现了这些􏱔性或方法，那么运行时行为可能就不是你所􏵺期的
```
protocol Storyboarded {
    static func instantiate() -> Self
}

extension Storyboarded where Self: UIViewController {
    static func instantiate() -> Self {
        let fullName = NSStringFromClass(self)
        let className = fullName.components(separatedBy: ".").last
        
        let storyboard = UIStoryboard(name: "Main", bundle: Bundle.main)
        
        return storyboard.instantiateViewController(identifier: className!) as! Self
        
    }
}
```
# for

```
for i in 0 ..< dataSource.numberOfColumns {
    let columnLabel = dataSource.label(forColumn: i)
    let columnHeader = " \(columnLabel) |"
    firstRow += columnHeader
    columnWidths.append(columnLabel.characters.count)
}
```