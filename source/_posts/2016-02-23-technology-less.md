---
layout: post
title: Less初学
date: 2016-02-23 13:32:20 +0300
description: 
cover: /img/mescal/16.jpg # Add image post (optional)
tags: 
    - 博客
    - 技术博客
    - css
categories: 技术博客
ascription: technology
author: # Add name author (optional)
---
### 什么是Less
官方文档上面说：Less 是一门 CSS 预处理语言，它扩展了 CSS 语言，增加了变量、Mixin、函数等特性，使 CSS 更易维护和扩展。

说白了Less就是动态的CSS，也就是具备函数、变量、控制语句的CSS。要知道，很多时候CSS都需要对同一个样式重复写很多遍，Less使得CSS的编写更加简单和方便。
<br>
### 如何使用Less
Less可以通过npm来安装
```
$ npm install -g less
```
我使用的是第三方的GUI——koala
koala使用起来非常简单，只需要将Less所在的文件夹放置到koala中配置好输出路径就可以正常使用Less来完成预处理，并且koala自带错误提示功能。
下载地址：http://koala-app.com/index-zh.html
### Less详解
#### 注释
Less新增了“//”注释法，更贴近与程序设计语言。这种注释方法在编译的时候不会被保留，也就是生成的css文件中这个注释会被 pass掉。而“/**/”这种注释法在编译的时候会被保留。
<br>
#### 变量
Less的变量声明方法为：@+变量名
```
@myColor:red;
div{
  background:@myColor;
}
```
编译后的css为
```
div{
  background:red;
}
```
<br>
#### 混合
什么是混合?我个人将它理解成css版本的函数
```
.border{
  border:1px solid black;
}
.box{
  color:white;
  background:red;
  .border;//应用.border的样式  
}
```
有人会问：这个Less哪里简单了，感觉反而更麻烦了。别急，往下面看。
<br>
#### 带参数的混合
带参数的混合就更好理解了，它更向函数去靠近了
```
.border(@myWidth){
  border:@myWidth solid black;
}
.box{
  color:white;
  background:red;
  .border(3px);//传入参数
}
```
当然，混合也可以指定默认的参数，这样就不必每次都给它传参啦
```
.border(@myWidth:3px){
  border:@myWidth solid black;
}
.box{
  color:white;
  background:red;
  .border();//这里可以不用传参了，但是必须要有括号！
}
```
再看一个例子，css3针对不同浏览器做了hack，也就是增加前缀，这种hack方法的确实现了兼容各浏览器开发，但是带来的弊端就是我们必须去写大量的hack代码，不仅浪费时间，整个css文件也十分臃肿。

下面是使用Less来解决这一问题
```
.border(@width:3px){
  -webkit-border-radius:@width;
  -moz-border-radius:@width;
  -ms-border-radius:@width;
  border-radius:@width;
}
div{
  background:red;
  height:100px;
  width:100px;
  .border(50px);
}
```
表面上看起来并不简单，但是当需要多个地方使用这个border-radius的时候只需要调用.border()就可以了而不必再去写重复的代码。把它映射到高级程序设计语言中，是封装一个函数每当需要的时候去调用它方便还是每次都重新敲一遍代码方便？
<br>
#### 匹配模式
这个有点像if语句，但是和if又有很大的区别，它是根据传入不同的参数来执行不同的样式操作
```
.color(r,@width:50px,@height:50px)//如果第一个参数是r则使用该样式
{
  background:red;
}
.color(b,@width:50px,@height:50px)//如果第一个参数是b则使用该样式
{
  background:blue;
}
.color(y,@width:50px,@height:50px)//如果第一个参数是r则使用该样式
{
  background:yellow;
}
.color(@_,@width:50px,@height:50px)
{
  width:@width;
  height:@height;
  .border(10px);
}
```
这里就和if语句不一样了，Less是根据传入的参数的不同来选择究竟进入到哪个“函数之中”。代码的最后一段有一个@_参数，它的意思是，无论匹配是否成功都会进入到这里
```
div{
  .color(r);
}
```
上述代码编译后的css为
```
div{
  background:red;
  width:50px;
  height:50px;
}
```
那么如果我输错了参数会怎样？
```
div{
  .color(g);
}
```
编译后的css就变为
```
div{
  width:50px;
  height:50px;
}
```
无论匹配是否成功，最终都会应用带有@_参数的样式。
<br>
#### 运算
Less支持变量的运算。比如
```
@myWidth:300px;
div{
  width:@myWidth-100;
}
```
最终div的宽度就是200px，可以看到100并没有加单位px，因为@myWidth已经有单位了，所以100可以不用加单位，但是为了更加直观最好还是加上单位。
<br>
#### 嵌套
嵌套可以说是Less最有意思的功能，也是最实用的功能。
举个例子，如果需要创建一个导航栏，样式可能会这样写
```
ul {
  width: 500px;
  margin: 30px auto;
  padding: 0px;
  list-style: none;
}
ul li {
  height: 30px;
  line-height: 30px;
  margin-bottom: 3px;
  background: green;
}
ul li a {
  float: left;
}
ul li a:hover {
  color: blue;
}
ul li span {
  float: right;
}
```
这种写法十分标准，它告诉了我们哪个位置的a标签应用了这个样式，而且标签和标签的嵌套关系也十分明了。但是如果父层的类名十分长，这种写法就比较麻烦了。

而Less模仿了html的嵌套语法
```
ul{
  width:500px;
  margin:30px auto;
  padding:0px;
  list-style: none;
  li{
     height:30px;
     line-height:30px;
     margin-bottom:3px;
     background:green;
     a{
       float:left;
       &:hover{//匹配上一级元素，也就是a
               color:blue;
              }
      }
     span{
          float:right;
      }
   }
}
```
代码中&代表着匹配它的父级选择器，在上述代码里&的父级选择器是a，也就是a:hover。