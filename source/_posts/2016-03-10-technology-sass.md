---
layout: post
title: Sass初学
date: 2016-03-10 13:32:20 +0300
description: 
cover: /img/mescal/15.jpg # Add image post (optional)
tags: 
    - 博客
    - 技术博客
    - css
categories: [技术博客]
ascription: technology
author: # Add name author (optional)
---
Sass作为css预处理器的一种，极大方便了我们编写css代码。与Less相比较，我个人更加喜欢使用Sass，因为它的代码风格更加接近高级程序设计语言。
Sass上手并不难，可以参考[官方文档](http://sass.bootcss.com/docs/sass-reference/)学习。
### 安装Sass
#### 1.安装ruby
Sass依赖于ruby，可以点击[这里](http://rubyinstaller.org/downloads)安装，在安装过程中勾选Add Ruby executables to your PATH。
#### 2.安装Sass
安装好ruby后，打开Start Command Prompt with Ruby，输入下面命令来进行安装。
```
gem install sass
```

可以通过
```
sass -v
```
来查看版本，这也正说明了Sass已经安装成功了。
<br>
### 编译Sass
#### 1.命令行编译
单文件一次性编译
```
sass style.scss style.css
```
单文件监听
```
sass --watch style.scss:style.css
```
#### 2.Koala
在Less初学篇里已经介绍过，Koala同样可以编译Sass文件，但是这里需要注意，ruby环境默认是不支持中文编码的，因此需要将Koala文件夹中的\rubygems\gems\sass-版本\lib\sass里面的engine.rb添加一行代码Encoding.default_external = Encoding.find('utf-8')(放在所有require xxx后面即可)。
#### 3.使用sublime的插件或者webstorm
sublime支持安装编译Sass文件的插件，webstorm本身就自带编译Sass，这里就不详细介绍了。
#### 4.使用在线编译器 
[戳这里](http://www.sassmeister.com/)
<br>
### Sass常用语法
sass有两种后缀名文件：一种后缀名为sass，不使用大括号和分号；另一种就是scss(支持css3，推荐使用)
#### 1.文件导入
使用@import "文件名"来导入css或scss文件，如果导入的是scss文件，那么被导入的文件(一个或多个)都会被编译最终只生成一个css文件。导入的scss文件可以省去后缀。如果导入的是css文件，那么它将以被导入的形式出现在那么最终生成的css中。

```
@import "style"//省略后缀的style.scss
```
<br>
#### 2.注释
与Less相同，Sass有两种注释：编译到css文件的/**/和不被保存的//。
<br>
#### 3.变量
使用"$变量名"的方式来声明一个变量，与Less的@变量名不同，Sass的这种声明方式更加接近高级程序设计语言。
```
$color:blue;
.div{
    background-color:$color;
}
```
特殊变量：一般用于属性，形式为#{变量名}。
```
$var:left;
.div{
    border-#{$var}:1px solid black;//border-left
}
```
<br>
#### 4.嵌套
Sass支持选择器的嵌套，使得父子关系和代码结构更加清晰。
```
.father{
    width:100px;
    height:100px;
    .son{
        background-color:blue;
        &:hover{
             background-color:red;//&匹配它的上一级选择器
        }
    }
}
```
属性嵌套
```
.div{
    width:100px;
    height:100px;
    border:{    //注意有:
        radius:10px;//相当于border-radius:10px
    }
}
```
@at-root
@at-root可以让它后面的选择器跳出嵌套，自己作为根。
```
.father{
    .son{
        @at-root .skip{
        }
    }
}
//.father{}
//.father .son{}
//.skip{}
```
<br>
#### 5.mixin 混合
mixin可以实现类似于函数的功能，它可以无参，可以有参，也可以指定默认参数。用法和Less相似。
通过@mixin来创建一个混合，通过@include来使用它。

通过无参的mixin来创建一个代码块
```
@mixin init{
    width:100px;
    height:100px;
    background-color:blue;
}
//调用这个代码块
.div{
    @include init;
}
```
为mixin指定参数和缺省值
```
@mixin width($width:50){   //可以指定默认值，这里是50
  width:$width px;
}
.div1{
  @include width(200); //200px
}
.div2{
  @include width();  //50px
```
mixin允许有多个参数
```
@mixin mul($width:100,$height:100,$background:blue){
  width:$width px;
  height:$height px;
  background-color: $background;
}
.div{
  @include mul();
}
```
mixin最常用的地方就是css3的hack代码(如border-radius等)
```
@mixin rounded($v,$h,$radius:5px){
　　-webkit-border-#{$v}-#{$h}-radius: $radius;
　　-moz-border-radius-#{$v}#{$h}: $radius;
　　border-#{$v}-#{$h}-radius: $radius;
}
```
<br>
#### 6.继承
Sass的继承类似于高级程序设计语言的继承，可以使用@extend从一个选择器(占位符)继承它的样式。
```
.father{
  width:100px;
  height:100px;
  background-color: black;
}
.son{
  @extend .father;  //继承了.father的所有样式
  border-width:2px;
}
```
占位符：要实现继承必须有父类，也就是必须要有一个选择器来实现被继承，但是这个父选择器最终也会被编译成css样式。Sass提供了占位符这个功能来实现类似于接口的功能，可以为占位符设定样式，占位符可以被继承并且它最终不会被编译到css文件中，这个功能大大减少了css代码。
```
%father{
  width:100px;
  height:100px;
  background-color: black;
}
.son{
  @extend %father;  //继承了%father的样式，但是%father最终不会被编译
  border-width:2px;
}
```
<br>
#### 7.函数
Sass提供了一些函数，其中最常用的是color函数。用户也可以自己定义函数。
常用的color函数:lighten($color,$amount)和darken($color,$amount)代表颜色减淡和加深,第一个参数为颜色，第二个参数为百分比。
```
body{
  background-color: lighten(black,50%);//gray
}
```
自定义函数
```
@function myfunction($width){
  @return $width*2 px;
}
.div3{
  width:myfunction(300);
}
```
<br>
#### 8.运算
Sass支持变量之间以及变量直接和数值的运算，要注意运算符前面要有一个空格。用法同Sass，这里就不详细叙述了。
条件语句@if和@else：
```
$var:100;
.div{
    @if($var==100){
    width:100px;
}
    @else{
    width:200px;
}
```
三目判断if(condition,true,false)第一个参数代表条件，第二个参数代表为真的时候的值，第三个代表为假的时候的值。
```
$var:100;
.div{
    width:if($var==100,200px,300px);   //width:200px
}
```
循环@for、@while
```
@mixin block{
    width:100px;
    height:100px;
    background-color:black;
}
@for $i from 1 through 10{    //through包括10而to不包括10
  .item-#{$i}{
    @include block;//.item-1 —— .item10:{}
  }
}
```
使用@each来遍历
单个list字段
```
$i:1,2,3,4,5,6,7,8,9,10;
@each $temp in $i{
  .item-#{$temp}{
    border:1px solid red;
  }
}
```
多个list字段
```
$i:(1,blue),(2,grey),(3,yellow),(4,red),(5,black),(6,green),(7,white),(8,gold),(9,blue),(10,red);
@each $temp1,$temp2 in $i{
  .item-#{$temp1}{
    background-color: $temp2;
  }
}
```
遍历map字段
```
$map:(h1:2em,h2:3em,h3:4em);
@each $key,$value in $map{
  #{$key}{
    font-size:$value;
  }
}
```