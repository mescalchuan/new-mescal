---
layout: post
title: 用ES6书写React
date: 2016-11-03 13:32:20 +0300
description: 
cover: /img/mescal/10.jpg # Add image post (optional)
tags: 
  - 博客
  - 技术博客
  - javascript
  - react
categories: [技术博客]
ascription: technology
author: # Add name author (optional)
---
前段时间学习react+redux的时候发现现在大家基本上都是在用es6的方式编写react代码。由于习惯了react本身的编写方式，起初看得云里雾里。当自己实践过后发现es6+react确实能够让代码更加简洁，提高开发效率。
### 起步
首先需要安装react
```
npm install react react-dom --save-dev
```

使用es6（7）需要babel转换器，本人使用的是webpack自动化工具
```
npm install webpack --save-dev
npm install babel-preset-es2015 --save-dev //转换至es5
npm install babel-preset-react --save-dev //转换jsx至es5
npm install babel-preset-stage-2 --save-dev //支持es7
npm install babel-loader --save-dev //webpack的babel加载器
```

配置webpack
```
//webpack.config.js
module.exports={
	entry:"./main.js",
	output:{
		path:__dirname,
		filename:"app.js"
	},
	module:{
		loaders:[{
			test:/\.js$/,
			exclude:/node_modules/,
			loader:"babel-loader",
			query:{
				presets:["es2015","stage-2","react"]
			}
		}]
	}
}
```

现在准备工作做好了，接下来的任务就是用新的方式编写入口文件。（main.js）
### ES6-React
#### 加载模块
通过引入react.js和react-dom.js我们可以直接使用React和ReactDOM这两个对象，这是传统的方式
```
//html中需要先引入react.js和react-dom.js
console.log(React);
console.log(ReactDOM);
```
现在我们使用es6的方式
```
//不需要提前引入任何文件
import React from "react";
import ReactDOM from "react-dom";
console.log(React);
console.log(ReactDOM);
```
#### 创建组件
使用类来创建组件代替React.createClass
```
import React from "react";
class MyComponent extends React.Component{
//组件内部代码
}
```
你也可以用另一种方式
```
import React,{Component} from "react";
class MyComponent extends Component{
//组件内部代码
}
```

#### State/Props/PropTypes
```
//传统的react
var MyComponent=React.createClass({
  getDefaultProps:function(){
    return {
      name:"SunnyChuan",
      age:22
    };
  },
  propTypes:{
    name:React.PropTypes.string.isRequired,
    age:React.PropTypes.number.isRequired
  }
});
```
es6允许将props和propTypes当作静态属性在类外初始化
```
class MyComponent extends React.Component{}
MyComponent.defaultProps={
  name:"SunnyChuan",
  age:22
};
MyComponent.propTypes={
  name:React.PropTypes.string.isRequired,
  age:React.PropTypes.number.isRequired
};
```
es7支持直接在类中使用变量表达式，这也是我推荐的写法
```
class MyComponent extends React.Component{
  static defaultProps={
    name:"SunnyChuan",
    age:22
  }
  static propTypes={
    name:React.PropTypes.string.isRequired,
    age:React.PropTypes.number.isRequired
  }
}
```
state和前两个不同，它不是静态的
```
class MyComponent extends React.Component{
  static defaultProps={
    name:"SunnyChuan",
    age:22
  }
  state={
     isMarried:false
  }
  static propTypes={
    name:React.PropTypes.string.isRequired,
    age:React.PropTypes.number.isRequired
  }
}
```
#### 函数
React.createClass本身接收的是一个对象，对于对象中的方法，es6允许使用`key(){}`的形式取代`key:function(){}`
```
class MyComponent extends React.Component{
  state={
    count:0
  }
  handleChange(){
    this.setState({count:this.state.count+1});
  }
}
```
需要注意的是，由于使用class创建组件，react不会再自动帮我们绑定作用域了，我们需要自己手动解决
```
class MyComponent extends React.Component{
  state={
    count:0
  }
  handleChange(){
    this.setState({count:this.state.count+1});
  }
  render(){
    return (
      <div>
        <h2>当前计数是：{this.state.count}</h2>
        <button onClick={this.handleChange.bind(this)}>点击</button>
      </div>
    )
  }
}
```
如果你觉得这种每次都需要绑定的方法太麻烦，也可以在构造函数中去绑定
```
class MyComponent extends React.Component{
  constructor(props){
    super(props);
    this.handleChange=this.handleChange.bind(this);
  }
  state={
    count:0
  }
  handleChange(){
    this.setState({count:this.state.count+1});
  }
  render(){
    return (
      <div>
        <h2>当前计数是：{this.state.count}</h2>
        <button onClick={this.handleChange}>点击</button>
      </div>
    )
  }
}
```
如果你觉得这种方式也麻烦，可以使用es6的箭头函数（自动绑定作用域），但是前提是你的环境要支持es7，因为箭头函数相当于表达式声明函数的简写，只有es7支持在类中这么使用（类中使用表达式state/props/propTypes也只有es7支持）
```
class MyComponent extends React.Component{
  state={
    count:0
  }
  handleChange=()=>{
    this.setState({count:this.state.count+1});
  }
  render(){
    return (
      <div>
        <h2>当前计数是：{this.state.count}</h2>
        <button onClick={this.handleChange}>点击</button>
      </div>
    )
  }
}
```

#### 组件生命周期
所有的组件生命周期都可以当作普通函数使用上述三种方式编写，componentWillMount比较特殊，它还可以在构造函数中编写
```
class MyComponent extends React.Component{
  componentWillMount(){
    console.log("Hello SunnyChuan");
  }
}
//二者等价
class MyComponent extends React.Component{
  constructor(props){
    console.log("Hello SunnyChuan")
  }
}
```

#### 扩展操作符
使用react开发最常见的问题就是父组件要传给子组件的属性较多时比较麻烦
```
class MyComponent extends React.Component{
//假设MyComponent已经有了name和age属性
  render(){
    return (
      <SubComponent name={this.props.name} age={this.props.age}/>
     )
  }
}
```
使用扩展操作符可以变得很简单
```
class MyComponent extends React.Component{
//假设MyComponent已经有了name和age属性
  render(){
    return (
      <SubComponent {...this.props}/>
     )
  }
}
```
上述方式是将父组件的所有属性都传递下去，如果这其中有些属性我不需要传递呢？也很简单
```
class MyComponent extends React.Component{
//假设MyComponent有很多属性，而name属性不需要传递给子组件
  var {name,...MyProps}=this.props;
  render(){
    return (
      <SubComponent {...Myprops}/>
     )
  }
}
```
上述方法最常用的场景就是父组件的class属性需要被单独提取出来作为某个元素的class，而其他属性需要传递给子组件。

#### 模块化开发组件
说了这么多，个人认为es6+react最吸引人的地方就是模块化开发，将每个小（大）组件当作一个模块
```
//father.js
import React from "react";
import ReactDOM from "react-dom";
import {SonComponent} from "son.js";
class FatherComponent extends React.Component{ 
  //省去中间的业务逻辑
  render(){
    return (<SonComponent/>);
  }
}
ReactDOM.render(<FatherComponent/>,document.getElementById("ss"));
```
```
//son.js
import React from "react";
class SonComponent extends React.Component{
  //省去中间的业务逻辑
  render(){
    return (<h2>"SunnyChuan"</h2>);
  }
}
export {SonComponent};
```
如果你把子组件的导出设置为default export，那么在导入时就不必再加{}
```
//father.js
import SonComponent from "son.js";
```
```
//son.js
export default class SonComponent extends React.Component{
  //省去中间的业务逻辑
}
```

花一点时间学习es6（7）+react的开发方式，你会更加喜欢使用react。