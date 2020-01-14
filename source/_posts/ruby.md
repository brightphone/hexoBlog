---
layout: post
title: "ruby基本使用"
date: 2019-07-20 12:00:00
comments: true
catagories: language
tags: [Ruby]
---
 

# 多重赋值
## 合并执行多个赋值操作
a, b, *c = 1, 2, 3, 4, 5
p [a, b, c] #=> [1, 2, [3, 4, 5]] a, * b, c = 1, 2, 3, 4, 5
p [a, b, c] #-> [1, [2, 3, 4], 5]

## 置换变量的值
a, b = 0, 1
a, b = b, a # 置换变量a、b 的值 p [a, b] #=> [1, 0

## 获取数组的元素
用数组赋值，左边有多个变量时，Ruby 会自动获取数组的元素进行多重赋值。
ary = [1, 2]
a, b = ary
p a #=> 1 p b #=> 2

只是希望获取数组开头的元素时，可以按照以下示例那样做。左边的变量列表以，结束，给人一种“是不是还没写完?”的感觉，建议尽量少用这样的写法。
ary = [1, 2]
a, = ary
p a #=> 1

## 获取嵌套数组的元素
我们来看看数组 [1, [2, 3], 4]，用之前介绍的方法，我们可以分别取出 1，[2, 3]、4 的值。
ary = [1, [2, 3], 4] 
a, b, c = ary
p a #=> 1
p b #=> [2, 3]
p c #=> 4

像下面那样把左边的变量括起来后，就可以再进一步将内部数组的元素值取出来。
ary = [1, [2, 3], 4]
a, (b1, b2), c = ary # 对与数组结构相对应的变量赋值
p a  #=> 1 
p b1 #=> 2 
p b2 #=> 3 
p c  #=> 4

以变量名开头来决定变量的种类，这是 Ruby 中对变量命名时唯一要坚决遵守的规则。虽然如此，但是根据以往的编程经验，也有一些非强制性的、约 定俗成的变量命名规则。在大多数情况下，遵循这些规则能使程序变得易于阅读，对我们来说有百利而无一害。
不要过多使用省略的名称
有些编程语言会限制变量名的长度，但 Ruby 不需要在意变量名的长度。当然，过长的名称是不便于阅读的，但是与其起个不知所云的短的名称， 老老实实地为变量取个长点的好理解的名称，对以后阅读、理解程序是非常有帮助的。

但是，我们也还是有一些约定俗成的短名称变量。进行数学、物理等计算时，根据计算对象的不同，很多情况下会使用短名称的变量名，像坐标使 用 x、y、z，速度使用 v、w，循环次数使用 m、n 等。另外，我们编写程序时，也经常使用 i、j、k 等作为循环时需用到的变量名。
对于多个单词组合的变量名，使用 _ 隔开各个单词，或者单词以大写字母开头
也就是说，要么这样叫做 sort_list_by_name，要么叫做 sortListByName。一般来讲，Ruby 中的变量名和方法名使用前者，类名和模块名的使用 后者。

在 Ruby 中还有个约定俗成的规则，为了使程序更 容易理解，返回真假值的方法都要以 ? 结尾。建议大家在写程序时也遵守这个规则。

# Ruby 中的条件

## if 语句

if 条件 1 then  
  处理 1
elsif 条件 2 then  
  处理 2
elsif 条件 3 then  
  处理 3
else
 处理 4
end

可以省略 then

## unless 语句

unless 语句的用法刚好与 if 语句相反。unless 语句的用法如下:
unless 条件 then 
  处理
end
※ 可以省略 then
unless 语句的形式和 if 语句一样。但 if 语句是条件为真时执行处理，unless 语句则刚好相反，条件为假时执行处理。
a = 10
b = 20 
unless a > b
  puts "a 不比b 大"
en

## case 语句
case 比较对象 
when 值 1 then  
  处理 1
when 值 2 then  
  处理 2
when 值 3 then  
  处理 3
else
  处理 4
end
※ 可以省略 then

```ruby
tags = [ "A", "IMG", "PRE" ] 
tags.each do |tagname|
    case tagname
    when "P","A","I","B","BLOCKQUOTE"
        puts "#{tagname} has child."
    when "IMG", "BR"
        puts "#{tagname} has no child."
    else
        puts "#{tagname} cannot be used."
    end 
end

array = [ "a", 1, nil ] 
array.each do |item|
    case item 
    when String
        puts "item "
    when Numeric 
        puts "item"
    else
        puts "item"
    end 
end

text.each_line do |line| 
    case line
    when /^From:/i
        puts "发现寄信人信息" 
    when /^To:/i
        puts "发现收信人信息" 
    when /^Subject:/i
        puts "发现主题信息" 
    when /^$/
        puts "头部解析完毕"
        exit 
    else
        ## 跳出处理 
    end
end
```
在 case 语句中，when 判断值是否相等时，实际是使用 === 运算符来判断的。左边是数值或者字符串时，=== 与== 的意义是一样的，除此以外，=== 还可以与=~ 一样用来判断正则表达式是否匹配，或者判断右边的对象是否属于左边的类，等等。对比单纯的判断两边的值是否相等，=== 能表达更加 广义的“相等”
```
p (/zz/ === "xyzzy") #=> true 
p (String === "xyzzy") #=> true 
p ((1..3) === 2) #=> true
```

## if 修饰符与 unless 修饰符

if 与 unless 可以写在希望执行的代码的后面。像下面这样:
```
puts "a 比b 大" if a > b
```
这与下面的写法是等价的。
```
if a > b
  puts "a 比b 大"
end
```
使用修饰符的写法会使程序更加紧凑。通常，我们在希望强调代码执行的内容时会使用修饰符写法。同样地，在使用修饰符写法时，请大家注意程序的易读 性。

对象的同一性
所有的对象都有标识和值。
标识(ID)用来表示对象同一性。Ruby 中所有对象都是唯一的，对象的 ID 可以通过 object_id(或者 __id__)方法取得。
```
ary1 = []
ary2 = []
p ary1.object_id #=> 67653636 
p ary2.object_id #=> 67650432
```

我们用 equal? 方法判断两个对象是否同一个对象(ID 是否相同)。
```
str1 = "foo"
str2 = str1
str3 = "f" + "o" + "o"
p str1.equal?(str2) #=> true 
p str1.equal?(str3) #=> false
```

对象的“值”就是对象拥有的信息。例如，只要对象的字符串内容相等，Ruby 就会认为对象的值相等。Ruby 使用 == 来判断对象的值是否相等。
```
str1 = "foo"
str2 = "f" + "o" + "o"
p str1 == str2 #=> true
```
除了 == 以外，Ruby 还提供 eql? 方法用来判断对象的值是否相等。== 与 eql? 都是 Object 类定义的方法，大部分情况下它们的执行结果都是一样 的。但也有例外，数值类会重定义 eql? 方法，因此执行后有不一样结果。
```
p 1.0 == 1 #=> true
p 1.0.eql?(1) #=> false
```


凭直觉来讲，把 1.0 与 1 判断为相同的值会更加方便。在一般情况进行值的比较时使用 ==，但是在一些需要进行更严谨的比较的程序中，就需要用到 eql? 方法。例如，0 与 0.0 作为散列的键时，会判断为不同的键，这是由于散列对象内部的键比较使用了 eql? 方法来判断。
# 实现循环的方法
## 使用循环语句
利用 Ruby 提供现有的循环语句，可以满足大部分循环处理的需求。
## 使用方法实现循环 
将块传给方法，然后在块里面写上需要循环的处理。一般我们在为了某种特定目的而需要定制循环结构时，才使用方法来实现循环。
下面是我们接下来要介绍的六种循环语句或方法。
### times 方法
```
循环次数.times do  
    希望循环的处理 
end

循环次数.times {  
    希望循环的处理 
}

7.times do
    puts "满地油菜花"
end

10.times do |i|
    puts i #在 times 方法的块里，也是可以获知当前的循环次数的
end

```
### while 语句 
```
while 条件 do  
    希望循环的处理 
end
※ 可以省略 do
i=1
while i < 3
    puts i
    i += 1 
end

names = ["awk","Perl","Python","Ruby"] names.each do |name|
    puts name 
end

```
### each 方法 
each 方法将对象集合里的对象逐个取出，这与 for 语句循环取出数组元素非常相似。实际上，我们可以非常简单地将使用 for 语句的程序(代码清单 6.5)
改写为使用 each 方法的程序
```
对象.each do |变量 |  
    希望循环的处理 
end

names = ["awk","Perl","Python","Ruby"] 
names.each do |name|
    puts name 
end

对象.each {|变量 |  
    希望循环的处理 
}
这与下面的程序的效果是几乎一样。
for 变量 in 对象  
    希望循环的处理 
end
```
### for 语句 
```
for 变量 in 开始时的数值..结束时的数值 do  
    希望循环的处理
end
※ 可以省略 do


sum = 0
for i in 1..5
    sum = sum + i
end
puts sum

for 变量 in 对象 do  
    希望循环的处理 
end
※ 可以省略 do

names = ["awk", "Perl", "Python", "Ruby"] 
for name in names
    puts name 
end


```
### until 语句 
与 if 语句相对的有 unless 语句，同样地，与 while 语句相对的有 until 语句。until 语句的结构与 while 语句完全一样，只是条件判断刚好相反，不满 足条件时才执行循环处理。换句话说，while 语句是一直执行循环处理，直到条件不成立为止;until 语句是一直执行循环处理，直到条件成立为止。
```
until 条件 do  
    希望循环的处理 
end
※ 可以省略 do
```
loop 方法
Ruby 的常用循环结构就介绍到这里，接下来就让我们来具体看看如何使用这些语句或方法实现循环。

## 循环控制
命令    用途
break 终止程序，跳出循环
next  跳到下一次循环
redo  在相同的条件下重复刚才的处理

```
puts "break 的例子:"
i = 0
["Perl", "Python", "Ruby", "Scheme"].each do |lang|
    i+=1
    if i == 3
        break 
    end
    p [i,lang]
end


puts "next 的例子:"
i = 0
["Perl", "Python", "Ruby", "Scheme"].each do |lang|
    i += 1
    if i == 3
        next 
    end
    p [i,lang]
end 



puts "redo 的例子:"
i = 0
["Perl", "Python", "Ruby", "Scheme"].each do |lang|
    i += 1
    if i == 3
        redo 
    end
    p [i,lang]
end

```
            主要用途
times 方法  确定循环次数时使用
for语句     从对象取出元素时使用(each 的语法糖)
while 语句  希望自由指定循环条件时使用
until 语句  使用 while 语句使循环条件变得难懂时使用
each方法    从对象取出元素时使用
loop方法    不限制循环次数时使用

do~end 与 {~}
程序是跨行写的时候使用 do ~ end
程序写在 1 行的时候用 { ~ }
# 方法
## 带块的方法调用
带块的方法的语法如下:
对象.方法名(参数,...) do |变量 1,变量 2,...|  
    块内容
end
do ~ end 这部分就是所谓的块。除 do ~ end 这一形式外，我们也可以用 {~} 将块改写为其他形式:
对象.方法名(参数,...){|变量 1,变量 2,...|  
    块内容
}
备注 使用 do ~ end 时，可以省略把参数列表括起来的 ()。使用 { ~ } 时，只有在没有参数的时候才可以省略 ()，有一个以上的参数时就不能省 略。
在块开头的 | ~ | 部分中指定的变量称为块变量。在执行块的时候，块变量由方法传到块内部。不同的方法对应的块变量的个数、值也都不一样。之前介绍 过的 times 方法有一个块变量，执行块时，方法会从 0 开始依次把循环次数赋值给块变量
## 方法的分类
实例方法
类方法
函数式方法
方法的返回值
可以省略 return 语句，这时方法的最后一个表达式的结果就会成为方法的返回值。下面我们再通过求立方体的表面积这个例子，来看看如何省略 return。 这里，area 方法的最后一行的 (xy + yz + zx) * 2 的结果就是方法的返回值。
```
def area(x, y, z) xy = x * y
    yz = y * z
    zx = z * x
    (xy + yz + zx) * 2 
end
```
如果省略 return 的参数，程序则返回 nil。方法的目的是程序处理，所以 Ruby 允许没有返回值的方法。Ruby 中有很多返回值为 nil 的方法，第 1 章中介 绍的 print 方法就是其中一。
定义带块的方法
首先我们来实现 myloop 方法，它与利用块实现循环的 loop 方法的功能是一样的。
```
def myloop 
while true
    yield end # 执行块
end
num = 1 # 初始化num
myloop do
    puts "num is #{num}" # 输出num
    break if num > 100  # num 超过 100 时跳出循环 # num 乘2
    num *= 2
end

```
参数个数不确定的方法
像下面的例子那样，通过用“* 变量名”的形式来定义参数个数不确定的方法，Ruby 就可以把所有参数封装为数组，供方法内部使用。
```
def foo(*args) 
    args
end
p foo(1, 2, 3) #=> [1, 2, 3]
```
所有不确定的参数都被作为数组赋值给变量 args。“* 变量名”这种形式的参数，只能在方法定义的参数列表中出现一次。只确定首个和最后一个参数名，并 省略中间的参数时，可以像下面这样定义:

```
def a(a, *b, c) 
    [a, b, c]
end
p a(1, 2, 3, 4, 5) #=> [1, [2, 3, 4], 5] 
p a(1, 2) #=> [1, [], 2]
```

关键字参数
```
def 方法名(参数 1: 参数 1 的值, 参数 2: 参数 2 的值, ...)  
    希望执行的处理
end

def area2(x:0,y:0,z:0)
    xy = x * y
    yz = y * z
    zx = z * x
    (xy + yz + zx) * 2
end
p area2(x:2,y:3,z:4)
p area2(z:4,y:3,x:2)
p area2(x:2,z:3)
```
这个方法有参数 x、y、z，各自的默认值都为 0。调用该方法时，可以像 x: 2 这样，指定一对实际的参数名和值。在用关键字参数定义的方法中，每个参 数都指定了默认值，因此可以省略任何一个。而且，由于调用方法时也会把参数名传给方法，所以参数顺序可以自由地更改。
不过，如果把未定义的参数名传给方法，程序就会报错，如下所示:
```
area2(foo: 0) #=> 错误:unknown keyword: foo(ArgumentError)
```
为了避免调用方法时因指定了未定义的参数而报错，我们可以使用“** 变量名”的形式来 接收未定义的参数。下面这个例子的方法中，除了关键字参数
x、y、z 外，还定义了 **arg 参数。参数 arg 会把参数列表以外的关键字参数以散列对象的形式保存。
```
def meth(x: 0, y: 0, z: 0, **args) 
    [x, y, z, args]
end
p meth(z: 4, y: 3, x: 2) #=> [2, 3, 4, {}]
p meth(x: 2, z: 3, v: 4, w: 5) #=> [2, 0, 3, {:v=>4, :w=>5}]
```
关键字参数与普通参数的搭配使用

关键字参数可以与普通参数搭配使用。
```
def func(a, b: 1, c:2)
┊
end
```
上述这样定义时，a 为必须指定的普通参数，b、c 为关键字参数。调用该方法时，可以像下面这样，首先指定普通参数，然后是关键字参数。
```
func(1, b: 2, c: 3)
```
用散列传递参数
调用用关键字参数定义的方法时，可以使用以符号作为键的散列来传递参数。这样一来，程序就会检查散列的键与定义的参数名是否一致，并将与散列 
关于方法调用的一些补充
把数组分解为参数
将参数传递给方法时，我们也可以先分解数组，然后再将分解后的数组元素作为参数传递给方法。在调用方法时，如果以“* 数组”这样的形式指定参 数，这时传递给方法的就不是数组本身，而是数组的各元素被按照顺序传递给方法。但需要注意的是，数组的元素个数必须要和方法定义的参数个数一 样。
```
def foo(a, b, c) 
    a+b+c
end
 
p foo(1, 2, 3) #=> 6
 
args1 = [2, 3]
p foo(1, *args1) #=> 6  
args2 = [1, 2, 3]
p foo(*args2) #=> 6
```
把散列作为参数传递
我们用 { ~ } 这样的形式来表示散列的字面量(literal)。将散列的字面量作为参数传递给方法时可以省略 {}。
```
def foo(arg) 
    arg
end
 
p foo({"a"=>1, "b"=>2}) #=> {"a"=>1, "b"=>2}
p foo("a"=>1, "b"=>2)   #=> {"a"=>1, "b"=>2}
p foo(a: 1, b:2) #=> {:a=>1, :b=>2}
```

当虽然有多个参数，但只将散列作为最后一个参数传递给方法时，可以使用下面的写法:
```
def bar(arg1, arg2) 
    [arg1, arg2]
end
 
p bar(100, {"a"=>1, "b"=>2}) #=> [100, {"a"=>1, "b"=>2}]
p bar(100, "a"=>1, "b"=>2)  #=> [100, {"a"=>1, "b"=>2}]
p bar(100, a: 1, b: 2) #=> [100, {:a=>1, :b=>2}]
```
# class
当想知道某个对象属于哪个类时，我们可以使用 class 方法 `p ary.class #=> Array`
当判断某个对象是否属于某个类时，我们可以使用 instance_of? 方法。`p ary.instance_of?(Array)`
BasicObject 类是 Ruby 中所有类的父类，它定义了作为 Ruby 对象的最基本功能。

备注 BasicObject 类是最最基础的类，甚至连一般对象需要的功能都没有定义。因此普通对象所需要的类一般都被定义为 Object 类。字符串、数组 等都是 Object 类的子类。
![image](/res/images/article/ruby/1.png)

子类与父类的关系称为“is-a 关系”。例如，String 类与它的父类 Object 就是 is-a 关系。
```
str = "This is a String."
p str.is_a?(String) #=> true 
p str.is_a?(Object) #=> true
```
顺便提一下，由于 instance_of? 方法与 is_a? 方法都已经在 Object 类中定义过了，因此普通的对象都可以使用这两个方法。
class 类名  
    类的定义 
end
类名的首字母必须大写。


```
class HelloWorld # class 关键字 
    def initialize(myname = "Ruby") # initialize 方法
        @name = myname # 初始化实例变量 end
    def hello                           # 实例方法 
        puts "Hello, world. I am        #{@name}."
    end 
end
bob = HelloWorld.new("Bob") 
alice = HelloWorld.new("Alice") 
ruby = HelloWorld.new
bob.hello

```
## initialize 方法
initialize 的方法比较特别。使用 new 方法生成新的对象时，initialize 方法会被调用，同时 new 方法的参数也会被原封不动地传给
initialize 方法。因此初始化对象时需要的处理一般都写在这个方法中。

```
def initialize(myname = "Ruby") # initialize 方法 
    @name = myname # 初始化实例变量
end
```
## 存取器
在 Ruby 中，从对象外部不能直接访问实例变量或对实例变量赋值，需要通过方法来访问对象的内部。
为了访问代码中 HelloWorld 类的 @name 实例变量，我们需要定义以下方法:
```
class HelloWorld
    def name # 获取@name
        @name 
    end

    def name=(value) # 修改@name 
        @name = value
    end
end
```
第二个方法的方法名为 name=，使用方法如下:
bob.name = "Robert"
乍一看，该语法很像是在给对象的属性赋值，但实际上却是在调用 name=("Robert") 这个方法。利用这样的方法，我们就可以突破 Ruby 原有的限制，从外 部来自由地访问对象内部的实例变量了。
当对象的实例变量有多个时，如果逐个定义存取器，就会使程序变得难懂，而且也容易写错。为此，Ruby 为了我们提供了更简便的定义方法 attr_reader、attr_writer、attr_accessor。只要指定实例变量名的符号(symbol)，Ruby 就会自动帮我们定义相应的存取器。
定义                     意义
attr_reader :name       只读(定义 name 方法)
attr_writer :name       只写(定义 name= 方法)
attr_accessor :name     读写(定义以上两个方法)
也可以像下面这样只写一行代码，其效果与刚才的 name 方法以及 name= 方法的效果是一样的。
```
class HelloWorld 
    attr_accessor :name
end
```
备注 Ruby 中一般把设定实例变量的方法称为 writer，读取实例变量的方法称为 reader，这两个方法合称为 accessor。另外，有时也把 reader 称为 getter，writer 称为 setter，合称为 accessor method4。一般把 accessor(method)翻译为存取器或者访问器，本书统一翻译为存取器
## 特殊变量 self
在实例方法中，可以用 self 这个特殊的变量来引用方法的接收者。接下来就让我们来看看其他的实例方法如何调用 name 方法。
greet 方法里的 self.name 引用了调用 greet 方法时的接收者。
调用方法时，如果省略了接收者，Ruby 就会默认把 self 作为该方法的接收者。因此，即使省略了 self，也还是可以调用 name 方法，如下所示:
另外，在调用像 name= 方法这样的以 = 结束的方法时，有一点需要特别注意。即使实例方法中已经有了 name = "Ruby" 这样的定义，但如果仅在方法内部 定义名为 name 的局部变量，也不能以缺省接收者的方式调用 name= 方法。这种情况下，我们需要用 self.name = "Ruby" 的形式来显式调用 name 方法。
```
def test_name
    name = "Ruby" # 为局部变量赋值 
    self.name = "Ruby" # 调用name= 方法
end
```
## 类方法
方法的接收者就是类本身(类对象)的方法称为类方法。正如我们在 7.2.2 节中提到的那样，类方法的操作对象不是实例，而是类本身。
下面，让我们在 class << 类名 ~ end 这个特殊的类定义中，以定义实例方法的形式来定义类方法。
```
class << HelloWorld 
    def hello(name)
        puts "#{name} said hello." 
    end
end
HelloWorld.hello("John") #=> John said hello.
```
在 class 上下文中使用 self 时，引用的对象是该类本身，因此，我们可以使用 class << self ~ end 这样的形式，在 class 上下文中定义类方法。
```
class HelloWorld 
    class << self
        def hello(name)
            puts "#{name} said hello."
        end 
    end
end
```
除此以外，我们还可以使用 def 类名 . 方法名 ~ end 这样的形式来定义类方法。
```
def HelloWorld.hello(name) 
    puts "#{name} said hello."
end
HelloWorld.hello("John") #=> John said hello.
```
同样，只要是在 class 上下文中，这种形式下也可以像下面的例子那样使用 self。
```
class HelloWorld
    def self.hello(name)
        puts "#{name} said hello." 
    end
end
```
备注 class << 类名 ~ end 这种写法的类定义称为单例类定义，单例类定义中定义的方法称为单例方法。
## 常量/类变量

对于在类中定义的常量，我们可以像下面那样使用 ::，通过类名来实现外部访问。
以 @@ 开头的变量称为类变量。类变量是该类所有实例的共享变量，这一点与常量类似，不同的是我们可以多次修改类变量的值。另外，与实例变量一样， 从类的外部访问类变量时也需要存取器。不过，由于 attr_accessor 等存取器都不能使用，因此需要直接定义。代码清单 8.4 的程序在代码清单 8.1 的 HelloWorld 类的基础上，添加了统计 hello 方法被调用次数的功能。
```
class HelloCount
    @@count = 0 # 调用hello 方法的次数
    def HelloCount.count # 读取调用次数的类方法 
        @@count
    end
    def initialize(myname="Ruby") 
        @name = myname
    end
    def hello
        @@count += 1 # 累加调用次数
        puts "Hello, world. I am #{@name}.\n"
    end 
end
bob = HelloCount.new("Bob") alice = HelloCount.new("Alice") ruby = HelloCount.new
p HelloCount.count bob.hello alice.hello ruby.hello
p HelloCount.count
#=> 0
#=> 3
```
## 限制方法的调用
到目前为止，我们定义的方法，都能作为实例方法被任意调用，但是有时候我们可能并不希望这样。例如，只是为了汇总多个方法的共同处理而定义的方
法，一般不会公开给外部使用。
Ruby 提供了 3 种方法的访问级别，我们可以按照需要来灵活调整。
public ......以实例方法的形式向外部公开该方法
private ......在指定接收者的情况下不能调用该方法(只能使用缺省接收者的方式调用该方法，因此无法从实例的外部访问) protected ......在同一个类中时可将该方法作为实例方法调用
在修改方法的访问级别时，我们会为这 3 个关键字指定表示方法名的符号。 首先来看看使用 public 和 private 的例子
希望统一定义多个方法的访问级别时，可以使用下面的语法 :
```
class AccTest
public # 不指定参数时，
# 以下的方法都被定义为public
def pub
    puts "pub is a public method."
end

private # 以下的方法都被定义为private
def priv
    puts "priv is a private method."
end end
```
没有指定访问级别的方法默认为 public，但 initialize 方法是个例外，它通常会被定义为 private
定义为 protected 的方法，在同一个类(及其子类)中可作为实例方法使用，而在除此以外的地方则无法使用。
代码清单 8.6 定义了拥有 X、Y 坐标的 Point 类。在这个类中，实例中的坐标可以被外部读取，但不能被修改。为此，我们可以利用 protected 来实现交换 两个坐标值的方法 swap。
```
class Point
    attr_accessor :x, :y # 定义存取器
    protected :x=, :y= # 把x= 与y= 设定为protected
    def initialize(x=0.0, y=0.0) 
        @x, @y = x, y
    end
    def swap(other) # 交换x、y 值的方法 
        tmp_x, tmp_y = @x, @y
        @x, @y = other.x, other.y
        other.x, other.y = tmp_x, tmp_y # 在同一个类中
                                        # 可以被调用
        return self 
    end
end
p0 = Point.new
p1 = Point.new(1.0, 2.0) 
p[p0.x,p0.y]        #=> [0.0, 0.0]
p[p1.x,p1.y]        #=> [1.0, 2.0]
p0.swap(p1) 
p[p0.x,p0.y]        #=> [1.0, 2.0]
p[p1.x,p1.y]        #=> [0.0, 0.0]
p0.x = 10.0         #=> 错误(NoMethodError)
```
## 扩展类
Ruby 允许我们在已经定义好的类中添加方法。下面，我们来试试给 String 类添加一个计算字符串单词数的实例方法 count_word(代码清单 8.7)。
```
class String
    def count_word
        ary = self.split(/\s+/) # 用空格分割接收者
        return ary.size # 返回分割后的数组的元素总数 
    end
end
str = "Just Another Ruby Newbie" 
p str.count_word #=> 4
```
## 继承
利用继承，我们可以在不对已有的类进行修改的前提下，通过增加新功能或重定义已有功能等手段来创建新的类。
定义继承时，在使用 class 关键字指定类名的同时指定父类名。
class 类名< 父类名  
    类定义
end
```
class RingArray < Array # 指定父类
    def [](i)   # 重定义运算符[]
        idx = i % size # 计算新索引值
        super(idx)   # 调用父类中同名的方法
    end 
end

wday = RingArray["日", "月", "火", "水", "木", "金", "土"] 
p wday[6] #=> "土"
p wday[11] #=> "木"
p wday[15] #=> "月"
p wday[-1] #=> "土"
```
利用继承，我们可以把共同的功能定义在父类，把各自独有的功能定义在子类。
定义类时没有指定父类的情况下，Ruby 会默认该类为 Object 类的子类。
Object 类提供了许多便于实际编程的方法。但在某些情况下，我们也有可能会希望使用更轻量级的类，而这时就可以使用 BasicObject 类。
BasicObject 类只提供了组成 Ruby 对象所需的最低限度的方法。类对象调用 instance_methods 方法后，就会以符号的形式返回该类的实例方法列表。下面 我们就用这个方法来对比一下 Object 类和 BasicObject 类的实例方法。
```
 irb --simple-prompt
>> Object.instance_methods
=> [:nil?, :===, :=~, :!~, :eql?, :hash, :<=>, :class, :singleton_class, :clone, :dup, :taint, :tainted?, :untaint, :untrust, :untrusted?, :trust, :freeze, :frozen?, :to_s, ...... 等众多方法名......]
>> BasicObject.instance_methods
=> [:==, :equal?, :!, :!=, :instance_eval, :instance_exec, :__send__, :__id__]
```
## alias
有时我们会希望给已经存在的方法设置别名。这种情况下就需要使用 alias 方法。alias 方法的参数为方法名或者符号名。 
alias 别名 原名   # 直接使用方法名
alias : 别名 : 原名  # 使用符号名
像 Array#size 与 Array#length 这样，为同一种功能设置多个名称时，我们会使用到 alias。 另外，除了为方法设置别名外，在重定义已经存在的方法时，为了能用别名调用原来的方法，我们也需要用到 alias。 下面的例子中定义了类 C1 及其子类 C2。在类 C2 中，对 hello 方法设置别名 old_hello 后，重定义了 hello 方法。
## undef
undef 用于删除已有方法的定义。与 alias 一样，参数可以指定方法名或者符号名。
undef 方法名    # 直接使用方法名 undef : 方法名   # 使用符号名
例如，在子类中希望删除父类定义的方法时可以使用 undef。
## 单例类
定义类方法的方法时，我们提到了单例类定义，而通过利用单例类定义，就可以给对象添加方法(单例方法)。单例类定义被用于定 义对象的专属实例方法。在下面的例子中，我们分别将 "Ruby" 赋值给 str1 对象和 str2 对象，然后只对 str1 对象添加 hello 方法。这样一来，两个 对象分别调用 hello 方法时，str1 对象可以正常调用，但 str2 对象调用时程序就会发生错误。
```
str1 = "Ruby" 
str2 = "Ruby"  
class << str1
    def hello
        "Hello, #{self}!"
    end 
end
 
p str1.hello #=> "Hello, Ruby!"
p str2.hello #=> 错误(NoMethodError)
```
Ruby 中所有的类都是 Class 类的实例，对 Class 类添加实例方法，就等于给所有的类都添加了该类方法。因此，只希望对某个实例添加方法时，就需 要利用单例方法。
单例类的英语为 singleton class 或者 eigenclass。
## 模块的使用方法

所谓命名空间(namespace)，就是对方法、常量、类等名称进行区分及管理的单位。由于模块提供各自独立的命名空间，因此 A 模块中的 foo 方法与 B 模块中的 foo 方法，就会被程序认为是两个不同的方法。同样，A 模块中的 FOO 常量与 B 模块的 FOO 常量，也是两个不同的常量。
无论是方法名还是类名，当然都是越简洁越好，但是像 size、start 等这种普通的名称，可能在很多地方都会使用到。因此，通过在模块内定义名称，就可 以解决命名冲突的问题。
例如，在 FileTest 模块中存在与获取文件信息相关的方法。我们使用“模块名 . 方法名”的形式来调用在模块中定义的方法，这样的方法称为模块函数。
## 利用 Mix-in 扩展功能
Mix-in 就是将模块混合到类中。在定义类时使用 include，模块里的方法、常量就都能被类使用。
像代码清单 8.9 那样，我们可以把 MyClass1 和 MyClass2 中两者共通的功能定义在 MyModule 中。虽然有点类似于类的继承，但 Mix-in 可以更加灵活地解决
下面的问题。
虽然两个类拥有相似的功能，但是不希望把它们作为相同的种类( Class)来考虑的时候 
Ruby 不支持父类的多重继承，因此无法对已经继承的类添加共通的功能的时候
## 创建模块
我们使用 module 关键字来创建模块。
语法与创建类时几乎相同。模块名的首字母必须大写。
module 模块名  
    模块定义 
end
```
module HelloModule  # module 关键字
    Version = "1.0" # 定义常量
    def hello(name) # 定义方法
        puts "Hello, #{name}."
    end
module_function :hello # 指定hello 方法为模块函数
end
p HelloModule::Version 
HelloModule.hello("Alice")
include HelloModule 
p Version 
hello("Alice")
```
然而，如果仅仅定义了方法，虽然在模块内部与包含此模块的上文中都可以直接调用，但却不能以“模块名 . 方法名”的形式调用。如果希望把方法作为模块函 数公开给外部使用，就需要用到 module_function 方法。module_function 的参数是表示方法名的符号。
如果想知道类是否包含某个模块，可以使用 include? 方法。
C.include?(M) #=> true
类 C 的实例在调用方法时，Ruby 会按类 C、模块 M、类 C 的父类 Object 这个顺序查找该方法，并执行第一个找到的方法。被包含的模块的作用就类似于 虚拟的父类。
我们用 ancestors 方法和 superclass 方法调查类的继承关系。在代码清单 8.11 中追加以下代码并执行，我们就可以通过 ancestors 取得继承关系的列
表。进而也就可以看出，被包含的模块 M 也被认为是类 C 的一个“祖先”。而 superclass 方法则直接返回类 C 的父类。
p C.ancestors #=> [C, M, Object, Kernel, BasicObject] 
p C.superclass #=> Object
ancestors 方法的返回值中的 Kernel 是 Ruby 内部的一个核心模块，Ruby 程序运行时所需的共通函数都封装在此模块中。例如 p 方 法、raise 方法等都是由 Kernel 模块提供的模块函数
虽然 Ruby 采用的是不允许多个父类的单一继承模型，但是通过利用 Mix-in，我们就既可以保持单一继承的关系，又可以同时让多个类共享其他功能。
在 Ruby 标准类库中，Enumerable 模块就是利用 Mix-in 扩展功能的一个典型例子。使用 each 方法的类中包含 Enumerable 模块后，就可以使用 each_with_index 方法、collect 方法等对元素进行排序处理的方法。Array、Hash、IO 类等都包含了 Enumerable 模块(图 8.7)。这些类虽然没有继承这 样的血缘关系，但是从“可以使用 each 方法遍历元素”这一点来看，可以说它们都拥有了某种相似甚至相同的属性。
## 查找方法的规则
首先，我们来了解一下使用 Mix-in 时方法的查找顺序。
1 同继承关系一样，原类中已经定义了同名的方法时，优先使用该方法。
2 在同一个类中包含多个模块时，优先使用最后一个包含的模块。
3 嵌套 include 时，查找顺序也是线性的，此时的关系如图 8.8 所示。
```
module M1
┊
end
module M2     
┊
end
module M3 
    include M2  #=> 包含M2
end
class C 
    include M1  #=> 包含M1 
    include M3  #=> 包含M3
end
p C.ancestors #=> [C, M3, M2, M1, Object, Kernel]
```
4 相同的模块被包含两次以上时，第 2 次以后的会被省略。
```
module M1
┊
end
module M2
┊
end
class C 
include     M1  #=> 包含M1
include     M2  #=> 包含M2
include     M1  #=> 包含M1
end
p C.ancestors   #=> [C, M2, M1, Object, Kernel, BasicObject]
```
## extend 方法
在之前的专栏中，我们已经介绍了如何逐个定义单例方法，而利用 Object#extend 方法，我们还可以实现批量定义单例方法。extend 方法可以使单例类包 含模块，并把模块的功能扩展到对象中。
```
module Edition def edition(n)
"#{self} 第#{n} 版" end
end
str = "Ruby 基础教程"
str.extend(Edition) #=> 将模块Mix-in 进对象
p str.edition(4) #=> "Ruby 基础教程第4 版"
```
include 可以帮助我们突破继承的限制，通过模块扩展类的功能;而 extend 则可以帮助我们跨过类，直接通过模块扩展对象的功能。
## 类与 Mix-in
在 Ruby 中，所有类本身都是 Class 类的对象。我们之前也介绍过接收者为类本身的方法就是类方法。也就是说，类方法就是类对象的实例方法。我们可以 把类方法理解为:
Class 类的实例方法
类对象的单例方法
继承类后，这些方法就会作为类方法被子类继承。对子类定义单例方法，实际上也就是定义新的类方法。
除了之前介绍的定义类方法的语法外，使用 extend 方法也同样能为类对象追加类方法。下面是使用 extend 方法追加类方法，并使用 include 方法追加实 例方法的一个例子。
```
module ClassMethods # 定义类方法的模块 def cmethod
"class method" end
end
module InstanceMethods # 定义实例方法的模块 def imethod
"instance method" end
end
class MyClass
# 使用extend 方法定义类方法 extend ClassMethods
# 使用include 定义实例方法 include InstanceMethods
end
p MyClass.cmethod #=> "class method"
p Myclass.new.imethod #=> "instance method"
```
在 Ruby 中，所有方法的执行，都需要通过作为接收者的某个对象的调用。换句话说，Ruby 的方法(包括单例方法)都一定属于某个类，并且 作为接收者对象的实例方法被程序调用。从这个角度来说，人们只是为了便于识别接收者的类型，才分别使用了“实例方法”和“类方法”这样的说法。
Ruby 中的变量没有限制类型，所以不会出现不是某个特定的类的对象，就不能给变量赋值的情况。因此，在程序开始运行之前，我们都无法知道变量指定
的对象的方法调用是否正确。
这样的做法有个缺点，就是增加了程序运行前检查错误的难度。但是，从另外一个角度来看，则可以非常简单地使没有明确继承关系的对象之间的处理变得 通用。只要能执行相同的操作，我们并不介意执行者是否一样;相反，虽然实际上是不同的执行者，但通过定义相同名称的方法，也可以实现处理通用化。 这就是鸭子类型思考问题的方法。
利用鸭子类型实现处理通用化，并不要求对象之间有明确的继承关系，因此，要想灵活运用，可能还需要花不少功夫。例如刚才介绍的 obj[index] 的形 式，就被众多的类作为访问内部元素的手段而使用。刚开始时，我们可以先有意识地留意这种简单易懂的方法，然后再由浅入深，慢慢地就可以抓住窍门 了。
## 范围运算符
范围运算符有 .. 和 ... 两种。x.. y 和 x... y 的区别在于，前者的范围是从 x 到 y;而后者的范围则是从x 到 y 的前一个元素。
对 Range 对象使用 to_a 方法，就会返回范围中从开始到结束的值。下面就让我们使用这个方法来确认一下 .. 和 ... 有什么不同。
## 定义运算符
Ruby 的运算符大多都是作为实例方法提供给我们使用的，因此我们可以很方便地定义或者重定义运算符，改变其原有的含义。但是，表 9.3 中列举的运算
符是不允许修改的。
不能重定义的运算符
::
&&
||
..
...
?:
not
=
and
or
定义四则运算符等二元运算符时，会将运算符名作为方法名，按照定义方法的做法重定义运算符。运算符的左侧为接收者，右侧被作为方法的参数传递。在
代码清单 9.1 的程序中，我们将为表示二元坐标的 Point 类定义运算符 + 以及 -
```
class Point 
attr_reader :x, :y
    def initialize(x=0, y=0)
        @x, @y = x, y
    end
    def inspect # 用于显示 
        "(#{x}, #{y})"
    end
    def +(other) # x、y 分别进行加法运算 
        self.class.new(x + other.x, y + other.y)
    end
    def -(other) # x、y 分别进行减法运算 
        self.class.new(x - other.x, y - other.y)
    end
point0 = Point.new(3, 6)
point1 = Point.new(1, 8)
p point0 #=> (3, 6)
p point1 #=> (1, 8)
p point0 + point1 #=> (4, 14)
p point0 - point1 #=> (2, -2)
end 
```
## puts 方法与 p 方法的不同点

定义了用于显示的 inspect 方法，在 p 方法中把对象转换为字符串时会用到该方法。另外，使用 to_s 方法也可以把对象转换为字符 串，在 puts、print 方法中都有使用 to_s 方法。下面我们来看看两者的区别。
```
> irb --simple-prompt
>> str = "Ruby 基础教程" => "Ruby 基础教程"
>> str.to_s
=> "Ruby 基础教程"
>> str.inspect
=> "\"Ruby 基础教程\""
```
## 一元运算符
可定义的一元运算符有 +、-、~、! 4 个。它们分别以 +@、-@、~@、!@ 为方法名进行方法的定义。下面就让我们试试在 Point 类中定义这几个运算符
。这里需要注意的是，一元运算符都是没有参数的。
```
class Point
    def +@
        dup # 返回自己的副本
    end
    def -@
        self.class.new(-x, -y) # 颠倒x、y 各自的正负
    end
    def ~@ 
        self.class.new(-y, x) # 使坐标翻转90 度
    end 
end
point = Point.new(3, 6) 
p +point #=> (3, 6)
p -point #=> (-3, -6) 
p ~point #=> (-6, 3)
```
## 下标方法
数组、散列中的 obj[i] 以及 obj[i]=x 这样的方法，称为下标方法。定义下标方法时的方法名分别为 [] 和 []=。在代码清单 9.3 中，我们将会定义 Point
类实例 pt 的下标方法，实现以 v[0] 的形式访问 pt.x，以 v[1] 的形式访问 pt.y。
```
class Point
    def [](index) 
        case index 
        when 0
            x 
        when 1
            y 
        else
            raise ArgumentError, "out of range `#{index}'"
        end
    end
    def []=(index, val) 
        case index
        when 0
            self.x = val 
        when 1
            self.y = val 
        else
            raise ArgumentError, "out of range `#{index}'"
        end
    end 
end

point = Point.new(3, 6)
p point[0]      #=> 3
p point[1] = 2  #=> 2
p point[1]      #=> 2
p point[2]      #=> 错误(ArgumentError)
```
# 异常处理
Ruby 中使用 begin ~ rescue ~ end 语句描述异常处理。
begin
 可能会发生异常的处理
rescue
 发生异常时的处理
end
在 Ruby 中，异常及其相关信息都是被作为对象来处理的。在 rescue 后指定变量名，可以获得异常对象。
begin
 可能会发生异常的处理 rescue => 引用异常对象的变量  发生异常时的处理
end
即使不指定变量名，Ruby 也会像表 10.1 那样把异常对象赋值给变量 $!。不过，把变量名明确地写出来会使程序更加易懂。
异常发生时被自动赋值的变量:
$!  最后发生的异常(异常对象)
$@  最后发生的异常的位置信息
异常对象的方法:
class   异常的种类
message 异常信息
backtrace   异常发生的位置信息($@ 与 $!.backtrace 是等价的)

## 后处理

不管是否发生异常都希望执行的处理，在 Ruby 中可以用 ensure 关键字来定义。
begin
    有可能发生异常的处理
rescue => 变量
    发生异常后的处理
ensure  
    不管是否发生异常都希望执行的处理 
end
在 rescue 中使用 retry 后，begin 以下的处理会再重做一遍。
## rescue 修饰符
与 if 修饰符、unless 修饰符一样，rescue 也有对应的修饰符。
表达式 1 rescue 表达式 2
如果表达式 1 中发生异常，表达式 2 的值就会成为整体表达式的值。也就是说，上面的式子与下面的写法是等价的:

begin  
    表达式 1 
rescue  
    表达式 2 
end
我们再来看看下面的例子:
```
n = Integer(val) rescue 0
```
Integer 方法当接收到 "123" 这种数值形式的字符串参数时，会返回该字符串表示的整数值，而当接收到 "abc" 这种非数值形式的字符串参数时，则会抛出 异常(在判断字符串是否为数值形式时经常用到此方法)。在本例中，如果 val 是不正确的数值格式，就会抛出异常，而 0 则作为 = 右侧整体表达式的返回 值。像这样，这个小技巧经常被用在不需要过于复杂的处理，只是希望简单地对变量赋予默认值的时候。
如果异常处理的范围是整个方法体，也就是说整个方法内的程序都用 begin ~ end 包含的话，我们就可以省略 begin 以及 end，直接书写 rescue 与
ensure 部分的程序。
## 指定需要捕捉的异常
当存在多个种类的异常，且需要按异常的种类分别进行处理时，我们可以用多个 rescue 来分开处理。
```
begin
 可能发生异常的处理
rescue Exception1, Exception2 => 变量  
    对Exception1 或者Exception2 的处理 
rescue Exception3 => 变量  
    对Exception3 的处理
rescue
    对上述异常以外的异常的处理
end

file1 = ARGV[0] 
file2 = ARGV[1] 
begin
    io = File.open(file1)
rescue Errno::ENOENT, Errno::EACCES
    io = File.open(file2) 
end
```
## 异常类
之前我们提到过异常也是对象。Ruby 中所有的异常都是 Exception 类的子类，并根据程序错误的种类来定义相应的异常。
在 rescue 中指定的异常的种类实际上就是异常类的类名。rescue 中不指定异常类时，程序会默认捕捉 StandardError 类及其子类的异常。
![image](/res/images/article/ruby/2.png)
rescue 不只会捕捉指定的异常类，同时还会捕捉其子类。因此，我们在自己定义异常时，一般会先定义继承 StandardError 类的新类，然后再继承这个新类
```
MyError = Class.new(StandardError) # 新的异常类 MyError1 = Class.new(MyError)
MyError2 = Class.new(MyError)
MyError3 = Class.new(MyError)
```
MyError = Class.new(StandardError)
上述写法的作用是定义一个继承 StandardError 类的新类，并将其赋值给 MyError 常量。这与下面的效果是一样的。
```
class MyError < StandardError 
end
```
使用 class 语句，我们可以进行定义方法等操作，但在本例中，由于我们只需要生成继承 StandardError 类的新类就可以了，所以就向大家介绍了这个只需 1 行代码就能实现类的定义的简洁写法。
## 主动抛出异常
使用 raise 方法，可以使程序主动抛出异常。在基于自己判定的条件抛出异常，或者把刚捕捉到的异常再次抛出并通知异常的调用者等情况下，我们会使用
raise 方法。
raise 方法有以下 4 种调用方式:
raise message
抛出 RuntimeError 异常，并把字符串作为 message 设置给新生成的异常对象。 raise 异常类
抛出指定的异常。
raise 异常类，message
抛出指定的异常，并把字符串作为 message 设置给新生成的异常对象。
raise
在 rescue 外抛出 RuntimeError。在 rescue 中调用时，会再次抛出最后一次发生的异常($!)。
# 块
块就是在调用方法时，能与参数一起传递的多个处理的集合。之前在介绍 each 方法、time 方法等与循环有关的部分时，我们就已经接触过块。接收块的
方法会执行必要次数的块。块的执行次数由方法本身决定，因此不需事前指定，甚至有可能一次都不执行。do 和 end 之间的部分就是所谓的块。
块的调用方法一般采用以下形式:
对象. 方法名( 参数列表) do | 块变量 |  
    希望循环的处理
end
或
对象. 方法名( 参数列表) { | 块变量 |  
    希望循环的处理
}
块的开头是块变量。块变量就是在执行块的时候，从方法传进来的参数。不同方法的块变量个数也不相同。例如，在 Array#each 方法中，数组的元素会作 为块变量被逐个传递到块中。而在 Array#each_with_index 方法中，则是 [ 元素 , 索引 ] 两个值被传递到块中。
## 块的使用方法
### 循环
在 Ruby 中，我们常常使用块来实现循环。在接收块的方法中，实现了循环处理的方法称为迭代器(iterator)。each 方法就是一个典型的迭代器。
和数组一样，散列也能将元素一个个拿出来，但与数组不同的是，散列会将 [key, value] 的组合作为数组来提取元素。如代码清单 11.1 所示，可以成对地 提取散列的全部键、值。本例中使用 pair[1] 提取并合计了散列的值，提取散列的键时则可以使用 pair[0]。
```
File.open("sample.txt") do |file| 
    file.each_line do |line|
        print line 
    end
end
```
块中最后一个表达式的值就是块的执行结果，因此 <=> 运算符必须在最后一行使用。块的最后一个表达式不是指块的最后一行表达式，而是指在块中最后执行的表达式。
Array#sort 方法没有指定块时，会使用 <=> 运算符对各个元素进行比较，并根据比较后的结果进行排序。<=> 运算符的返回值为-1、0、1 中的一个。
a <=> b 的结果
a <> 时     -1(比 0 小)
a == b时    0
a>b时       1(比 0 大)
## 定义带块的方法
首先让我们重温一下 myloop 方法
```
def myloop 
    while true
        yield   # 执行块
    end
end
num = 1     # 初始化num
myloop do
    puts "num is #{num}"    # 输出num
    break if num > 100      # num 超过100 后跳出循环
    num *= 2                # num 乘2
end
```
myloop 方法在执行 while 循环的同时执行了 yield 关键字，yield 关键字的作用就是执行方法的块。因为这个 while 循环的条件固定为 true，所以会无限 循环地执行下去，但只要在块里调用 break，就可以随时中断 myloop 方法，来执行后面的处理。
## 传递块参数，获取块的值
在刚才的例子中，块参数以及块的执行结果都没有被使用。接下来，我们会定义一个方法，该方法接收两个整数参数，并对这两个整数之间的整数做某种处
理后进行合计处理，而“某种处理”则由块指定
```
def total(from, to)
  result = 0  # 合计值
  from.upto(to) do |num|  # 处理从from 到to 的值
    if block_given?       # 如果有块的话
      result += yield(num)  # 累加经过块处理的值
    else                # 如果没有块的话
      result += num # 直接累加
    end
  end
  return  result  # 返回方法的结果
end

p total(1,10) # 从1 到10 的和 => 55
p total(1,10) {|num|
  num ** 2
} # 从1 到10 的2 次幂的和 => 385
```
total 方法会先使用 Integer#upto 方法把 from 到 to 之间的整数值按照从小到大的顺序取出来，然后交给块处理，最后再将块处理后的值累加到变量 result。程序第 5 行中，对 yield 传递参数后，参数值就会作为块变量传递到块中。同时，块的运行结果也会作为 yield 的结果返回。
程序第 4 行的 block_given? 方法被用来判断调用该方法时是否有块被传递给方法，如果有则返回 true，反之返回 false。如果方法没有块，则在程序第 7 行中直接把 num 相加。
在块中使用 break，程序会马上返回到调用块的地方
