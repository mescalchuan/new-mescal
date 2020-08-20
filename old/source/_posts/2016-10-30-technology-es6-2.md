---
layout: post
title: ES6学习笔记（二）
date: 2016-10-30 13:32:20 +0300
description: 
cover: /img/mescal/11.jpg # Add image post (optional)
tags: 
	- 博客
	- 技术博客
	- javascript
categories: [技术博客]
ascription: technology
author: # Add name author (optional)
---
### Set和Map
#### Set
Set类似于数组，但是里面成员的值不能重复。Set构造函数可以传入一个数组用于初始化
```
var s=new Set();
[1,2,3].map(i=>s.add(i));
for(var i of s){console.log(s)}; //1 2 3

var s=new Set([1,2,3]);
for(var i of s){console.log(s)}; //1 2 3
```

Set拥有以下属性和方法
1.size，返回成员个数
2.add(value)，添加值，返回自身
3.delete(value)，删除值，返回布尔值，代表是否删除成功
4.has(value)，布尔值，表示是否含有value成员
5.clear()，无返回值，删除所有成员

遍历操作
1.keys()
2.values()
3.entries()
4.forEach()
由于Set中只有值，因此keys()等于values()
```
var s=new Set([1,2,3,4]);
for(var i of s)console.log(i); //1 2 3 4
for(var i of s.keys())console.log(i); //1 2 3 4
for(var i of s.values())console.log(i); //1 2 3 4
for(var [k,v] of s.entries())console.log(`${k} ${v}`); //1 1 2 2 3 3 4 4，key值等于value值 
s.forEach((v,k)=>{console.log(`${v} ${k}`)}); //1 1 2 2 3 3 4 4
```

通过Set可以很方便地实现数组去重而不必借用一次循环
```
var deleteSample=arr=>{var s=new Set(arr);return [...s];}
console.log(deleteSample([1,1,2,2,3,3,4])); //[1,2,3,4]
```

#### Map
对象只能使用字符串当作key，而Map可以使用任意类型作为key。Map构造函数可以传入一个数组用于初始化，该数组由若干个数组构成，每个数组包含key和value
```
var m=new Map(),obj={a:1};
m.set(obj,"key is Object"); //将obj作为key，value是一个字符串
var m=new Map([["a",1],["b",2],["c",3]]);
for(var [k,v] of m)console.log(`${k} ${v}`); //a 1 b 2 c 3
```

Map拥有以下属性和方法
1.size，返回成员个数
2.set(key,vaule)，设置key所对应的value，如果key存在则覆盖旧的value，如果不存在则新建key并赋值value。返回Map本身
3.get(key)，获取key所对应的value
4.has(key)，布尔值，判断Map中是否存在key键
5.delete(key)，删除key键，返回布尔值，表示是否删除成功
6.cleear()，没有返回值，删除所有成员

遍历操作
1.keys()
2.values()
3.entries()
4.forEach()
```
var m=new Map([["a",1],["b",2],["c",3]]);
for(var [k,v] of m)console.log(`${k} ${v}`); // a 1 b 2 c 3
for(var k of m.keys())console.log(k); //a b c
for(var v of m.values())console.log(v); //1 2 3
for(var [k,v] of m.entries())console.log(`${k} ${v}`); //a 1 b 2 c 3
m.forEach((v,k)=>{console.log(`${v} ${k}`)}); //1 a 2 b 3 c
```

通过扩展运算符可以将Map转换成数组
```
var m=new Map([["a",1],["b",2],["c",3]]);
console.log([...m.keys()]); //a b c
console.log([...m.values()]); //1 2 3
console.log([...m.entries()]); //[["a",1],["b",2],["c",3]]
console.log([...m]); //[["a",1],["b",2],["c",3]]
```

### Iterator和for...of循环
#### Iterator
Iterator为各种数据结构提供统一的遍历机制。在es6中，数组/伪数组/Set/Map具备原生的Iterator接口
```
let arr=[1,2,3];
var it=arr[Symbol.iterator]();
it.next(); //{value:1,done:false}
it.next(); //{value:2,done:false}
it.next(); //{value:3,done:false}
it.next(); //{value:undefined,done:true}
```
#### for...of循环
for...of循环内部调用的是Symbol.iterator方法，也就是说，只要是支持Symbol.iterator方法的数据结构就可以用for...of循环，与for...in不同的是，for...of可以返回键值和键名（默认数组只返回键值，可以通过keys()/values()/entries()进行修改），而for...in只能返回键名
```
var arr=[1,2,3];
for(var i in arr)console.log(i); //0 1 2
for(var i in arr)console.log(arr[i]); //1 2 3
for(var i of arr)console.log(i); //1 2 3
```

普通的对象不支持for...of循环，可以使用Object.keys方法将对象的键名生成一个数组，遍历这个数组。其实相当于for...in
```
var obj={a:1,b:2,c:3};
for(var k of Object.keys(obj))console.log(`${k} ${obj[k]}`); //a 1 b 2 c 3
for(var k in obj)console.log(`${k} ${v}`); //a 1 b 2 c 3
```

### Class
#### 创建Class和实例
es6支持类似于Java、C++等高级程序设计语言的类的写法，但是它的底层仍然使用的是es5的面向对象声明方式，相当于是语法糖，es6将定义在this上的属性使用es5的构造函数的方式，对方法使用原型的方式，和es5的混合模式是一样的
```
class MyClass{
	constructor(x,y){
		this.x=x;
		this.y=y;
	}
	print(){
		console.log(`(${this.x},${this.y})`);
	}
}
var myclass=new MyClass(2,3);
myclass.print(); //(2,3)
myclass.hasOwnProperty("x"); //true
myclass.hasOwnProperty("print"); //false
```

可以通过Object.assign为类添加方法
```
Object.assign(MyClass.prototype,{printName(){/**/},printAge(){/**/}});
```

Class也可以使用表达式的形式定义，无论是表达式形式还是一般形式，Class均不存在变量声明提升，必须先声明后使用
```
var MyClass=class{/**/};
var other=new OtherClass(); //ReferenceError
var OtherClass=class{/**/};
```

#### Class的继承
es6的Class继承和Java相似，使用extends关键字。必须在constructor中调用super方法，并且只有调用了super后才能使用this，也就是说super必须放在所有this的前面
```
class SubClass extends MyClass{
	constructor(x,y,z){
		super(x,y); //调用父类的构造函数
		this.z=z;
	}
	print(x,y,z){
		console.log(`(${this.x},${this.y},${this.z})`); //重写父类的print
	}
}
var sub=new SubClass(2,3,4;
console.log(sub instanceof SubClass); //true
console.log(sub instanceof MyClass); //true
```

可以通过object.getPrototypeOf方法从子类上获取父类，用于判断一个类是否继承自另一个类
```
console.log(Object.getPrototypeOf(SubClass)===MyClass); //true
```

es6允许继承原生构造函数（Boolean/Number/String/Array/Date/Function/RegExp）
```
class MyNumber extends Number{
	constructor(...agrgs){
		super(...args);
	}
	//添加一些方法
}
```

#### Class的取值和存值函数
es6支持类似于C#的get和get方法，但是调用它们的时候是以属性的形式调用，而不是方法
```
class MyClass{
	constructor(x){
		this.x=x;
	}
	get getX(){
		return this.x;
	}
	set setX(xx){
		this.x=xx;
	}
}
var myclass=new MyClass(2);
console.log(myclass.getX); //2
myclass.setX=3;
console.log(myclass.getX); //3
```

#### Class的静态方法
在一个方法上加上static关键字表示该方法是一个静态方法，它不能被实例所调用，只能直接通过类去调用
```
class MyClass{
	static print(){
		console.log("hello world");
	}
}
var myclass=new MyClass();
myclass.print(); //TypeError
MyClass.print(); //"hello world"
```

静态方法可以被子类继承，在子类中仍然是静态方法，可以通过super调用父类的静态方法
```
class SubClass extends MyClass{
	static print(){
		super.print();
	}
}
SubClass.print(); //"hello world"
```

### 模块的import和export
#### export
export命令用于规定模块的对外接口，允许其他模块通过import加载该模块
```
//export.js
export var obj={a:1,b:2,c:3};
export var name="SunnyChuan";
export var age=22;
```

如果觉得上面的形式较繁琐，可以简写为
```
var [obj,name,age]=[{a:1,b:2,c:3},"SunnyChuan",22];
export {obj,name,age};
```

模块输出的变量就是它的名字，可以通过as进行重命名
```
var [obj,name,age]=[{a:1,b:2,c:3},"SunnyChuan",22];
export {obj as o,name as n,age as a}; 
```

#### import
export用于导出模块，import用于加载模块，注意要加载的模块的名字必须与导出的模块的名字相同。import具有声明提升，会提升到整个代码块的头部
```
//import.js
import {obj,name,age} from "./export.js";
//如果export的时候已经重命名了，可以import {o,n,a} from "./export.js";
console.log(obj); //{a:1,b:2,c:3}
console.log(name); "SunnyChuan"
console.log(age); 22
```

同export，在import的时候也可以为模块重命名
```
import {obj as o,name as n,age as a} form "./exoprt.js"
```

使用“*”加载整个模块，然后用as指定给一个对象，通过调用对象的属性使用相应模块，也可以使用module命令取代import实现整体加载
```
import * as qc from "./exports.js";
console.log(qc.obj); //{a:1,b:2,c:3}
console.log(qc.name); "SunnyChuan"
console.log(qc.age); 22

module qc from "./exports.js";
console.log(qc.obj); //{a:1,b:2,c:3}
console.log(qc.name); "SunnyChuan"
console.log(qc.age); 22
```

以上是我经常用到的es6的知识点，想要学习更多强大的功能（Proxy/Reflect/Symbol/二进制数组/Generator/异步编程）或者想要更加细致地学习es6，可以访问阮一峰的教程[ECMAScript 6入门](http://es6.ruanyifeng.com/)，本人也是在这里学习的。