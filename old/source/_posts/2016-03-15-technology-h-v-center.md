---
layout: post
title: 水平垂直居中那些事
date: 2016-03-15 13:32:20 +0300
description: 
cover: /img/mescal/14.jpg # Add image post (optional)
tags: 
	- 博客
	- 技术博客
	- css
categories: [技术博客]
ascription: technology
author: # Add name author (optional)
---
关于水平垂直居中想必大家一定不陌生了，对于前端工程师来说它是基础中的基础也是重点中的重点，我在刚刚接触div+css布局的时候经常因为如何实现水平垂直居中而困扰。下面是我不断积累和总结出来的常见实现方案。
<br>
### 一、水平居中
#### 1.内联元素
```
text-align:center;
```
#### 2.块级元素
上下外边距为0，左右外边距自适应。
```
margin:0 auto;
```
<br>
### 二、水平垂直居中
#### 1.宽高固定时
通过绝对定位和外边距来实现。缺点十分明显，不能自适应。不支持百分比尺寸和min-/max-属性设置并且当含有内边距的时候需要一定的计算量。
```
#parent
{
	height:200px;
	width:200px;
	background: yellow;
	position: relative;
}
#child
{
	height: 100px;
	width: 100px;
	background: red;
	position: absolute;
	left: 50%;
	top:50%;
	margin-left:-50px;/*如果有padding的话就是(width+内边距)/2*/
	margin-top: -50px;/*同上*/
}
```
<br>
#### 2.改进版
当宽高不固定的时候，可以通过css3的transform的translate属性来进行子层的位置调整。这种方案是上述方案的改进版，但是需要实现兼容各浏览器的hack代码。
```
#parent
{
	height:200px;
	width:200px;
	background: yellow;
	position: relative;
}
#child
{
	height: 100px;
	width: 100px;
	background: red;
	position: absolute;
	left: 50%;
	top:50%;
	transform:translate(-50%,-50%);
}
```
<br>
#### 3.换一种思路
既然块级元素可以通过margin:0 auto来实现水平居中，那可不可以使用margin:auto来实现水平垂直居中呢？当然可以，不过需要先做一些铺垫。
```
#parent1
{
	height:200px;
	width:200px;
	background: yellow;
	position: relative;
}
#child1
{
	height: 100px;
	width: 100px;
	background: red;
	position: absolute;
	left: 0;
	right:0;
	top:0;
	bottom:0;
	margin:auto;
}
```
这种方案也是我经常使用的，它支持跨浏览器，包括IE8-IE10.无需其他特殊标记，支持百分比%属性值和min-/max-属性，完美支持图片居中。
<br>
#### 4.不用绝对定位行不行？强大的table
利用table本身的特性来实现水平垂直居中，总的说来这可能是最好的居中实现方法，因为内容块高度会随着实际内容的高度变化，浏览器对此的兼容性也好。唯一的美中不足就是需要在父层和子层之间增加一个中间层。
```
#parent
{
	height:200px;
	width:200px;
	background: yellow;
	display: table;/*!!*/
}
#middle
{
	display: table-cell;/*!!*/
	vertical-align: middle;/*!!*/
}
#child
{
	height: 100px;
	width: 100px;
	background: red;  
    margin: 0 auto;/*!!*/ 
}
```
<br>
#### 5.说个简单的——文字水平垂直居中
假如只有一个层，这个层的高度已知并且这个层中只有文字。那么可以让文字的行高等于层的高度来实现垂直居中，利用text-align属性实现水平居中。
```
div
{
    width:200px;
    height:200px;
    background:yellow;
    line-height:200px;
    text-align:center;
}
```
<br>
#### 6.未来的主流——Flexbox
对于flexbox来说，解决水平垂直居中简直小菜一碟，它甚至可以用来解决更加复杂的布局问题。但是flexbox的兼容性不是很高，需要大量的hack代码，而且不支持IE8/IE9。
```
#parent
{
	width:200px;
	height:200px;
	background: yellow;
	display: -webkit-box;  /* 老版本语法: Safari,  iOS, Android browser, older WebKit browsers.  */
    display: -moz-box;    /* 老版本语法: Firefox (buggy) */ 
    display: -ms-flexbox;  /* 混合版本语法: IE 10 */
    display: -webkit-flex;  /* 新版本语法： Chrome 21+ */
    display: flex;       /* 新版本语法： Opera 12.1, Firefox 22+ */

    /*垂直居中*/	
    /*老版本语法*/
    -webkit-box-align: center; 
    -moz-box-align: center;
    /*混合版本语法*/
    -ms-flex-align: center; 
    /*新版本语法*/
    -webkit-align-items: center;
    align-items: center;
            
    /*水平居中*/
    /*老版本语法*/
    -webkit-box-pack: center; 
    -moz-box-pack: center; 
    /*混合版本语法*/
    -ms-flex-pack: center; 
    /*新版本语法*/
    -webkit-justify-content: center;
    justify-content: center;
}
#child
{
	width:100px;
	height:100px;
	background: red;
}
```
<br>
#### 7.多个块级元素的居中
将子层（块级）设置为inline-block，高度和父层一致。可以自适应。
```
#parent
{
	width:500px;
	height:100px;
	background: yellow;
	text-align: center;
}
#child1,#child2,#child3
{
	width:100px;
	height:100px;
	background: red;
	display:inline-block;
}
```