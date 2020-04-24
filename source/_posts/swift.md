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

