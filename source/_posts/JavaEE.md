---
layout: post
title: "JavaEE 简介"
date: 2020-05-05 12:00:00
comments: true
catagories: language
tags: [JavaEE]
---
为什么使用框架：
企业应用有两个重要的关注点：可维护性和复用
<!--more-->
# JSP的4种基本语法
## JSP注释：
<%-- 注释内容 --%>
## JSP 声明：
<%! 声明部分 %>
### JSP输出表达式：
<%=表达式%>(输出表达式语法后不能有分号)
对应生成的Java语句为`out.print(表达式)`
## JSP小脚本
JSP静态导入是将被导入页面的代码完全融入，两个页面融合成一个整体Servlet;
动态导入则在servlet中使用include方法来引入被导入页面的内容。
静态导入时被导入页面的编译指令会起作用；而动态导入时被导入页面的编译指令则失去作用，只是插入被导入页面的body内容。
动态导入还可以增加额外的参数。

JSP页面会编译成一个Servlet类，每个Servlet在容器中只有一个实例


