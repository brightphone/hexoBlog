---
layout: post
title: "iOS - Coordinators Pattern"
date: 2021-03-16 12:00:00
comments: true
catagories: iOS
tags: [iOS][Design Pattern]
---

Child coordinators, navigating backwards, passing data between view controllers, and more.
<!--more-->

In this article we’ll look at solutions for six common problems people face when adopting the coordinator pattern in iOS apps:

How and when do you use child coordinators?

How do you handle moving back from a navigation controller?

How do you pass data between view controllers?

How do you use tab bar controllers with coordinators?

How do you handle segues?

How do you use protocols or closures instead?

I'll be giving you lots of hands-on code along the way, because I want you to see real-world examples of how these problems are solved.

# How and when to use child coordinators

We’re going to start off by looking at child coordinators. As I explained in the first article, if you have a larger app you can split off functionality into child coordinators: one responsible for creating an account, one responsible for buying a product, and so on.

These all report back to a parent coordinator, which can then continue the flow once the child coordinator has finished. The end result is that we can split up complicated app functionality into smaller, discrete chunks that are easier to work with and easier to re-use.

But the problem is: when do you use these things, and how is it best done?

Let’s try it out with a real project - we’re going to use the same Coordinators project we had at the end of the first video. We already have a buy view controller and a create account view controller, but we’re going to modify it so that buying is handled using a child coordinator.

First, create a new Swift file called BuyCoordinator.swift, and give it this code:

```
class BuyCoordinator: Coordinator {
    var childCoordinators = [Coordinator]()
    var navigationController: UINavigationController

    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }

    func start() {
        // we'll add code here
    }
}
```
For its start method, we need to move some of the existing code from our main coordinator across, because that already creates and shows a BuyViewController.

So, open MainCoordinator.swift and move the contents of buySubscription() over to the start() method above, like this:
```
func start() {
    let vc = BuyViewController.instantiate()
    vc.coordinator = self
    navigationController.pushViewController(vc, animated: true)
}  
```
Leave the buySubscription() method empty in MainCoordinator.swift – we’ll be coming back to it in a moment.

Assigning self to the coordinator property of BuyViewController isn’t possible right now because it expects a MainCoordinator. To fix this, open BuyViewController.swift and change the property to be a BuyCoordinator:
```
weak var coordinator: BuyCoordinator?
```
Back in MainCoordinator, we need to create an instance of the child coordinator and tell it to take over control. This is done by adding three new lines of code to its buySubscription() method:
```
let child = BuyCoordinator(navigationController: navigationController)
childCoordinators.append(child)
child.start()
```
That’s all it takes to create a child coordinator and make it take over control, but a more interesting problem is how we handle removing the child coordinator when we come back. We’ll look at more advanced solutions later on, but first let’s look at a simple solution to get us started.

For simpler apps you could treat your child coordinator array as a stack, pushing and popping things as needed. While that works well enough, I prefer to allow coordinators to be added and removed at any point, giving me more of a tree-like structure than a stack.

To make this work we first need to establish communication between BuyCoordinator and MainCoordinator, so the child can tell the parent when work has finished.

The first step is to add a parentCoordinator property to BuyCoordinator:
```
weak var parentCoordinator: MainCoordinator?
```
That needs to be weak to avoid a retain cycle, because MainCoordinator already owns the child.

We need to set that when our BuyCoordinator is being created, so please add this to the buySubscription() method of MainCoordinator:
```
child.parentCoordinator = self
```
Next, we need a way for BuyViewController to report back when its work has finished. We don’t have any special buttons on there, and we don’t even have any child view controllers that form much of a sequence, so instead we’re going to use this view controller being dismissed as our signal that the buying process has finished.

The easiest way to do this is by implementing viewDidDisappear() in BuyViewController, like this:
```
override func viewDidDisappear(_ animated: Bool) {
    super.viewDidDisappear(animated)
    coordinator?.didFinishBuying()
}
```
We need to add that didFinishBuying() method to BuyCoordinator. How you handle this depends on your application flow: if your main coordinator needs to respond specifically to buying finishing – perhaps to synchronize user data, or cause some UI to refresh – then you might implement a specific method to handle that flow.

In this instance, we’re going to write a general method to handle all child coordinators that don’t need any special behavior.

Add this method to BuyCoordinator now:
```
func didFinishBuying() {
    parentCoordinator?.childDidFinish(self)
}
```
We can then add childDidFinish() to our main coordinator:
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
That uses Swift’s triple equals operator to find the child coordinator in our array. That only works with classes, and right now in theory our coordinators could be used by structs.

Fortunately, coordinators should always be classes because they need to be shared in lots of places, so we can mark our coordinator protocol as being class-only to make all this code work. Modify Coordinator.swift to this:
```
protocol Coordinator: AnyObject {
```
As you can see the trick with child coordinators is to make sure you carve off one discrete chunk of your app for them to handle. This helps you avoid massive coordinators, but also just makes your code easier to follow.

# Navigating backwards
Our current solution for navigating backwards to a previous view controller is fine for simple projects, but what happens when you have multiple view controllers being shown in the child coordinator? Well, viewDidDisappear() will be called prematurely, and your coordinator stack will get confused.

Fortunately, there’s a great solution already written for this by Soroush Khanlou, who developed the coordinator pattern in the first place.

In BuyViewController please comment out the viewDidDisappear() method entirely – we don’t need it any more. And in BuyCoordinator comment out didFinishBuying(), because we don’t need that either.

What we’ll do instead is make our main coordinator detect interactions with the navigation controller directly. First, we need to make it conform to the UINavigationControllerDelegate protocol. That’s only possible if also make it a subclass of NSObject.

Modify the definition of MainCoordinator to this:

