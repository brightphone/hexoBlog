---
layout: post
title: "Swift备忘"
date: 2020-04-24 12:00:00
comments: true
catagories: language
tags: [iOS]
---
# 函数
func funcName(outerName innerName:Type,... )

outerName：是外部参数名，如果省略不写即表示外部参数名与内部参数名一致，如果为“_”表示无外部参数名。
"_":可以使函数被调用时省去外部名，
## 变长参数
变长参数是可以接受零个或更多输入值作为实参，函数只能有一个变长参数，而且一般应该是参数列表中的最后一个。
参数值在函数内部以数组的形式可用.
## 默认参数值
```
func test(par1:Double,par2:Double = 4.0){
    ...
}
```
## in-out 参数
```
func test(_ code:Int,error:inout String) {
    ...
}
var error = "error"
test(400,&error)
```
in-out 参数能让函数修改函数体外的变量，实际上就相当于指针。
in-out 参数不能有默认值，也不可以是变长参数。
调用时要使用取地址符号"&"

<!--more-->
函数的形参不可以再赋值

```
func printName(to names:String...) {
    for name in names {
        print("Hello \(name),Welcome ~~")
    }
}
printName(names:["heliang","Feng"])
```
一个函数本质上是一个对象的实例，因此是可以作为返回值返回；
函数类型：
函数类型是由函数参数和返回值组成，例如：
```
func sortedEvenOddNumbers(_ numbers: [Int]) -> (evens: [Int], odds: [Int]) { 
    var evens = [Int]()
    var odds = [Int]()
    for number in numbers {
        if number%2==0 { 
            evens.append(number)
        } else { 
            odds.append(number)
        } 
    }
    return (evens, odds) 
}
```
这个函数类型可以表示为：
([Int])-([Int],[Int])
函数的参数在左边的圆括号中列出，返回值跟着->后面,可以解读为：
"一个接受整数数组作为参数并返回带有两个整数数组的元组的函数."
即没有参数也没有返回值的函数类型为：() -> ().
我们可以把函数类型实例赋给变量：
let evenOddFunction:([Int])-([Int],[Int]) = sortedEvenOddNumbers
evenOddFunction

# 闭包：
闭包（closure）是在应用中完成特定任务的互相分离的功能组。函数是闭包的特殊情况，可以把函数理解为有名字的闭包。

语法：
```
{
    (parameters) -> return type in
    // code
}
```
关键字in是用来分隔闭包的参数、返回值与闭包体的语句。
```
eg 1:
let array = [1,2,54,53,9]
func sortAscending(_ i:Int,_ j:Int) -> Bool {
    return i < j
}
let sortedArray = array.sorted(by:sortAscending)
```
```
eg 2:内联闭包
let array = [1,2,54,53,9]
let sortedArray = array.sorted(by:{
    (i:Int,j:Int) -> Bool in
    return i < j
})
```

```
eg 3:
let array = [1,2,54,53,9]
let sortedArray = array.sorted(by:{
    (i,j) in i < j
})
```
如果有多个表达式不可省略return
```
eg 4:
let array = [1,2,54,53,9]
let sortedArray = array.sorted(by:{$0 < $1})
```
$number,是利用联快捷参数语法，这样就可以不用声明参数了。
```
eg 5:
let array = [1,2,54,53,9]
let sortedArray = array.sorted{$0 < $1}
```
## 尾部闭包语法
上面是尾部闭包语法，对于闭包体很长的情况特别有用，如果一个闭包是以一个函数的最后一个参数传递的，那么它就可以在函数的圆括号以外内联，因为sorted(by:)只接受一个参数，所以可以根本不需要圆括号，只所以可以省略闭包的参数名，是因为尾部闭包语法允许这么做。

## 闭包是引用类型
这意味着你把函数赋给变量或常量时，实际上是让这个常量或变量指向这个函数。
# 函数式编程
funtional programming
## 一等公民函数
函数可以作为返回值从别的函数返回，也可以作为参数传递给别的函数，可以存储在变量中，等等，就跟其他类型一样。
## 纯函数
函数没有副作用，给定同样的输入，函数永远返回同样的输出，而且不会修改程序中其他地方的状态。
## 不可变性
不鼓励可变性，因为值可变的数据更难分析
## 强类型
强类型系统能增加代码的允许时安全性
## 高阶函数
sorted(by:),map(_:),filter(_:),reduce(_:_:)
高阶函数至少接受一个函数作为输入
```
let curPopulations = [1233,3322,4444]
let mayPopulations = curPopulations.map{
    (population: Int) -> Int in
    return population * 2
}
let bigPopulations = curPopulations.filter {
    (population: Int) -> Int in
    return population > 4000
}

let totalPopulation = curPopulations.reduce(0) {
    (first: Int,second: Int) -> Int in
    return first + second
}
```
map：可以对数组中的每一个值进行修改。
filter:可基于某些条件对数组进行过滤
reduce:可以把一组数组中所有值相加
# enum 枚举
在swift中 不需要使用break跳出当前匹配，默认只执行一个case就结束
一次匹配多个模式
```
enum Weather {
    case rain, snow, wind, sunny
}

switch todayWeather {
case .rain, .snow:
    print("天气不太好，出门要打伞")
case .wind:
    print("刮风")
case .sunny:
    print("晴天")
}
```
```
enum TextAlignment {
case left
case right
case center 
}
```
var alignment = TextAlignment.left

枚举体至少包含一个case语句。
所有的switch语句必须被全覆盖。
对枚举类型建议不使用default分支，这样更面向未来
## 原始值枚举
```
enum TextAlignment: Int {
    case left
    case right
    case center
}
```
Int即为原始值，默认第一个为0，也可以指定值

```
enum TextAlignment: Int {
    case left = 10
    case right = 20
    case center = 30
}
print("Left has raw value \(TextAlignment.left.rawValue)")
let rawValue = 20
let myAlignment = TextAlignment(rawValue:rawValue)

```
Swift 中对enum除了可以用int做原始值，还可以使用所有的内建值类型和字符串。
```
enum Language: String {
    case swift = "Swift"
    case c = "C"
    case java
    case python
}
```
如果是String作为原始值，如果省略了原始值，会使用成员本身的名字作为原始值
enum是值类型，值类型的方法不能对self进行修改。如果需要修改需要在函数前加上mutating
```
mutating fucn tlggle(){

}
```
## 关联值
关联值可以把数据附在枚举实例上，不同成员可以有不同类型的关联值
```
enum ShapeDimensions {
    case oquare(side:Double)
    case rectangle(width:Double,height:Double)

}
var squreShape = ShapeDimensions.square(side: 10.0)
var rectShape = ShapeDimensions.rectangle(width:5.0,height:10.0)
```
要创建shapDimension实例，必须指定成员和相应的关联值。
```
enum ShapeDimension {
    case square(side: Double)
    case rectangle(width: Double,height: Double)
    
    func area() -> Double {
        switch self {
        case let .square(side: side):
            return side * side
        case let .rectangle(width: w, height: d):
            return w * d
        }
    }
}
```
这里的switch分支利用了swift的模式匹配

## 递归枚举
枚举工作原理，Swift编译器必须知道程序中每种类型的每一个实例占据多少内存空间。如果无法知道将要占用多少内存需要使用关键字indirect告诉编译器把枚举数据放到一个指针指向的地方。
```
indirect enum FamilyTree {
    case noKnownParents
    case oneKnownParent(name:String, ancestors:FamilyTree)
    case twoKnownParents(fatherName:String,fatherAncestor:FamilyTree,motherName: String,motherAncestor:FamilyTree)
}
```
swift中一个指针是8个字节
也可以把成员标记为indirect
```
enum FamilyTree {
    case noKnownParents
    indirect case oneKnownParent(name:String, ancestors:FamilyTree)
    indirect case twoKnownParents(fatherName:String,fatherAncestor:FamilyTree,motherName: String,motherAncestor:FamilyTree)
}
```
# switch
swift中的switch功能远比OC强大：
## 无需break ,
而swift的switch语句是满足其中一个条件后，执行完就会停止.
## fallthrough
状态转移语句：如果某一个匹配上的分支末尾有fallthrough，它会把控制权传递给下一个分支，无论它跟正在检验的值是否匹配。
## 一个分支多个值，用逗号分开
```
switch statusCode {
    case 100,101:
    print("1")
    case 300...307:
    print ("2")
    case let unknownCode:
    print("unknown")
}
```
## 区间
switch 语句可以使用valueX...valueY这样的语法来把某个区间的内容值与给定值比较。

## 值绑定
值绑定能在某个特定分支中把待匹配的值绑定到本地常量或变量上，这个常量或变量只能在该分支上使用。

## where子句

## 类型匹配
匹配模式可以应用于类型上，这时我们需要用到两个关键字 is、as （注意：不是as?，尽管它们的机制很相似，但是它们的语义是不同的（“尝试进行类型转换，如果失败就返回 nil” vs “判断这个模式是不是匹配这种类型”））
```
protocol Animal {
    var name: String { get }
}

struct Dog: Animal {
    var name: String {
        return "dog"
    }
    
    var runSpeed: Int
}

struct Bird: Animal {
    var name: String {
        return "bird"
    }
    
    var flightHeight: Int
}

struct Fish: Animal {
    var name: String {
        return "fish"
    }
    
    var depth: Int
}

let animals = [Dog.init(runSpeed: 55), Bird.init(flightHeight: 2000), Fish.init(depth: 100)]

for animal in animals {
    switch animal {
    case let dog as Dog:
        print("\(dog.name) can run \(dog.runSpeed)")
    case let fish as Fish:
        print("\(fish.name) can dive depth \(fish.depth)")
    case is Bird:
        print("bird can fly!")
    default:
        print("unknown animal!")
    }
}
```
## 自定义类型匹配
通常情况下，我们自定的类型是无法进行模式匹配的，也就是不能在 switch/case 语句中使用。如果想要达到可匹配的效果，那么就有必有了解一下匹配操作符 ~=
```
struct BodyFatRate {
    var weight: Float
    var fat: Float
}

let player = BodyFatRate(weight: 180, fat: 30)

func ~=(lhs: Range<Float>, rhs: BodyFatRate) -> Bool {
    return lhs.contains(rhs.fat / rhs.weight)
}

switch player {
case 0.0..<0.15:
    print("难以置信")
case 0.15..<0.2:
    print("健康")
case 0.21..<0.99:
    print("该减肥了")
default:
    break
}
```
注意这里区间是"..<"

## Optional 匹配
当switch传入的值为optional时，如果不想解包，可以使用x?（相当于Optional.some(x)）语法糖来匹配可选值
```
let optionalValue: Int? = 5

switch optionalValue {
case 1?:
    print("it's one")
case 2?:
    print("it's two")
case .none:
    print("it's nil")
default:
    print("it's others")
}
```
上面的代码中，optionalValue相当于 Optional.some(5)，所以也需要同Optional.some(x)进行比较。如果case中的值没有加上 ？则会报错:expression pattern of type 'Int' cannot match values of type 'Int?'。当 optionalValue 为nil时，则与 .none 匹配。在Swift中，Int型被认为是无法穷举的，故必须有default
## switch模式匹配
Swift中的匹配模式要比OC中强大的多。归纳起来大概分为以下几点：
除了可以匹配枚举类型外，支持更多的原生类型匹配，包过Int、String、Float、Tuple、Range等
可以对类型进行匹配，配合 as、is 来使用
重载匹配操作符~=，可以支持自定义类型的匹配操作
可结合where、if、guard、for来使用，使得代码简洁优雅而高效
对Optional类型的匹配支持很友好，可简化部分判断逻辑
### String
当匹配的类型无法穷举时，必须添加 default
```
let name = "Spiderman"

switch name {
case "Ironman":
    print("钢铁侠")
case "Spiderman":
    print("蜘蛛侠")
default: // 由于无法穷举所有字符串，所以必须添加 default 
    print("不认识")
}
```
### Tuple
元组匹配类似于枚举关联值的匹配


```
let point = (x: 10, y: 0)

switch point {
case (0, 0): 
    print("原点")
case (0, _): // 由于不关心 y，所以使用 _ 来进行占位
    print("Y轴p偏移")
case (let x, 0):
    print("X轴偏移：\(x)")
case (let x, let y) where x == y: 
    print("X = Y")
default: 
    break
}
```
# is as as? as!
## is
is 相当于OC中的isKindOfClass

## as
从派生类转换为基类，向上转型，即可以把子类转换为父类;
```
class Amimal{}
class Cat:Animal{}
let cat = Cat()
let animal = cat as Animal
```
除二义性，数值类型转换
```
let age = 28 as Int
let money = 20 as CGFloat
let cost = (50 / 2) as Double
```
switch 语句中进行模式匹配
```
switch person1 {
    case let person1 as Student:
        print("是Student类型，打印学生成绩单...")
    case let person1 as Teacher:
        print("是Teacher类型，打印老师工资单...")
    default: break
}
```

## as!
向下转型
子类可以向上转换为超类，但超类不能向下（downcast）转换为子类。除非某个子类的对象表现形式为超类，但实际是子类，这时可以使用as！进行向下转换（downcast）

# stuct
如果一个结构体的一个实例方法要修改结构体的属性，就必须标记为mutating。结构体和Enum没有继承.
对值类型的变量进行传递时总是会被复制。
要声明类型方法和属性时需要用到static关键字 。

## mutating

# class
## 禁止重写
```
class Zombie:Monster {
    final override func terrorizeTown() {
        ...
    }
}
```
可以用关键字final让方法或属性不可重写
类的类型方法要用class 关键字标记。
如果不想让子类覆盖某个类方法可以加上static关键字或者final class 关键字
类方法可以调用其他类方法或者类属性
swift的实例方法，实际上是返回函数的类级别的方法。
## 禁止继承
用final 标记的类可以防止其他人继承

# 属性
## 存储属性
惰性存储属性必须为var变量。lazy属性只有在第一次访问时才会计算它的值。
存储属性的类属性必须有默认值，因为类型没有初始化方法。


## 属性观察
只用于存储属性
```
var population = 23 {
    didSet(OldPopulation) {
        ...
    }
    willSet(newPopulation) {
        ...
    }
}
```
didSet观察者能让我们问􏵋性的􏶞值。 如果不􏰆定新名字，Swift会自动把􏲺数命名为oldValue,willSet默认名字为newValue
但是在初始化函数中不会走到观察属性的didSet和willSet



## 计算属性
计算属性不会像存储属性那样存储值，而是提供一个读取方法(get)来获取属性的值，并可选地提供一个写入方法（set）设置属性的值。
只读的计算属性是用var定义的。
类级别的计算属性的定义跟方法很像，主要区别是用关键字var不是func，以及没有圆括号。
```
class Zombie: Monster {
    class var spookyNoise: String {
        return "Brains ... "
    }
    ...
}
```
上面是读取方法的快捷语法，如果计算属性没有写入方法，就可以省略计算属性定义中的get，并直接返回所需的计算值。
静态属性和类型属性最大的区别是静态属性无法被子类覆盖。

## 访问控制

访问控制主要围绕模块（比如某个framework）和源代码文件。

### open

### public

### internal
默认访问层级


### fileprivate

### private

![image](/res/images/article/swift/1.png)
如果属性既有存取方法又有写入方法，可以分别控制它们的可见度，默认情况下，读取方法和写入方法的可见度相同。
```
class Zombie: Monster {
    internal private(set) var isFallingApart = false
}
或
class Zombie: Monster {
    private(set) var isFallingApart = false
}
```
只要写入方法的可见度不比读取方法更高就可以。

# 初始化
结构体和类的存储属性在初始化完成的时候需要有初始值。
```
struct CustomType {
    init (someValue:SomeType) {

    }
}
```
用init表示，前面不用有func
结构体可以有默认初始化方法:`空初始化方法`和`成员初始化方法`（对类型的每个存储属性都有相应的外部参数）
如果有自定义初始化方法了，就不会提供默认的初始化方法了。
一旦写了自定义初始化方法，Swift就不会提供默认的初始化方法了
Swift 允许在初始化过程中初始化常量属性。

## 委托初始化
初始化方法的定义中包含对该类型其他初始化方法的调用。为了提供多种创建实例的路径。

## （designated & convenience）初始化方法的概念
指定初始化方法负责确保初始化完成前所有的属性都有值，便捷初始化方法是指定初始化方法的补充，通过调用所在类的指定初始化方法来实现。

类没有默认的成员初始化方法。
初始化方法自动继承
一般来说，类不会继承父类的初始化方法，Swift的这个特性是希望避免子类在不经意间提供无法为所有属性赋值的初始化方法。以下情况类会继承父类的初始化方法：
1.如果子类没有定义任何指定初始化方法。
2.如果子类实现了父类的所有指定初始化方法（无论是通过显示实现还是隐私继承），就会继承所有便捷初始化方法。

## 指定初始化方法
类的主要初始化方法就是指定初始化方法，指定初始化方法的一部分作用是确保类的所有属性在初始化完成前都有值。
**如果类有父类，那么子类的指定初始化方法必须调用父类的指定初始化方法。**
如果在调用指定初始化之前访问了属性的值会报这个错误：Use of 'self' in delegating initializer before self.init is called
在便捷初始化函数中无法给let型常量赋值

## 便捷初始化方法
用关键字 convenience修饰。
便捷初始化必须调用到其所在类的其他初始化方法。

## 必须初始化方法
类可以要求其子类提供特定的初始化方法

```
required init (...) {
    ...
}
```

## 反初始化

```
deinit {

}
```
deinit方法可以访问实例的所有属性和方法
Swift中只有可空类型可以是nil

## 可失败的初始化方法
```
init?(...) {
    ...
}
```

# 值类型和引用类型

值类型内使用引用类型时要务必小心，在引用类型内使用值类型倒不会有什么问题
建议不要在值类型内使用引用类型。如果确实需要在值类型中使用引用类型属性，最好使用不可变实例。

swift没有提供可以执行深复制的方法，如果需要，必须自己编写

## 相等和同一
相等是指两个实例就可见的特性来说具有一样的值。比如同样文本的两个String实例。
同一是指两个变量或常量是否指向内存中的同一个实例。

## stuct 与 class
1.如果类型需要传值，用struct。
2.如果类型不支持子类继承，用struct。
3.如果类型要表达的行为相对比较直观，而且包含一些简单值，那么考虑优先使用struct。
4.其他情况都用class

## 写时复制（COW）

写时复制是指对值类型的底层存储的隐式共享，这种优化能够让某个值类型的多个实例共享同一个底层存储。
如果某个实例需要修改或者写入存储那么这个实例就会产生一份自己的副本。

```
@inlinable public func isKnownUniquelyReferenced<T>(_ object: inout T) -> Bool where T : AnyObject

/// Returns a Boolean value indicating whether the given object is known to
/// have a single strong reference.
```

swift 的容器以及提供类COW支持，你一般不需要自己实现COW类型。


# 协议（protocol）
```
protocol TabularDataSource {
    var numberOfRows: Int { get }
    var numberOfColumns: Int { get }
    func label(forColumn column: Int) -> String
    func itemFor(row: Int, column: Int) -> String 
}
```
{get}表示这些属性可读，如果需要被读写那就用{get set}，协议只可有计算属性和函数

最后，类也可以符合协议。如果类没有父类，语法就跟结构体和枚举一样:
class ClassName: ProtocolOne, ProtocolTwo { // ...
}
如果类有父类，那么父类的名字在前，后跟协议(或者多个协议)。
class ClassName: SuperClass, ProtocolOne, ProtocolTwo { // ...
}
## 协议组合 (&)
``` Swift
func printTable(dataSource: TabularDataSource & CustomStringConvertible) {  
    print("Table: \(dataSource.description)")
    ...
}
```
# 错误处理

可恢复错误和不可恢复错误
throw必须抛出符合Swift.Error协议类型的实例
do/catch
在do中至少有一个try语句
```Swift
do {
    let tokens = try lexer.lex() 
    print("Lexer output: \(tokens)")
} 
catch Lexer.Error.invalidCharacter(let character) { 
    print("Input contained an invalid character: \(character)")
} catch {
    print("An error occurred: \(error)")
}
```
catch 语句支持模式匹配，跟switch语句一样

## try!
关键字末尾的惊叹号是个很强的暗示，如果用强制性的关键字try！则一旦出现错误程序就会触发陷阱
## try?
调用一个可能抛出错误的函数，得到函数原本返回值对应的可空类型返回值，这意味着需要类似guard这样的工具检查可空实例是否有值
```
guard let tokens = try? lexer.lex() else {
    print("Lexing failed, but I don't know why")
    return 
}
```
带throws的函数不会声明自己会抛出什么样的错误。
这样会产生两个实际的影响：
1.不需要修改函数的API就可以随意添加潜在的Error。
2.在用catch处理错误时，必须总是准备好处理未知的错误类型。
# extension
对类型的扩展支持下面几个能力：
1.添加计算属性
2.添加新初始化方法
3.使类型符合协议
4.添加新方法
5.添加嵌入类型


## typealias 
为已有类型起别名
typealias Velocity = Double

swift扩展不允许为类型添加存储属性
## 为结构体添加初始化方法
使用extension可以给结构体添加初始化方法，又不会失去成员初始化方法。
## 嵌套类型和扩展
Swift 扩展还能给已有的类型添加嵌套类型。
``` Swift
extension Car {
    enum Kind {
        case coupe, sedan
    }
    var kind: Kind {
        if numberOfDoors == 2 {
            return .coupe 
        } else {
            return .sedan 
        }
    } 
}
```


# 泛型
``` Swift
struct Stack <Elment>{
    var items = [Elment]()
    
    mutating func push(_ newItem: Elment) {
        items.append(newItem)
    }
    mutating func pop() -> Elment? {
        guard !items.isEmpty else {
            return nil
        }
        return items.removeLast()
    }
    func map<U>(_ f:(Elment)->(U)) -> Stack<U> {
        var mappedItems = [U]()
        for item  in items {
            mappedItems.append(f(item))
        }
        return Stack<U>(items: mappedItems)
    }
}
```
### 泛型函数和方法
```
func myMap<T,U>(_ items:[T], _f:(T) -> (UI)) -> [U]{

}
```
![image](/res/images/article/swift/2.png)
myMap(_:_:)的使用方式跟map(_:)一样
```
let strings = ["one","two","three"]
let stringLengths = myMap(strings){$0.characters.count}
print(stringLengths)
```
Swift 允许使用类型约束对传递给泛型函数的具体类型进行一些限制，有两种类型约束：
一种是必须是给定类的子类，还有一种是类型必须符合一个协议（或者一个协议组合）
# 关联类型 associated
``` Swift
protocol IteratorProtocol {
    associatedtype Element
    mutating func next() -> Element? 
}
 ```
 associatedtype Element表示符合这个协议的类型必须提供具体类型作为Element类型。符合这个协议的类型应该在其定义内部为Element提供typealias定义
``` Swift
struct StackIterator<T>:IteratorProtocol {
    typealias Element = T
    var stack: Stack<T>
    
    mutating func next() -> Element? {
        return stack.pop()
    }
}
```

StackIterator有点冗余。因为Swift可以判断协议的关联类型，所以只要声明next()返回的是T?，就可以删除显式的类型别名
关联类型也可以约束

``` Swift
public protocol Sequence {

    /// A type representing the sequence's elements.
    associatedtype Element where Self.Element == Self.Iterator.Element

    /// A type that provides the sequence's iteration interface and
    /// encapsulates its iteration state.
    associatedtype Iterator : IteratorProtocol

    /// Returns an iterator over the elements of this sequence.
    func makeIterator() -> Self.Iterator

    /// A value less than or equal to the number of elements in the sequence,
    /// calculated nondestructively.
    ///
    /// The default implementation returns 0. If you provide your own
    /// implementation, make sure to compute the value nondestructively.
    ///
    /// - Complexity: O(1), except if the sequence also conforms to `Collection`.
    ///   In this case, see the documentation of `Collection.underestimatedCount`.
    var underestimatedCount: Int { get }

    /// Call `body(p)`, where `p` is a pointer to the collection's
    /// contiguous storage.  If no such storage exists, it is
    /// first created.  If the collection does not support an internal
    /// representation in a form of contiguous storage, `body` is not
    /// called and `nil` is returned.
    ///
    /// A `Collection` that provides its own implementation of this method
    /// must also guarantee that an equivalent buffer of its `SubSequence` 
    /// can be generated by advancing the pointer by the distance to the
    /// slice's `startIndex`.
    func withContiguousStorageIfAvailable<R>(_ body: (UnsafeBufferPointer<Self.Element>) throws -> R) rethrows -> R?
}

```
如果协议有关联类型，那么这个协议就不能用作具体类型。举个例子，不能声明一个IteratorProtocol类型的变量，也不能声明一个参数类型是IteratorProtocol的函数,因为IteratorProtocol有关联类型。不过，有关联类型的协议对于在泛型声明中使用where子句至关重要
# where
```
associatedtype Element where Self.Element == Self.Iterator.Element
```
```swift
    mutating func pushAll<S: Sequence> (_ sequence: S) where S.Iterator.Element == Elment{
        for item in sequence {
            self.push(item)
        }
    }
```
# @escaping
# @inlinable
# @frozen
# some
```
    func makeIterator() -> some IteratorProtocol {
        
    }
```
