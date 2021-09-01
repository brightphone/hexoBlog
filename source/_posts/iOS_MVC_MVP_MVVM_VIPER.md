
---
layout: post
title: "MVC MVP MVVM"
date: 2021-04-19 12:00:00
comments: true
catagories: language
tags: [iOS]
---



# 什么是MVC模式呢
先来看看下面这张图：

![image](/res/images/article/mvc/1.png)

当然这只是最理想的模式情况下，为什么这么说呢，实际运用中就会出现问题，不然也就不会出现其他mvp，mvvm等模式了?。待会说why，我们先看看这幅图的每个层是什么意思：
- Models： 数据层，负责数据的处理和获取的数据接口层。
- Views： 展示层，即UI层。
- Controller： 控制器层，它是 Model 和 View 之间的媒人，负责牵线搭桥的?。当用户对 View 有操作时它负责去修改相应 Model；当 Model 的值发生变化时它负责去更新对应 View。
# MVC中每层的具体作用
C层：

- 生成view，然后组装view
- 响应View的事件和作为view的代理
- 调用model的数据获取接口，拿到返回数据，处理加工，渲染到view显示
- 处理view的生命周期
- 处理界面之间的跳转

model层：

- 业务逻辑封装
- 提供数据接口给controller使用
- 数据持久化存储和读取
- 作为数据模型存储数据

view层：

- 界面元素搭建，动画效果，数据展示，
- 接受用户操作并反馈视觉效果


# 接下来就说说，倒是什么M层
那么到底什么是M层呢，M层是业务模型。也就是说你所有业务数据和业务实现逻辑都应该定义在M层里面，因为你的业务逻辑的实现和定义是和具体的界面无关的，也就是和视图以及控制之间没有任何的关系，它是可以独立存在的，也就是你用单元测试的时候，是可以直接对立面的业务逻辑进行测试的，不需要运行C层的。说白了就是一块独立的代码块，谁都可以调用。

如果还不理解的话，可以找一个系统的类，他的父类是NSObject类，这个类绝对不会只是一个数据模型，肯定还有很多其他功能的方法或类方法.

# 那么定义M层的原则是什么呢
定义的M层中的代码应该和V层和C层完全无关的，也就是M层的对象是不需要依赖任何C层和V层的对象而独立存在的。整个框架的设计最优结构是V层不依赖C层而独立存在，M层不依赖C层和V层独立存在，C层负责关联二者，V层只负责展示，M层持有数据和业务的具体实现，而C层则处理事件响应以及业务的调用以及通知界面更新。三者之间一定要明确的定义为单向依赖，而不应该出现双向依赖

M层要完成对业务逻辑实现的封装，一般业务逻辑最多的是涉及到客户端和服务器之间的业务交互。M层里面要完成对使用的网络协议(HTTP, TCP，其他)、和服务器之间交互的数据格式（XML, JSON,其他)、本地缓存和数据库存储（COREDATA, SQLITE,其他)等所有业务细节的封装，而且这些东西都不能暴露给C层。所有供C层调用的都是M层里面一个个业务类所提供的成员方法来实现。也就是说C层是不需要知道也不应该知道和客户端和服务器通信所使用的任何协议，以及数据报文格式，以及存储方面的内容。这样的好处是客户端和服务器之间的通信协议，数据格式，以及本地存储的变更都不会影响任何的应用整体框架，因为提供给C层的接口不变，只需要升级和更新M层的代码就可以了。比如说我们想将网络请求库从ASI换成AFN就只要在M层变化就可以了，整个C层和V层的代码不变。

# 现在看看一下ViewController层
首先知道：ViewController是V和C两层的混合体，并不是真正意义上的C层。
上面已经说了VC层的功能和作用了，那么怎么才能给VC减负呢？
我们可以将VC看成是一个view container。
- 管理View Container的生命周期
- 负责生成所有的View实例，并放入View Container
- 监听来自View与业务有关的事件，通过与Model的合作，来完成对应事件的业务。
做到这三点，就可以大大减少VC的代码，而且逻辑更加清楚，因为每个模块的展示和交互是自管理的, 所以VC只需要负责和自身业务强相关的部分即可。

# MVC优缺点

优点

前面介绍了VC和C的区别，按照上面的方法，应对一般场景应该够了。不管界面多复杂，都可以拆分成更小的MVC然后再组装起来。

其实任何好的代码都是经过无数遍的重构和修改而来的（写多了就能感觉出来），尤其是对于圈复杂度和代码深度很高的代码更是经过不断的修改和重构才是代码更简洁更优化。

举个例子：其实很多大公司写的一些大型项目都是无数个sdk组合起来的，打个比方，开发一个商城app，搭建一个app框架，分别显示几个主要的首页，然后把里面的每一个大的模块都分配给不同的项目组，比A项目组负责开发商城，B项目组开发个人中心.....等等，每个项目组开发的功能都以framework的形式提供，然后继承到app这个大壳子里，一个大型项目就成了。再比如，微信的发现列表里的每一项点击进去的功能都可能是一个单独的sdk。这就是功能或者逻辑的单元话，在这个功能模块中，你又可以无限的单元化，直到单元最小为止，然后无论在哪个地方，可以调用你的最小单元功能。也就是所谓的争取做到单一原则，这样做的好处就能最大化的复用和减少代码，也能减少bug，千万不要在一个类里面啥都往里面堆。貌似扯远了。

总结下MVC的优点有如下几点：

- 代码复用: V尽量只对外暴露Set方法, 对M甚至C都是隔离状态, 复用完全没有问题.

- 代码臃肿: 因每种场景的大部分的逻辑和布局都转移到了相应的M、V、C中, 我们做的仅仅是拼装MVC构建出各个场景, 而每个业务场景都能正常的进行相应的数据展示, 也有相应的逻辑交互,就OK了，说白了就是最大化的解耦，也就是上面说的最小功能单元。按照这种写法，还会出现C层一大堆代码的情况吗？当然就不会了，也就是说你严格按照这种MVC的模式写代码，对于大多场景都不会出现代码臃肿现象。

- 易拓展性: 如果后期增加需求了, 我只需要新建相应的MVC模块, 加到对应的场景中即可，也就是上面所说的拼装.

- 可维护性: 各个模块间职责分离, 哪里出错改哪里, 完全不影响其他模块. 另外, 各个模块的代码其实并不算多, 哪一天即使写代码的人离职了, 接手的人根据错误提示也能快速定位出错模块.

- 易测试性: 这个对于C层就有点欠缺了，因为C中有V，通俗点，就是self.view导致的，因为view的生命周期收viewcontroller的影响了。

缺点

对于MVC缺点，其实上面我们也提到了，我们先来看看C主要都干了些什么事情？

作为View和Model的中介者，从model获取数据，经过数据加工，渲染到view上面显示

响应view的点击事件，然后执行相应的业务逻辑

作为view的代理和数据源

暴露接口给SceneVC来驱动自己获取数据

一句话，也就是说C就是这一种场景的管理者，控制这V和M，对于这一个特定的场景，肯定就只有唯一的一个管理者了，所以说C就没发被复用，可能有些人见过给C写子类，按功能一层一层写继承，可能你就想当然的认为这样就可以复用了。这样想就错了，你的子类的功能的实现是建立在父类的功能的基础之上，这不叫复用，应该说这样的复用会出大事，按我的理解，使用继承去实现复用的代码都是垃圾代码，为什么这么说呢？我们来举个例子：

如果你是架构师，搭建了一个大型项目的框架，如果你是开发者，你要实现A功能，需要继承A类，同时你在当前类里还要实现B功能，而实现B功能就必须去继承B类，你可能就会把架构师骂的狗血淋头了?。

再换个场景，说说多层继承，A->B->C，即A的子类是B，B的子类是C。为了实现某个功能你继承了A类的子类B，然后同时你又要实现另外一个功能，需要继承C类，通过重写子类来实现功能，你傻眼了，到底该选择继承哪一个呢？看到这了，你还会觉得通过继承来达到复用的这种方式还叫复用吗？

再比如我上次在github上看到的一个滚动tableview取消键盘的三方库，采用的就是你的vc继承他的vc就能实现键盘的很好处理，可能很多人都觉得没啥，大多项目都有自己的BaseVC，为了处理个键盘，你觉得别人回去使用你的三方库吗？

通过上面的例子说明，既然C的代码没发被复用，那就尽量把C层的代码给拆出来，比如按照上面C层中的四个功能把可以复用的部分代码写到分类里去，记住尽量不要使用继承来实现复用。

# MVP模式
MVP从视图层中分离了行为（事件响应）和状态（属性，用于数据展示），它创建了一个视图的抽象，也就是presenter层，而视图就是P层的【渲染】结果。P层中包含所有的视图渲染需要的动态信息，包括视图的内容（text、color）、组件是否启用（enable），除此之外还会将一些方法暴露给视图用于某些事件的响应。

# 先来看看MVP的架构：
![image](/res/images/article/mvc/2.png)


MVP 架构模式是 MVC 的一个变种，MVC 与 MVP 之间的区别其实并不明显，两者之间最大的区别就是 MVP 中使用 Presenter 对视图和模型进行了解耦，它们彼此都对对方一无所知，沟通都通过 Presenter 进行。具体来说就是：UI的所以点击事件通过delegate来告诉P，然后由P来处理逻辑，然后V也表露一部分接口，P通过V暴露的接口和V进行通信。

通过上图，你可能就疑惑了?，添加了一个P层，那么VC跑哪去了呢？
## MVP模式下的VC的功能
- view的布局和组装

- view的生命周期控制

- 通知各个P层去获取数据然后渲染到view上面展示

也就是说VC由“监听来自View与业务有关的事件，通过与Model的合作，来完成对应事件的业务”变成了“通知各个P层去获取数据然后渲染到view上面展示”。很明显，和MVC中的VC不同点在于把第三条功能给封装到了P里去了，那么具体P层都干了些什么呢，到这里可能你们都已经直到P的作用了。
## MVP中的P到底干了些什么
- 实现view的事件处理逻辑，暴露相应的接口给view的事件调用

- 调用model的接口获取数据，然后加工数据，封装成view可以直接用来显示的数据和状态

- 处理界面之间的跳转（这个根据实际情况来确定放在P还是C）

也就是说V之前的事件逻辑由MVC中的C（不是真正意义上的C，是VC）转移到了P中，既然V的事件逻辑交互跑P里了，那么V和之前的V当然也就有点变化了。 

## MVP中的V的功能？

- 监听P层的数据更新通知, 刷新页面展示.（MVC里由C层负责）

- 在点击事件触发时, 调用P层的对应方法, 并对方法执行结果进行展示.（MVC里由C层负责）

- 界面元素布局和动画

- 反馈用户操作

这也能看出P层的作用以及增加P导致V的交互方向的倾斜变化。那么到底怎么来实现P和V的交互呢？

## MVP中P和V的交互方式
既然谈到交互，那就避免不了数据的传递以及事件的传递。先来问大家一个问题，我们一般是怎么进行值的传递的，即怎么将A中的model传递给B。

一般的做法是，将这个A对象给B，也就是说B持有A，那么当然就B就持有了A的数据了。

还有中做法，就是反过来，也就是将B对象给A，让A持有B，通过B中的回调将数据传给B。

想想是不是很简单，因为这就是我开发中经常用的值传递两种方法，而P和V的交互也是如此。

让P持有V，P通过V的暴露接口改变V的显示数据和状态，P通过V的事件回调来执行自身的业务逻辑

让V持有P，V通过P的代理回调来改变自身的显示数据和状态，V直接调用P的接口来执行事件响应对应的业务逻辑

第一种，很明显，V和被动，因为显示数据是P来通知V的，而响应事件也是由P来调用的，只不过是由V来回调的，但是调用的主动权还是在P上，所以V显得特别被动，也就是所谓的变种被动视图。

那么反过来，第二种，P就相对独立了，他只负责业务逻辑，而由于业务逻辑导致的数据显示的变化，让view实现对应的代理来实现。所以增加了View的复杂程度。

综合对比，第一种View可以实现单纯的数据的显示，可以实现最大复用性，因为他和其他任何人都关系，我只负责显示，说白了就是，对于懒懒的胖子而言，你踢我一脚，我动一步，你不踢我，我就躺着不动，没有外力促使我动，我就什么也不做，我只负责显示我的外表和姿势?。第二种P只处理逻辑，至于事件的触发交互部分偏移到了View，由View负责选择回调的代理方法。所以View的复用的纯粹性变差了。

任何事物都是由两面性的（形容的不一定恰当哈，?不要在意细节）。针对上面的MVP模式也就是常说的被动MVP模式，我们可以看到，M和V是完全隔离的，对于任何的M的变化或者任何的V的变化都是通过P来当中间人的，这就导致了代码变多了。为啥这么说呢？我们来看一下他们之间是怎么通信的。
## 被动MVP模式的通信方式
![image](/res/images/article/mvc/3.png)
当视图接收到来自用户的事件时，会将事件转交给 Presenter 进行处理；

被动的视图向外界暴露接口，当需要更新视图时 Presenter 通过视图暴露的接口更新视图的内容；

Presenter 负责对模型进行操作和更新，在需要时取出其中存储的信息；

当模型层改变时，可以将改变的信息发送给观察者 Presenter

在 MVP 的变种被动视图中，模型的操作以及视图的更新都仅通过 Presenter 作为中间人进行。

看着上面的通信方式，然后我再举个简单的例子，由于例子太简单，我就不些代码了。

现在需求是点击V的关注按钮，按钮由关注变为已关注，不通通过网络请求，只是单纯的改变状态。有两种办法：

如果我只是点击V的一个关注按钮，然后将M中的一个是否关注的Bool属性从NO变为YES。

点击V的一个关注按钮，通过代理告诉P，我的状态变了，你去告诉M，然后他去告诉M，然后M改这个属性的值，然后再通知P，我已经把值改了，你现在去通知V，让他改变按钮的显示状态。

如果是你选，你会选择1还是2？一目了然，你当然会选择1，为啥？因为代码少呀，而且简单呀。也就是说视图和模型之间新增了数据绑定的依赖关系。那么这种变化后的模式也就是所谓的监督控制器模式的MVP。接下来就看看这种监督控制器模式的MVP具体是怎样的。
## 什么是监督控制器模式的MVP

与被动视图中状态同步都需要显式的操作不同，监督控制器（Supervising Controller）就将部分需要显式同步的操作变成了隐式的。

![image](/res/images/article/mvc/4.png)
在监督控制器中，视图层接管了一部分视图逻辑，主要内容就是同步简单的视图和模型的状态；而监督控制器就需要负责响应用户的输入以及一部分更加复杂的视图、模型状态同步工作。

对于用户输入的处理，监督控制器的做法与标准 MVP 中的 Presenter 完全相同。但是不同点在于对视图、模型的数据同步工作，监督控制器会尽可能地将所有简单的属性以数据绑定的形式声明在视图层中，而无法通过直接绑定的属性就需要通过监督控制器来操作和更新了。

由上图可以看出视图和模型之间新增了双向的数据绑定的依赖关系；视图通过声明式的语法与模型中的简单属性进行绑定，当模型发生改变时，会通知其观察者视图作出相应的更新。

通过这种方式能够减轻监督控制器的负担，减少其中简单的代码，将一部分逻辑交由视图进行处理；这样也就导致了视图同时可以被 Presenter 和数据绑定两种方式更新，相比于被动视图，监督控制器的方式也降低了视图的可测试性和封装性。

有兴趣的可以看看这篇博客：https://www.martinfowler.com/eaaDev/SupervisingPresenter.html

说了这么多MVP的原理，那么到底怎么实现呢？

## MVP的实现举例

```
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    
    CustomTableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:ReuseIdentifier];
    //关注的业务逻辑
    Model *model = self.datas[indexPath.row];
    __weak typeof(cell) weakCell = cell;
    [cell attentionHandler:^{
        //........更新model
        处理逻辑
        if(){
          model.isAttention = NO;
        }else{
          model.isAttention = YES;
        }
        //然后再更新当前Cell
    }];
    return cell;
}
```
上面的是MVC下的代码，那么在MVP模式下就是将逻辑部分放到P层就行了，代码如下：

```
ViewController.m
 
//关注事件
 
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    
    CustomTableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:ReuseIdentifier];
    //关注的业务逻辑
    Model *model = self.datas[indexPath.row];
    __weak typeof(cell) weakCell = cell;
    [cell attentionHandler:^{
        //........更新model
        处理逻辑
 
        [weakCell.presenter attentionAction];
        
    }];
    return cell;
}
 
==========================================
CostomCellPresenter.m
 
- (void)attentionAction {
        if(){
          self.model.isAttention = NO;
        }else{
          self.model.isAttention = YES;
        }
    
        [self.cell didUpdateAttentionState:self];
}
 
==========================================
CustomViewCell.m
 
#pragma mark - PresenterCallBack
 
- (void)didUpdateAttentionState:(CustomCellPresenter *)presenter {
    
    //改变按钮状态
}
 
 
#pragma mark - Action
 
- (IBAction)onClickAttentionButton:(UIButton *)sender {
    !self.attentionHandler ?: self.attentionHandler();
}

```
# MVVM
## 到底什么是MVVM模式？MVVM和MVP有什么区别呢
上面所说的MVP的数据双向绑定的实现就是MVVM模式解决的事情，也就是说MVVM其实是在MVP的基础上发展起来的。

有兴趣的同志可以看看这几篇文章，看完就知道MVVM的由来了。

https://blogs.msdn.microsoft.com/johngossman/2005/10/08/introduction-to-modelviewviewmodel-pattern-for-building-wpf-apps/

https://www.martinfowler.com/eaaDev/PresentationModel.html

https://docs.microsoft.com/en-us/previous-versions/msp-n-p/hh848246(v=pandp.10)
废话就不扯了，来看看MVVM的结构：

![image](/res/images/article/mvc/5.jpg)
它由Model、View 和 ViewModel三个部分组成，其中视图模型（ViewModel）在 MVVM 中叫做视图模型。

看上图，相比MVP模式，MVVM的改进就很一目了然了，在 MVVM 的实现中引入了隐式的一个 Binding层，也就是说在MVP模式中的声明式的数据和命令的绑定在 MVVM 模式中就是通过binding层来完成的。

MVVM各层的职责和MVP的类似，VM对应P层，只是在MVVM的View层多了数据绑定的操作。既然是建立在MVVM的基础上发出来的东东，那么这样做相对于MVP肯定就有他的优势或者说优点，说白了就是对MVP进行了那些短版的弥补.
## MVVM相对于MVP所做的改进
还是上面的关注例子，我们先来看看MVP实现一个关注业务的流程

点击关注按钮--->通知P处理关注逻辑---->处理完逻辑后P去改变M的数据--->然后P再通知cell中的关注按钮来改变按钮状态由【关注】变为【已关注】

其实上面也已经提到过，这种行为和数据的同步比较繁琐，写起代码来比较费时间，代码量增大了很多。这对这个短版，就出现了MVVM模式，我们先来看看这binding是个什么鬼。
![image](/res/images/article/mvc/6.jpg)
还是上面的关注例子，我们来看看

先看MVP的代码：
```
ViewController.m
 
//关注事件
 
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    
    CustomTableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:ReuseIdentifier];
    //关注的业务逻辑
    Model *model = self.datas[indexPath.row];
    __weak typeof(cell) weakCell = cell;
    [cell attentionHandler:^{
        //........更新model
        处理逻辑
 
        [weakCell.presenter attentionAction];
        
    }];
    return cell;
}
 
==========================================
CostomCellPresenter.m
 
- (void)attentionAction {
        if(){
          self.model.isAttention = NO;
        }else{
          self.model.isAttention = YES;
        }
    
        [self.cell didUpdateAttentionState:self];
}
 
==========================================
CustomViewCell.m
 
#pragma mark - PresenterCallBack
 
- (void)didUpdateAttentionState:(CustomCellPresenter *)presenter {
    
    //改变按钮状态
}
 
 
#pragma mark - Action
 
- (IBAction)onClickAttentionButton:(UIButton *)sender {
    !self.attentionHandler ?: self.attentionHandler();
}

```
我们再来看MVVM的代码：
```
ViewModel.m
 
        @weakify(self);
        self.attentionCommand = [[RACCommand alloc] initWithSignalBlock:^RACSignal *(id input) {
            @strongify(self);
         
            RACSubject *subject = [RACSubject subject];
            //一堆逻辑,更新属性值
                 .....
            if () {
                
                
            } else {
                
            }
            [subject sendCompleted];
            return subject;
        }];
        
 
- (void)awakeFromNib {
    [super awakeFromNib];
    
    //数据绑定操作
    @weakify(self);
    RAC(self.attentionButton, selected) = [RACObserve(self, viewModel.isAttention) ignore:nil];
    [RACObserve(self, viewModel.blogLikeCount) subscribeNext:^(NSString *title) {
        @strongify(self);
        [self.attentionButton setTitle:title forState:UIControlStateNormal];
    }];
    
}
 
- (IBAction)onClickLikeButton:(UIButton *)sender {
    //事件响应
    [[self.viewModel.attentionCommand execute:nil] subscribeError:^(NSError *error) {
            //
    }];
}
```
## RAC到底是个什么东东？她有什么用？她的魅力在哪？
ReactiveCocoa（简称为RAC）,是由Github开源的一个应用于iOS和OS开发的一个框架。

ReactiveCocoa是做什么的？

在我们iOS开发过程中，当某些事件响应的时候，需要处理某些业务逻辑,这些事件都用不同的方式来处理。

比如按钮的点击使用action，ScrollView滚动使用delegate，属性值改变使用KVO等系统提供的方式。

其实这些事件，都可以通过RAC处理

ReactiveCocoa为事件提供了很多处理方法，而且利用RAC处理事件很方便，可以把要处理的事情，和监听的事情的代码放在一起，这样非常方便我们管理，就不需要跳到对应的方法里。非常符合我们开发中高聚合，低耦合的思想。

换句话说，ReactiveCocoa 其实就是一个用来帮我们处理事件的一个三方的框架！

好了，加了点小插曲，至于RAC到底该怎么用，你们自己去了解，这么大的一个三方库，有自己的demo，传送门。

言归正传，上面的MVVM经过这个RAC这样一整，看似代码清晰明了，并且代码也变简单了，但是却引入了其他问题，下来我们就简单的说说MVVM的缺点。优点就不说了，为了解决MVP的短版，优点就一目了然了，这里就不赘述了。
## MVVM的缺点
前面就说过事物都是有两面性的，既然有优点就肯定有缺点。那么MVVM的缺点是什么呢？

数据绑定使得Bug很难被调试。如果界面发生错误，有可能是View的代码有问题，也有可能是Model的代码有问题。数据绑定使得一个位置的Bug被快速传递到别的位置，要想定位原始出问题的地方，就变得不那么容易了。

对于大型项目，数据绑定和数据转化需要花费更多的内存（成本）。主要在于：数组内容的转化成本较高：数组里面每项都要转化成Item对象，如果Item对象中还有类似数组，就很头疼。转化之后的数据在大部分情况是不能直接被展示的，为了能够被展示，还需二次转化。只有在API返回的数据高度标准化时，这些对象原型（Item）的可复用程度才高，否则容易出现类型膨大，提高维护成本。

调试时通过对象原型查看数据内容，不如直接通过NSDictionary或者NSArray这样直观。

同一API的数据被不同View展示时，难以控制数据转化的代码，它们有可能会散落在任何需要的地方。

这些缺点就不一一举例说明啦，我也没发给你整个大项目放这呀?。了解了解就行啦，这不是重点，重点是理解MVVM???。

网上很多流传着还有一种模式，叫VIPER，我也学习了下，这里就也唠叨唠叨哈。顺便也可以做个对比

# VIPER模式是什么样子的？
![image](/res/images/article/mvc/7.png)

从图中可以看出，VIPER彻底将VC变成了真正意义上的View。把VC的职责进行了彻底的拆分，分散到各个子层里面了。单看结构图就吓了一跳，会不会代码更复杂了呀。是的，代码复杂度确实大大增加了。具体分析下，其他架构因为有VC的存在，或多或少都会导致各层的职责划分不明确。但是也由于VIPER的分层过多，并且是唯一一个把界面路由功能单独分离出来放到一个单独的类里面处理，所有的事件响应和界面跳转都需要自己处理，所以就导致代码复杂度大大增加。

现在想想，苹果为啥宁愿导致层次耦合也要搞出个VC（V和C的混合体）了，人家的那个VC确实是简化了开发流程呀。而VIPER则是彻底抛弃了VC，重新进行分层，做到了每个模块都可以单独测试和复用，但是也导致了代码过多、逻辑比较绕的问题。

其实只要做好分层和规划，MVC架构足够应付大多数场景。有些文章上来就说MVVM是为了解决C层臃肿, MVC难以测试的问题, 其实并不是这样的. 按照架构演进顺序来看, C层臃肿大部分是没有拆分好MVC模块, 好好拆分就行了, 用不着MVVM。 而MVC难以测试也可以用MVP来解决, 只是MVP也并非完美, 在VP之间的数据交互太繁琐, 所以才引出了MVVM。 而VIPER则是跳出了MVX架构，自己开辟一条新的路。

VIPER将每个模块与其他模块隔离开来。因此，更改或修复错误非常简单，因为您只需要更新特定的模块。此外，VIPER还为单元测试创建了一个非常好的环境。由于每个模块独立于其他模块，因此保持了低耦合。在开发人员之间划分工作也很简单。

那么VIPER具体的每个模块的功能是什么呢？
## VIPER各层职责
Interactor（交互器） - 这是应用程序的主干，因为它包含应用程序中用例描述的业务逻辑。交互器负责从数据层获取数据，并执行特定场景下的业务逻辑，其实现完全独立于用户界面。

Presenter（展示器） - 它的职责是从用户操作的Interactor获取数据，创建一个Entities实例，并将其传送到View以显示它。

Entities（实体） - 纯粹的数据对象。不包括数据访问层，因为这是 Interactor 的职责。

Router（路由） - 负责 VIPER 模块之间的跳转

View（视图）- 视图的责任是将用户操作发送给演示者，并显示presenter告诉它的任何内容

也就是说一个应用场景的所有功能点都被分离成功能完全独立的层，每个层的职责都是单一的。在VIPER架构中，每个块对应于具有特定任务，输入和输出的对象。它与装配线中的工作人员非常相似：一旦工作人员完成其对象上的作业，该对象将传递给下一个工作人员，直到产品完成。

所以VIPER架构将您的应用程序逻辑分为较小的功能层，每个功能都具有严格的预定责任。这使得更容易测试层之间边界的交互。它适用于单元测试，并使您的代码更可重用。

## VIPER 架构的优点
简化复杂项目。由于模块独立，VIPER对于大型团队来说真的很好。

使其可扩展。使开发人员尽可能无缝地同时处理它

代码达到了可重用性和可测试性

根据应用程序的作用划分应用程序组件，设定明确的责任

可以轻松添加新功能

由于您的UI逻辑与业务逻辑分离，因此可以轻松编写自动化测试

它鼓励分离使得更容易采用TDD的关注。Interactor包含独立于任何UI的纯逻辑，这使得通过测试轻松开车

创建清晰明确的接口，独立于其他模块。这使得更容易更改界面向用户呈现各种模块的方式。

通过单一责任原则，通过崩溃报告更容易地跟踪问题

使源代码更清洁，更紧凑和可重用

减少开发团队内的冲突数量

适用SOLID原则

使代码看起来类似。阅读别人的代码变得更快。

所以呢，在小项目中使用VIPER那是坑自己呀，除非你的时间很充足，你们团队都闲的慌，或者有这种自虐的爱好?，因为MVP或MVC就足够了。
