---
layout: post
title: "html认识标签"
date: 2019-03-22 12:00:00
comments: true
catagories: Html
tags: [Html]
---

# 认识标签(第一部分)
语义化，让你的网页更好的被搜索引擎理解
我们要开始把网页中常用到的标签一 一向大家介绍，学习这一章节的时候要记住学习html标签过程中，主要注意两个方面的学习：标签的用途、标签在浏览器中的默认样式。

标签的用途：我们学习网页制作时，常常会听到一个词，语义化。那么什么叫做语义化呢，说的通俗点就是：明白每个标签的用途（在什么情况下使用此标签合理）比如，网页上的文章的标题就可以用标题标签，网页上的各个栏目的栏目名称也可以使用标题标签。文章中内容的段落就得放在段落标签中，在文章中有想强调的文本，就可以使用 em 标签表示强调等等。

讲了这么多语义化，但是语义化可以给我们带来什么样的好处呢？

<!--more-->

1. 更容易被搜索引擎收录。

2. 更容易让屏幕阅读器读出网页内容。

在后面的章节会带领大家学习了解html中每个标签的语义（用途）。

单击提交按钮进行下一小节的学习！
## \<body>标签，网页上显示的内容放在这里
在网页上要展示出来的页面内容一定要放在body标签中
```
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>了不起的盖茨比</title>
</head>
<body>
    <h1>了不起的盖茨比</h1>
    <p>1922年的春天，一个想要成名名叫<em>尼克•卡拉威</em>（托比•马奎尔Tobey Maguire 饰）的作家，离开了美国中西部，来到了纽约。那是一个道德感渐失，爵士乐流行，走私为王，股票飞涨的时代。为了追寻他的<span>美国梦</span>，他搬入纽约附近一海湾居住。</p>
    
    <p>菲茨杰拉德，二十世纪美国文学巨擘之一，兼具作家和编剧双重身份。他以诗人的敏感和戏剧家的想象为<strong>"爵士乐时代"</strong>吟唱华丽挽歌，其诗人和梦想家的气质亦为那个奢靡年代的不二注解。</p>
    <h1>
        了不起的程序员
    </h1>
    <p>
        我是一个了不起的程序员，<strong>“程序员”</strong>，哈哈哈哈哈哈啊fasfsdfsdfsdfsdfsdfsdf沈德符
        dsfsdfds<em>"程序员"</em>
    </p>
</body>
</html>
```
## \<p>标签，添加段落
如果想在网页上显示文章，这时就需要\<p>标签了，把文章的段落放到\<p>标签中
注意一段文字一个\<p>标签，如在一篇新闻文章中有3段文字，就要把这3个段落分别放到3个\<p>标签中
```
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title> p标签</title>
</head>
<body>
<p>我的第一个段落。</p>
<p>我的第二个段落。</p>
</body>
</html>
```
## \<hx>标签，为你的网页添加标题
标题标签一共有6个，h1、h2、h3、h4、h5、h6分别为一级标题、二级标题、三级标题、四级标题、五级标题、六级标题。并且依据重要性递减。\<h1>是最高的等级。
语法：

`<hx>标题文本</hx> (x为1-6)`

文章的标题前面已经说过了，可以使用标题标签，另外网页上的各个栏目的标题也可使用它们

## \<strong>和\<em> 强调语气
了段落又有了标题，现在如果想在一段话中特别强调某几个文字，这时候就可以用到\<em>或\<strong>标签。
但两者在强调的语气上有区别:\<em> 表示强调，\<strong> 表示更强烈的强调。并且在浏览器中\<em> 默认用斜体表示，\<strong> 用粗体表示。两个标签相比，目前国内前端程序员更喜欢使用\<strong>表示强调。

## \<span>标签为文字设置单独样式
这一小节讲解\<span>标签，我们对\<em>、\<strong>、\<span>这三个标签进行一下总结：

1. \<em>和\<strong>标签是为了强调一段话中的关键字时使用，它们的语义是强调。

2. \<span>标签是没有语义的，它的作用就是为了设置单独的样式用的。

如果现在我们想把上一小节的第一段话“美国梦”三个字设置成blue（蓝色），但注意不是为了强调“美国梦”，而只是想为它设置和其它文字不同的样式（并不想让屏幕阅读器对“美国梦”这三个字加重音读出），所以这样情况下就可以用到\<span>标签了。

如下面例子：
```
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>了不起的盖茨比</title>
<style>
span{
    color:blue;    
}
</style>
</head>
<body>
    <p>1922年的春天，一个想要成名名叫尼克•卡拉威（托比•马奎尔Tobey Maguire 饰）的作家，离开了美国中西部，来到了纽约。那是一个道德感渐失，爵士乐流行，走私为王，股票飞涨的时代。为了追寻他的<span>美国梦</span>，他搬入纽约附近一海湾居住。</p>
    <p>菲茨杰拉德，二十世纪美国文学巨擘之一，兼具作家和编剧双重身份。他以诗人的敏感和戏剧家的想象为"爵士乐时代"吟唱华丽挽歌，其诗人和梦想家的气质亦为那个奢靡年代的不二注解。</p>
</body>
</html>
```
## \<q>标签，短文本引用
想在你的html中加一段引用吗？比如在你的网页的文章里想引用某个作家的一句诗，这样会使你的文章更加出彩，那么\<q>标签是你所需要的
注意要引用的文本不用加双引号，浏览器会对q标签自动添加双引号
```
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>q标签</title>
</head>
<body>
<p>周瑜，不可否认，他是历史上一个了不起的英雄人物！确实也配的上那句<q>聪明秀出为之英，胆略过人为之雄。</q></p>
</body>
</html>
```

##   \<blockquote>标签，长文本引用

\<blockquote>的作用也是引用别人的文本。但它是对长文本的引用，如在文章中引入大段某知名作家的文字，这时需要这个标签。浏览器对\<blockquote>标签的解析是缩进样式。

```
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>blockquote标签的使用</title>
</head>
<body>
<h2>心似桂花开</h2>
<p>大家都在忙于自认为最重要的事情，却没能享受到人生的乐趣，反而要吞下苦果？</p>
<blockquote>暗淡轻黄体性柔，情疏迹远只香留。何须浅碧深红色，自是花中第一流。”</blockquote>
<p>这是李清照《咏桂》中的词句，在李清照看来，桂花暗淡青黄，性情温柔，淡泊自适，远比那些大红大紫争奇斗艳花值得称道。</p>
</body>
</html>
```

## \<br>标签分行显示文本
怎么可以让每一句诗词后面加入一个折行呢？那就可以用到、\<br />标签了，在需要加回车换行的地方加入\<br />，\<br />标签作用相当于word文档中的回车。与以前我们学过的标签不一样，\<br />标签是一个空标签，没有HTML内容的标签就是空标签，空标签只需要写一个开始标签，这样的标签有\<br />、\<hr />和\<img />。在 html 代码中输入回车、空格都是没有作用的。在html文本中想输入回车换行，就必须输入\<br />。

```
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>br标签的使用</title>
</head>
<body>
<h2>《咏桂》</h2>
<p>暗淡轻黄体性柔，情疏迹远只香留。<br/>何须浅碧深红色，自是花中第一流。<br/></p>
</body>
</html
```

## &nbsp为你的网页中添加一些空格
我们已经讲解过在html代码中输入空格、回车都是没有作用的。要想输入空格，必须写入&nbsp;

## \<hr>标签，添加水平横线
在信息展示时，有时会需要加一些用于分隔的横线，这样会使文章看起来整齐些。
\<hr />标签和\<br />标签一样也是一个空标签，所以只有一个开始标签，没有结束标签。
\<hr />标签的在浏览器中的默认样式线条比较粗，颜色为灰色，可能有些人觉得这种样式不美观，没有关系，这些外在样式在我们以后学习了css样式表之后，都可以对其修改。

## \<address>标签，为网页加入地址信息
一般网页中会有一些网站的联系地址信息需要在网页中展示出来，这些联系地址信息如公司的地址就可以<address>标签。也可以定义一个地址（比如电子邮件地址）、签名或者文档的作者身份。

```
<address>
本文的作者：<a href="mailto:lilian@imooc.com">lilian</a>
</address>
```

## \<code>标签
在介绍语言技术的网站中，避免不了在网页中显示一些计算机专业的编程代码，当代码为一行代码时，你就可以使用<code>标签了

```
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>code标签介绍</title>
</head>
<body>
<p>我们可能知道水平渐变的实现，类似这样：<code>{background-image:linear-gradient(left, red 100px, yellow 200px);}</code></p>
</body>
</html>
```
## \<pre>标签为你的网页加入大段代码
在上节中介绍加入一行代码的标签为\<code>，但是在大多数情况下是需要加入大段代码的，
\<pre> 标签的主要作用:预格式化的文本。被包围在 pre 元素中的文本通常会保留空格和换行符。
```
<!DOCTYPE HTML>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>pre标签的使用</title>
</head>

<body>
input:
<pre>
var message="欢迎";
for(var i=1;i<=10;i++)<br>
{<br>
    alert(message); <br>
}<br>
</pre>
</body>
</html>
```

# 认识标签(第二部分)

## ul，添加新闻信息列表
ul-li在网页中显示的默认样式一般为：每项li前都自带一个圆点，如下图所示：
```
<ul>
  <li>信息</li>
  <li>信息</li>
   ......
</ul>
```

## ol，添加图书销售排行榜
如果想在网页中展示有前后顺序的信息列表，怎么办呢？如，当当网上的书籍热卖排行榜，如下图所示。这类信息展示就可以使用\<ol>标签来制作有序列表来展示。
```
<ol>
   <li>信息</li>
   <li>信息</li>
   ......
</ol>
```
## div在排版中的作用
在网页制作过程过中，可以把一些独立的逻辑部分划分出来，放在一个\<div>标签中，这个\<div>标签的作用就相当于一个容器。

语法：
`<div>…</div>`
确定逻辑部分：
什么是逻辑部分？它是页面上相互关联的一组元素。如网页中的独立的栏目版块，就是一个典型的逻辑部分。如下图所示：图中用红色边框标出的部分就是一个逻辑部分，就可以使用\<div>标签作为容器。
```
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>div标签</title>
</head>
<body>
    <div>
    <h2>热门课程排行榜</h2>
    <ol>
        <li>前端开发面试心法 </li>
        <li>零基础学习html</li>
        <li>javascript全攻略</li>
    </ol>
    </div>
    <div>
    <h2>最新课程排行</h2>
    <ol>
        <li>版本管理工具介绍—Git篇 </li>
        <li>Canvas绘图详解</li>
        <li>QQ5.0侧滑菜单</li>
    </ol>
    </div>
</body>
</html>
```

## div命名，使逻辑更加清晰
在上一小节中，我们把一些标签放进\<div>里，划分出一个独立的逻辑部分。为了使逻辑更加清晰，我们可以为这一个独立的逻辑部分设置一个名称，用id属性来为\<div>提供唯一的名称，这个就像我们每个人都有一个身份证号，这个身份证号是唯一标识我们的身份的，也是必须唯一的。
`<div  id="版块名称">…</div>`

## table标签，认识网页上的表格
创建表格的四个元素：
table、tbody、tr、th、td
```
1、<table>…</table>：整个表格以<table>标记开始、</table>标记结束。

2、<tbody>…</tbody>：如果不加<thead><tbody><tfooter> , table表格加载完后才显示。加上这些表格结构， tbody包含行的内容下载完优先显示，不必等待表格结束后在显示，同时如果表格很长，用tbody分段，可以一部分一部分地显示。（通俗理解table 可以按结构一块块的显示，不在等整个表格加载完后显示。）

 

3、<tr>…</tr>：表格的一行，所以有几对tr 表格就有几行。

4、<td>…</td>：表格的一个单元格，一行中包含几对<td>...</td>，说明一行中就有几列。

5、<th>…</th>：表格的头部的一个单元格，表格表头。

6、表格中列的个数，取决于一行中数据单元格的个数。

上述代码在浏览器中显示的默认的样式为：
```
总结：

1、table表格在没有添加css样式之前，在浏览器中显示是没有表格线的

2、表头，也就是th标签中的文本默认为粗体并且居中显示
```
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>认识table表标签</title>
</head>
<body>
<table>
  <tbody>
    <tr>
      <th>班级</th>
      <th>学生数</th>
      <th>平均成绩</th>
      <th>男生数</th>
    </tr>
    <tr>
      <td>一班</td>
      <td>30</td>
      <td>89</td>
      <td>4</td>
    </tr>
    <tr>
      <td>二班</td>
      <td>35</td>
      <td>85</td>
      <td>5</td>
    </tr>
    <tr>
        <td>三班</td>
        <td>32</td>
        <td>80</td>
    </tr>




  </tbody>
</table>
</body>
</html>
```
## 用css样式，为表格加入边框
Table 表格在没有添加 css 样式之前，是没有边框的。这样不便于我们后期合并单元格知识点的讲解，所以在这一节中我们为表格添加一些样式，为它添加边框
```
<!DOCTYPE HTML>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>为表格添加边框</title>
<style type="text/css">
table tr td,th{border:1px solid #000;}
</style>
</head>

<body>
<table summary="">
  <tr>
    <th>班级</th>
    <th>学生数</th>
    <th>平均成绩</th>
  </tr>
  <tr>
    <td>一班</td>
    <td>30</td>
    <td>89</td>
  </tr>
  <tr>
    <td>二班</td>
    <td>35</td>
    <td>85</td>
  </tr>
  <tr>
    <td>三班</td>
    <td>32</td>
    <td>80</td>
  </tr>
</table>

</body>
</html>
```
## caption标签，为表格添加标题和摘要
表格还是需要添加一些标签进行优化，可以添加标题和摘要。
摘要
摘要的内容是不会在浏览器中显示出来的。它的作用是增加表格的可读性(语义化)，使搜索引擎更好的读懂表格内容，还可以使屏幕阅读器更好的帮助特殊用户读取表格内容。
语法:`<table summary="表格简介文本">`
标题

用以描述表格内容，标题的显示位置：表格上方。
语法：
```
<table>
    <caption>标题文本</caption>
    <tr>
        <td>…</td>
        <td>…</td>
        …
    </tr>
…
</table>
```

