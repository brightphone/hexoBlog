---
layout: post
title: "iOS - Coordinators Pattern"
date: 2021-03-16 12:00:00
comments: true
catagories: iOS
tags: [iOS][Design Pattern]
---

一般情况下push或present到一个新的vc页面，都是在现有的vc页面创建view controller，设置相应的传递数据，然后进行push或present操作
<!--more-->

# Coordinators模式

```
var viewController = BaseViewController()
viewController.data = data
self.navigationController?.pushViewController(viewController, animated: true)       
```
而且这样形式的代码，项目中随处可见，特别是当一个vc事件多的时候，大量的跳转操作，如下所示
```
func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
     guard let type = ContentType(rawValue: indexPath.row) else { return }
     var vc: UIViewController!
     switch type {
     case .tips:
         vc = TipsViewController()
     case .swiftLanguage:
         vc = SwiftLanguageViewController()
     case .newKnowledge:
         vc = NewKnowledgeViewController()
     case .foundation:
         vc = FoundationViewController()
     case .arichtecture:
         vc = ArichtectureViewController()
     case .notification:
         vc = NotificationViewController()
     case .desginPattern:
         vc = DesignPatternViewController()
     case .thread:
         vc = ThreadViewController()
     case .network:
         vc = NetWorkViewController()
     case .framework:
         vc = ThirdFrameworkViewController()
     case .UIkit:
         vc = UIKitViewController()
     case .dataCache:
         vc = DataCacheViewController()
    }
     self.navigationController?.pushViewController(vc, animated: true)
}
```
这里还比较简单，实际情况更加复杂，因为涉及到值的传递和回调。这会带来一些问题

a.我们必须硬编码实现一个viewController到另一个viewController的跳转
b.而且可能会存在重复创建viewController进行跳转的代码，因为一个viewController页面有可能存在多个地方要进入通一个viewController
c.如果需要在不同的设备使用，那么需要添加一些确认逻辑

Coordinator模式能够帮助我们解决这些问题，把跳转逻辑剥离出来，也不需要进行硬编码操作，而且能够让view controllers进行解耦合，更容易重用
什么是Coordinators?
A Coordinator is an object the encapsulates a lifecycle that is spread over a collection of view controllers.

So what is a coordinator? A coordinator is an object that bosses one or more view controllers around. Taking all of the driving logic out of your view controllers, and moving that stuff one layer up is gonna make your life a lot more awesome.

Soroush Khanlou (Coordinators Redux)[https://links.jianshu.com/go?to=http%3A%2F%2Fkhanlou.com%2F2015%2F10%2Fcoordinators-redux%2F]
Coordinator模式简单来说就是分离所有的navigation操作【push/present】到自定义类，由Coordinator控制页面的流向，在自定义类中实现页面之间的切换【当前vc和子vc之间或者子vc和子vc之间的跳转】、包括事件【按钮点击、cell的点击】的处理。每一个功能模块都可以定义一个Coordinator类来处理事件。

比如下图：
![image](/res/images/article/coordinator/coordinator1.png)

A, B, C 3个vc在Coordinator中进行使用，每一个vc都由自己的代理(delegate)来处理交互事件，然后由Coordinator来处理这些事件，执行具体的跳转

Coordinator优点

Each view controller is now isolated.
View controllers are now reusable.
Every task and sub-task in your app now has a dedicated way of being encapsulated.
Coordinators separate display-binding from side effects.
Coordinators are objects fully in your control.
(Souroush Khanlou: Coordinators Redux)[https://khanlou.com/2015/10/coordinators-redux/]

# 2、使用例子
使用Xcode创建一个新工程，创建Coordinator文件，声明Coordinator协议

```
import UIKit

protocol Coordinator {
    // 存储所有的子Coordinators
    var childCoordinators: [Coordinator] { get set }
    // 存储导航控制器
    var navigationController: UINavigationController { get set }
    // 启动方法
    func start()
}
```
创建一个Storyboarded文件，定义Storyboarded初始化协议方便获取ViewController，后面会通过Storyboarded布局并加载ViewController
```
import UIKit

// 定义Storyboarded初始化协议
protocol Storyboarded {
    static func instantiate() -> Self
}

// 通过Storyboard获取vc
extension Storyboarded where Self: UIViewController {
    static func instantiate() -> Self {
        
        let fullName = NSStringFromClass(self)

        // this splits by the dot and uses everything after, giving "MyViewController"
        let className = fullName.components(separatedBy: ".")[1]
        
        // load our storyboard
        let storyboard = UIStoryboard(name: "Main", bundle: Bundle.main)
        
        // instantiate a view controller with that identifier, and force cast as the type that was requested
        return storyboard.instantiateViewController(withIdentifier: className) as! Self
    }
}
```
使ViewController实现Storyboarded的协议

```
import UIKit

class ViewController: UIViewController, Storyboarded {

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
    }


}
```
打开Main.storyboard进行页面布局，设置ViewController的 storyboard identifier为ViewController，方便获取ViewController，取消its Initial View Controller

创建MainCoordinator文件，用于启动app


```
import UIKit

// 控制app整体的流向，实现Coordinator协议
class MainCoordinator: Coordinator {
    var childCoordinators = [Coordinator]()
    var navigationController: UINavigationController

    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }

    func start() {
        // 获取主vc
        let vc = ViewController.instantiate()
        navigationController.pushViewController(vc, animated: false)
    }
}
```
修改AppDelegate文件

```
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    var coordinator: MainCoordinator?
    var window: UIWindow?

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        // Override point for customization after application launch.

        // create the main navigation controller to be used for our app
        let navController = UINavigationController()

        // send that into our coordinator so that it can display view controllers
        coordinator = MainCoordinator(navigationController: navController)

        // tell the coordinator to take over control
        coordinator?.start()

        // create a basic UIWindow and activate it
        window = UIWindow(frame: UIScreen.main.bounds)
        window?.rootViewController = navController
        window?.makeKeyAndVisible()

        return true
    }
 ....
}
```
创建一个BuyViewController和CreateAccountViewController,进入Main.storyboard拖2个UIViewController，分别指定storyboard identifier为BuyViewController和CreateAccountViewController，简单为两个vc各个添加一个label标识，然后在ViewController上添加两个按钮，分别为Buyfood和CreateAccount，并且添加添加点击事件
![image](/res/images/article/coordinator/coordinator2.png)


```
import UIKit

class ViewController: UIViewController, Storyboarded {

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
    }

    @IBAction func buyTapped(_ sender: Any) {

    }
    
    @IBAction func createAccount(_ sender: Any) {
        
    }
}
```
接下来在3个vc中添加coordinator属性
```
 weak var coordinator: MainCoordinator?
```
并且让BuyViewController和CreateAccountViewController遵守Storyboarded协议
```
import UIKit

class BuyViewController: UIViewController, Storyboarded {

    weak var coordinator: MainCoordinator?

    override func viewDidLoad() {
        super.viewDidLoad()

        // Do any additional setup after loading the view.
    }

}

import UIKit

class CreateAccountViewController: UIViewController, Storyboarded {

    weak var coordinator: MainCoordinator?

    override func viewDidLoad() {
        super.viewDidLoad()

        // Do any additional setup after loading the view.
    }

}

```
最后实现交互事件，修改MainCoordinator文件


```
import UIKit

// 控制app整体的流向，实现Coordinator协议
class MainCoordinator: Coordinator {
    var childCoordinators = [Coordinator]()
    var navigationController: UINavigationController

    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }

    func start() {
        // 获取主vc
        let vc = ViewController.instantiate()
        vc.coordinator = self // 设置coordinator
        navigationController.pushViewController(vc, animated: false)
    }
}

// 添加交互事件
extension MainCoordinator {
    func buyFood() {
        let vc = BuyViewController.instantiate()
        vc.coordinator = self
        navigationController.pushViewController(vc, animated: true)
    }

    func createAccount() {
        let vc = CreateAccountViewController.instantiate()
        vc.coordinator = self
        navigationController.pushViewController(vc, animated: true)
    }
}
```
在ViewController使用coordinator执行事件

```
class ViewController: UIViewController, Storyboarded {

    weak var coordinator: MainCoordinator?

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
    }

    @IBAction func buyTapped(_ sender: Any) {
        coordinator?.buyFood()
    }
    
    @IBAction func createAccount(_ sender: Any) {
        coordinator?.createAccount()
    }
}
```
代码源于 (How to use the coordinator pattern in iOS apps)[https://www.hackingwithswift.com/articles/71/how-to-use-the-coordinator-pattern-in-ios-apps]，具体内容可以自己阅读

3、coordinator使用的常见问题

How and when do you use child coordinators?
How do you handle moving back from a navigation controller?
How do you pass data between view controllers?
How do you use tab bar controllers with coordinators?
How do you handle segues?
How do you use protocols or closures instead?

具体可参考 (Advanced coordinators in iOS)[https://www.hackingwithswift.com/articles/175/advanced-coordinator-pattern-tutorial-ios]

