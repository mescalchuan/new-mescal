---
layout: post
title: ES6学习笔记（一）
date: 2016-10-30 13:32:20 +0300
description: 
cover: /img/mescal/12.jpg # Add image post (optional)
tags: 
	- 博客
	- 技术博客
	- javascript
categories: [技术博客]
ascription: technology
author: # Add name author (optional)
---
### 最近学习了ES6，被它更简洁的代码编写方式、面向对象以及模块化开发所吸引。ES6有一定的学习成本而且知识点比较琐碎，我把自己经常用到的知识点进行了整理。

#### 安装与配置
es6很强大，遗憾的是所有浏览器都没有完美支持它。因此我们需要通过babel将es6转换成es5让我们的代码在浏览器端运行。

我本人用的是webpack+babel，你也可以使用gulp或者直接将babel的browser.js引入，将type设为"text/babel"。

首先安装webpack
```
npm install webpack --save-dev 
```

webpack需要安装相应的loader加载器—babel-loader，别忘了安装转换规则—babel-preset-es2015
```
npm install babel-loader --save-dev
npm install babel-preset-es2015 --save-dev
```

接下来简单地配置一下webpack.config.js文件
```
module.exports = {
  entry: "./main.js",
  output: {
    path: __dirname,
    filename: "after.js"
  },
  module: {
    loaders: [
      {
        test: /\.js$/,
        loader: 'babel-loader',
        query: {
          presets: ['es2015']
        }
      }
    ]
  },
}
```

上面的main.js就是我们即将要写的es6代码，通过在命令行输入`webpack`将其转化成es5并保存在after.js中，因此我们只需要引入after.js即可。如果你希望webpack可以实时监听并编译，可以安装webpack-dev-server，这里不多说明。

小贴士：如果你在使用es6中的Generator、async的时候遇到了错误信息`regeneratorRuntime is not defined`，请单独安装babel-polyfill：在webpack.config.js的头部`var babelpolyfill = require("babel-polyfill");`，将入口文件改写成`entry:["babel-polyfill","./main.js"],`，然后在你的js中`import "babel-polyfill"`。

准备工作已经完成，接下来我们可以测试一下，我准备了一个用es6编写的main.js文件
```
class MyClass{
	constructor(x,y){
		this.x=x;
		this.y=y;
	}
	print(){
		return `(${this.x},${this.y})`;
	}
}
var myclass=new MyClass(2,3);
console.log(myclass.print());
```

运行webpack后看看生成的after.js里面都有啥
```
//部分代码
var MyClass = function () {
		function MyClass(x, y) {
			_classCallCheck(this, MyClass);
			this.x = x;
			this.y = y;
		}
		_createClass(MyClass, [{
			key: "print",
			value: function print() {
				return "(" + this.x + "," + this.y + ")";
			}
		}]);
		return MyClass;
	}();
    var myclass = new MyClass(2, 3);
	console.log(myclass.print());
```
可以看到webpack已经将es6转化为es5，现在可以尽情使用es6了。
### let和const
#### let
let是新的变量声明方式，使用方法类似于var，但是let声明的变量只在它所在的代码块内有效
```
{
	var name="SunnyChuan";
	let age=22;
}
console.log(name); //"SunnyChuan"
console.log(age);  //age is not defined
```

let变量只在自己的块级作用域内有效，也就意味着使用let可以代替闭包解决常见的for循环引用同一个变量的问题
```
for(let i=0;i<5;i++)console.log(i);  //0 1 2 3 4
```

let不存在变量声明提升，必须先声明后使用
```
console.log(name); //ReferenceError
let name="SunnyChuan";
```

let不允许在同一个作用域内重复声明同一个变量（无论新变量是let/const/var）
```
let name="SunnyChuan";
var name="DannyQin"; //报错
```

#### const
const是es6增加的另一个变量声明方式，它用来声明常量，并且声明后变量的值不能再更改。因此一旦声明必须初始化，不能稍后再赋值
```
const a=100;
a=10; //TypeError
const b  //SyntaxError;
b=100;
```

const和let一样存在自己的作用域，个人认为除了变量的值是否可以改变以及是否必须声明立即初始化之外，let和const没有其他区别。需要注意的是，对于对象，数组等复合型的变量，const保证的是地址不变而不是值不变
```
const arr=[1,2,3];
arr.push(4); //地址不变，改变值是可以的
arr=[5]; //修改了地址，报错
```

### 变量的解构赋值
es6支持解构赋值的形式将变量的赋值简单化，数组/对象/字符串/数值/布尔/函数均可解构赋值。最为常用的是数组和对象。
#### 数组的解构赋值
只要等号两边的模式相同即可解构成功
```
var [a,b,c]=[1,2,3] //a=1,b=2,c=3
var [a,,b] //a=1,b=3
var [a,..b] //a=1,b=[2,3]
```

当左边的数量大于右边时，右边会默认用undefined进行补充，导致解构不成功
```
let [a,b,c]=[1,2] //a=1,b=2,c=undefined
```

允许指定默认值，默认值只有当左边的变量所对应的右边的值严格等于undefined才生效
```
var [a,b,c=3]=[1,2]; //a=1,b=2,c=3
var [a,b,c=3]=[1,2,null]; //a=1,b=2,c=null
```

#### 对象的解构赋值
与数组不同的是，对象解构赋值依靠的是key值，因此只要左边和右边key值同名即可（变量名等于属性名），无所谓顺序，而数组是按照顺序赋值的
```
var {name,age}={name:"SunnyChuan",age:22}; //name="SunnyChuan" age=22 
//相当于
var {name:name,age:age}={name:"SunnyChuan",age:22};
```

按照上面的匹配方式，变量名必须等于属性名，如果不相等则匹配失败。如果变量名不等于属性名，只能使用属性名：变量名的形式，不能省去变量名。这时候，属性只不过是一个模式，真正被赋值的是变量
```
var {name:myName,age:myAge}={name:"SunnyChuan",age:22};
//name is not defined, age is not defined, myName="SunnyChuan", myAge=22
```

对象解构赋值也提供了默认值，与数组相同，只有属性值严格等于undefined时才生效
```
var {name,age=22}={name:"SunnyChuan"} //name="SunnyChuan" age=22
var {name,age=22}={name:"SunnyChuan",age:null} //name="SunnyChuan" age=null
```

解构赋值最常用的地方就是交换变量。不需要额外的临时变量temp，一句代码就搞定
```
[x,y]=[y,x]
```

### 字符串/数值/数组的扩展
#### 字符串的扩展
includes(str)，返回布尔值，是否在当前字符串中找到了str
```
"SunnyChuan".includes("Chuan"); //true
```

startsWith(str)，返回布尔值，当前字符串是否以str开头
```
"SunnyChuan".startsWidth("Sunny"); //true
```

endsWith(str)，返回布尔值，当前字符串是否以str结尾
```
"SunnyChuan".endsWith("Chuan"); //true
```

repeat(n)，将当前字符串重复n次（可以理解为在原有的基础上重复n-1次）并返回新字符串，如果n是小数则向下取整。当n小于等于0时，返回空字符串
```
"x".repeat(3); // "xxx"
```

模板字符串
```
var name="SunnyChuan";
var str=`hello ${name},this is ES6 `;
//等同于"hello"+name+",this is ES6";
```

#### 数值的扩展
Number.isFinite(n)，布尔值，检查n是否有穷，与es5的isFinite()不同的是，它不会进行类型转换，因此只对真正的数值有效
```
Number.isFinite(10); //true
Number.isFinite(NaN); //false
Number.isFinite(Infinity); //false
Number.isFinite("10"); //false
```

Number.isNaN(n)，布尔值，检查n是否是NaN，与es5的isNaN()不同的是，它不会进行类型转换，因此只对真正的NaN有效
```
Number.isNaN(NaN); //true
Number.isNaN("str"); //false
```

Number.isInteger(n)，布尔值，检查n是否是整数，需要注意1.0和1是同一个值
```
Number.isInteger(1); //true
Number.isInteger(1.0); //true
Number.isInteger(1.2); //false
```

 Math.trunc(n)，去除小数部分并返回（正数向下取整，负数向上取整）
 ```
 Math.trunc(1.2); //1
 Math.trunc(-1.2); //-2
 ```

Math.sign(n)，布尔值，n是正数返回+1，是负数返回-1，0返回0，-0返回-0，其他值返回NaN
```
Math.sign(100); //+1
Math.sign(-100); //-1
Math.sign(0); //0
Math.sign(-0); //-0
Math.sign("str"); //NaN
```

#### 数组的扩展
Array.from(list)，将list转化成真正的数组，常用于将nodeList/arguments等伪数组
转化成数组
```
var obj={a:1,b:2,c:3};
Array.from(obj); //[1,2,3]
var div=document.getElementsByTagName("div");
Array.from(div); //[[object HTMLDivElement],[object HTMLDivElement],[object HTMLDivElement]]
```

includes(n)，布尔值，检查当前数组是否包含n，可以接收两个参数，第二个参数代表搜索的起始位置（负数代表从后往前）
```
[1,2,3,4].include(2); //true
[1,2,3,4].include(2,2); //false
[1,2,3,4].include(2,-3); //true
```

数组新增三种遍历方式：entries()（遍历key和value）/keys()（遍历key）/values()（遍历value）
```
for(let v of ["a","b","c","d"].values()){
	console.log(v);
} //"a" "b" "c" "d"

for(let k of ["a","b","c","d"].keys()){
	console.log(k);
} //0 1 2 3

for(let [k,v] of ["a","b","c","d"].entries()){
	console.log(k,v);
} //0 "a" 1 "b" 2 "c" 3 "d"
```
### 函数和对象的扩展
#### 函数的扩展
es6支持函数的参数设置默认值，但是这些参数必须位于函数的尾部，并且参数的变量是默认声明的，因此不能用let/const再次声明
```
function func(x,y=5){
	console.log(y); //5
	let y=3; //error
}
```

扩展运算符，用于将数组（伪数组）展开成参数序列
```
console.log(...[1,2,3]); //1 2 3
[...document.getElementsByTagName("div")]; //[[object HTMLDivElement],[object HTMLDivElement],[object HTMLDivElement]]
```

扩展运算符有很多应用场景，这里列举三个
1. 数组合并
```
[1,2,3].concat([4,5,6]); //es5
[1,2,3,...[4,5,6]]; //es6
```

2.与解构结合（用于生成数组）
```
let [a,b,...c]=[1,2,3,4,5]; //a=1,b=2,c=[3,4,5] 扩展运算符必须放在最后
```

3.Math.max的参数不支持数组，在es5中需要用到apply。用扩展运算符可以解决该问题
```
Math.max.apply(Math,[1,2,3]); //es5
Math.max(...[1,2,3]); //es6 相当于Math.max(1,2,3)
```

箭头函数，如果不需要参数则用圆括号代替参数部分，如果函数体有多行代码则用大括号括起来
```
var func=x=>x*2; //var func=function(x){return x*2;}
var func=()=>true; //var func=function(){return true;}
var func=(x,y)=>{var z=x+y;console.log(z);} //var func=function(x,y){var z=x+y;console.log(z);}
var func=()=>({a:1,b:2}); //返回对象需要用圆括号包起来，圆括号代表函数体，花括号代表这是一个对象
```

函数绑定，使用a::b的形式取代传统的bind/call/apply
```
a::b; //b.bind(a);
a::b(...arguments); //b.apply(a,arguments);
```
#### 对象的扩展
es6允许在对象中只写key，这样默认了value等于key所代表的变量值
```
var [name,age]=["SunnyChuan",22];
var obj={name,age}; //obj={name:"SunnyChuan",age:22} 
//相当于obj={name:name,age:age};
```

对象的方法也可以简写
```
var obj={
	func(){
		//函数体
	}
}
//相当于
var obj={
	func:function(){
		//函数体
	}
}
```

Object.assign(obj,obj1,obj2,...)，将obj1/obj2/...与obj进行拼接（修改obj本身）
```
var [obj,obj1,obj2]=[{a:1},{b:2},{c:3}];
Object.assign(obj,obj1,obj2);
console.log(obj); //{a:1,b:2,c:3}
```

Object.is(obj1,obj2)，布尔值，判断两个值是否严格相等，与===不同的是，+0不等于-0，NaN等于NaN
```
Object.is(100,100); //true
Object.is(NaN,NaN); //true
Object.is(+0,-0); //false
Object.is({a:1},{a:1}); //false
```