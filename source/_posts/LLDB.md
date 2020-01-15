---
layout: post
title: "LLDB使用"
date: 2019-05-18 12:00:00
comments: true
catagories: Tools
tags: [iOS]
---

LLDB全称Low Level Debugger ，并不是低水平的调试器，而是轻量级的高性能调试器，默认内置于Xcode中。能够很好的运用它会使我们的开发效率事半功倍，接下来将讲解lldb常用命令及一些高级用法。

<!--more-->

# LLDB常用调试命令

## p、po及 image命令
***p***是打印对象，***po***是打印对象的description，演示如下:  
![image](/res/images/article/lldb/1.gif)   
***p***命令修改变量,演示如下:
![image](/res/images/article/lldb/2.webp)   
***image lookup -a***用于寻找栈地址对应的代码位置,演示如下:    
![image](/res/images/article/lldb/3.webp)      
从上图中我们可以看到当程序崩溃时并不能定位到指定的代码位置，使用image寻址命令可以定位到具体的崩溃位置在viewDidLoad方法中的第51行   
![image](/res/images/article/lldb/4.webp)   
这里说明为什么是程序的名称，因为LLDBDebug在编译后就是一个Macho的可执行文件，也可以理解为镜像文件，image并不是图像的意思，而是代表镜像。这里跟上我们自己的工程名，即用image定位寻址才是寻找我们自己的代码.
## bt及frame命令
使用***bt***命令可以查看函数调用堆栈，然后用***frame select***命令即可查看对应函数详细，演示如下   
![image](/res/images/article/lldb/5.webp)   
上面函数执行的顺序如下：点击登录按钮--验证手机号--验证密码--开始登录
```
- (IBAction)login:(UIButton *)sender {
    
   [self validationPhone];
}
#pragma mark --验证手机号
-(void)validationPhone{

    [self validationPwd];
}
#pragma mark --验证密码
-(void)validationPwd{
    
    [self startLogin];
}
#pragma mark --开始登陆
-(void)startLogin{
   
    NSLog(@"------开始登录...------");
}
```
从***bt***命令的打印信息中，我们可以很清楚看到函数调用顺序，如下图:   
![image](/res/images/article/lldb/6.webp)   
接下来我们执行 ***frame select***命令即可以查看函数相关信息，同时配合***up***和***down***命令追踪函数的调用和被调用关系，演示如下 ：   
![image](/res/images/article/lldb/7.webp)   
同时可以使用***frame variable***很方便的查方法的调用者及方法名称，如下图:   
![image](/res/images/article/lldb/8.webp)   
## breakpoint命令
***b***命令给函数下断点，演示如下图：   
![image](/res/images/article/lldb/9.webp)   
当我们的断点下成功后，控制台会打印如下信息:
Breakpoint 1: where = LLDBDebug`-[ViewController login:] at ViewController.m:53, address = 0x00000001034fb0a0

我们可以看到断点的位置在.m文件的53行，Breakpoint 1这里的1代表的是编号为1的组断点。
使用***breakpoint list***我们可以看到断点的数量，同时使用***breakpoint  delete***后面跟上组号，即可删除，演示如下：  
![image](/res/images/article/lldb/10.webp)   
breakpoint c,\color{red}{n},\color{red}{s}以及\color{red}{finish}命令，对应关系如下图:     
![image](/res/images/article/lldb/11.webp)    
我们执行 c , n , s 及 finish 命令演示如下:   
![image](/res/images/article/lldb/12.webp)   
target stop-hook add -o "frame variable"每次进入断点都会自动打印详细的参数信息，演示如下:   
![image](/res/images/article/lldb/13.webp)   
# LLDB高级用法
我们先来简单看下 menthods 和 pviews 命令的执行效果,演示如下图:    
![image](/res/images/article/lldb/14.webp)   
menthods命令可以打印当前对象的属性和方法，如下所示:   
```
(lldb) methods p1
<Person: 0x60000003eac0>:
in Person:
    Properties:
        @property (copy, nonatomic) NSString* name;  (@synthesize name = _name;)
        @property (nonatomic) long age;  (@synthesize age = _age;)
    Instance Methods:
        - (void) eat; (0x1098bf3e0)
        - (void) .cxx_destruct; (0x1098bf4f0)
        - (id) description; (0x1098bf410)
        - (id) name; (0x1098bf430)
        - (void) setName:(id)arg1; (0x1098bf460)
        - (void) setAge:(long)arg1; (0x1098bf4c0)
        - (long) age; (0x1098bf4a0)
(NSObject ...)
```
pviews命令可以打印当前视图的层级结构，如下所示：
```
(lldb) pviews
<UIWindow: 0x7fd1719060a0; frame = (0 0; 414 736); gestureRecognizers = <NSArray: 0x60c000058660>; layer = <UIWindowLayer: 0x60c0000364c0>>
   | <UIView: 0x7fd16fc06d10; frame = (0 0; 414 736); alpha = 0.8; autoresize = W+H; layer = <CALayer: 0x60000003e7e0>>
   |    | <UIButton: 0x7fd16fe0b520; frame = (54 316; 266 53); opaque = NO; autoresize = RM+BM; layer = <CALayer: 0x60400003b040>>
   |    |    | <UIButtonLabel: 0x7fd16fe023f0; frame = (117.667 17.6667; 30.6667 18); text = '登录'; opaque = NO; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x60400008ac80>>
   |    |    |    | <_UILabelContentLayer: 0x600000220260> (layer)
   |    | <UILabel: 0x7fd16fc04a60; frame = (164 225; 80 47); text = 'Qinz'; opaque = NO; autoresize = RM+BM; userInteractionEnabled = NO; layer = <_UILabelLayer: 0x600000088fc0>>
(lldb) 
```
如果你在原生的XCode中，是敲不出这些命令的，上面只是演示了两个常见的LLDB插件命令的用法，更加高级的用法下面会详细说明。不过在这之前，我们要安装两个插件，接下来先讲解环境的配置。
## LLDB插件配置：chisel及LLDB
chisel是facebook开源的一款LLDB插件，里面封装了很多好用的命令，当然这些命令都是基于苹果提供的api。[chisel下载](https://github.com/facebook/chisel)
这里建议使用包管理工具Homebrew来安装，然后配置脚本路径，演示如下：  
![image](/res/images/article/lldb/15.webp)     

然后在lldb窗口执行命令，演示如下:   
![image](/res/images/article/lldb/16.webp) 

看到输出"command script import /usr/local/opt/chisel/libexec/fblldb.py"即代表安装成功，这里还会看到一个"command script import /opt/LLDB/lldb_commands/dslldb.py
"路径，这是我们接下来要安装的第二个插件
```
Executing commands in '/Users/Qinz/.lldbinit'.
command script import /usr/local/opt/chisel/libexec/fblldb.py
command script import /opt/LLDB/lldb_commands/dslldb.py
(lldb) 
```
这个插件的名称也叫LLDB,[LLDB下载](https://github.com/DerekSelander/LLDB)。我们先clone文件，我这里放置在opt文件夹下，你可以选择自己的文件目录放置，然后依次找到dslldb文件，在~/.initlldb文件中配置路径，演示如下：

![image](/res/images/article/lldb/17.webp)   

接下来依然在lldb窗口执行 command source ~/.lldbinit命令。到此LLDB插件的配置环境完成，接下来我们讲解这些插件的实用命令。

taplog 搭配 flicker ，让你快速找准控件,演示如下：
![image](/res/images/article/lldb/18.webp)  

 taplog是点击控件，会打印控件的地址，大小及透明度等信息，我们拿到地址后执行flicker 0x7fd321e09710命令，此时控件会进行闪烁，这里动态图显示的闪烁效果明显。

 show和hide显示和隐藏控件，演示如下:   
![image](/res/images/article/lldb/19.webp)   
 vs 命令方便动态查看控件的层级关系,演示如下:  
![image](/res/images/article/lldb/20.webp)    
当我们执行\color{red}{vs}命令后会进入动态调试阶段，会出现以下五个命令，每个命令我做了详细注释如下:

```
(lldb) vs 0x7fe73550a090
Use the following and (q) to quit.
(w) move to superview //移动到父视图
(s) move to first subview //移动到第一个子视图
(a) move to previous sibling  //移动上一个兄弟视图
(d) move to next sibling  //移动下一个兄弟视图
(p) print the hierarchy  //打印视图层级结构
```
pactions 直接打印对象调用者及方法,演示如下:

![image](/res/images/article/lldb/21.webp) 

border unborder给控件增加和去除边框，演示如下:   
![image](/res/images/article/lldb/22.webp)   
这里的-c即是color，-w即设置边框的宽度。通过这个命令我们可以很方便的查看边框的边缘的问题，而不需要每次重启运行
pclass 打印对象的继承关系，演示如下图:   
![image](/res/images/article/lldb/23.webp)   
presponder 命令打印响应链,演示如下图:   
![image](/res/images/article/lldb/24.webp)    
caflush这个命令会重新渲染，即可以重新绘制界面， 相当于执行了 [CATransaction flush] 方法,演示如下:   
![image](/res/images/article/lldb/25.webp)    
search 搜索已经存在于栈中的控件及其子控件，演示如下：   
![image](/res/images/article/lldb/26.webp)  
lookup 搜索，可执行正则表达式。演示如下:   
![image](/res/images/article/lldb/27.webp)   
上面的搜索会搜索所用镜像模块，我们重点看与我们工程名字相同的模块，即可查看哪些地方调用了这些方法。
pbundlepath打印app路径及 pdocspath打印文档路径，演示如下：  
![image](/res/images/article/lldb/28.webp) 