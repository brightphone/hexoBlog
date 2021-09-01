
---
layout: post
title: "Swift Code"
date: 2021-02-25 12:00:00
comments: true
catagories: language
tags: [Swift]
---




# KeyPath

Key paths实质上可以让我们将任意一个属性当做一个独立的值。因此，它们可以传递，可以在表达式中使用，启用一段代码去获取或者设置一个属性，而不用确切地知道它们使用哪个属性。

<!--more-->

Key paths主要有三种变体:

KeyPath: 提供对属性的只读访问
WritableKeyPath: 提供对具有价值语义的可变属性提供可读可写的访问
ReferenceWritableKeyPath: 只能对引用类型使用(比如一个类的实例), 对任意可变属性提供可读可写的访问

```
class Article {
    let id: UUID
    let source: URL
    title: String
    let body: String
}

let articleIDs = articles.map { $0.id }
let articleSources = articles.map { $0.source }

extension Sequence {
	func map<T>(_ keyPath: KeyPath<Element, T>) -> [T] {
	  return map { $0[keyPath: keyPath] }
	}
}
let articleIDs = articles.map(\.id)
let articleSources = articles.map(\.source)

extension Sequence {
    func sorted<T: Comparable>(by keyPath: KeyPath<Element, T>) -> [Element] {
        return sorted { a, b in
            return a[keyPath: keyPath] < b[keyPath: keyPath]
        }
    }
}

struct CellConfigurator<Model> {
    let titleKeyPath: KeyPath<Model, String>
    let subtitleKeyPath: KeyPath<Model, String>
    let imageKeyPath: KeyPath<Model, UIImage?>
 
    func configure(_ cell: UITableViewCell, for model: Model) {
        cell.textLabel?.text = model[keyPath: titleKeyPath]
        cell.detailTextLabel?.text = model[keyPath: subtitleKeyPath]
        cell.imageView?.image = model[keyPath: imageKeyPath]
    }
}

func print<Object: AnyObject, Value>(for object: Object, keyPath: KeyPath<Object, Value>) {
    print(object[keyPath: keyPath])
}

```

```
    func childDidFinish(_ child: Coordinator?) {
        for (index, coordinator) in childCoordinators.enumerated() {
            if coordinator === child {
                childCoordinators.remove(at: index)
                break
            }
        }
    }
```
```
let trickNamePublisher = NotificationCenter.Publisher(center: .default, name: .newTrickDownloaded)
    .map{ notification -> Data in
        let userInfo = notification.userInfo
        return userInfo?["data"] as! Data
    }
    .decode(type: MagicTrick.self, decoder: JSONDecoder())
```

# swift AnyObject

