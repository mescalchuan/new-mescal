---
layout: post
title: 基于Express搭建前后端分离的开发环境
date: 2017-04-26 13:32:20 +0300
description: 
catalog: true
header-img: /img/9.jpg # Add image post (optional)
tags: 
    - 博客
    - 技术博客
    - node
categories: [技术博客]
ascription: technology
author: # Add name author (optional)
---
对于一个前端开发人员来说，node.js可以说是开发后端的福音。本人毕设项目使用的是asp.net（只是使用ashx来接收前端的ajax，并没有使用那些恶心的控件），通过对比发现整个项目都使用js实在是幸福。

### Node.js
关于node.js的优点就不再多说，网上一搜一大堆，项目也可以找到很多。这里主要说一下node.js关于搭建web应用这一块的缺点。
众所周知，node.js需要自己搭建服务器，也就是说你需要自己监听一个端口并启动它
```
var http=require('http');
http.createServer(function(req,res) {
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.write("hello sunnychuan");
    res.end();
}).listen(3000);
console.log("server is running!");
```
这里有一本电子书帮助你使用原生的node.js搭建web应用，[点击这里](http://www.nodebeginner.org/index-zh-cn.html#a-basic-http-server)

不仅仅是监听端口，我们往往会根据用户输入的url从本地找到相应文件并将它写回请求
```
fs.exists(filePath,function(exists){
    if(exists){
       res.writeHead(200,{'Content-Type':contentType});
       var stream=fs.createReadStream(filePath);
       stream.on('error',function(){
          res.writeHead(500,{'Content-Type':'text/html'});
          res.end('<h1>500 Server Error</h1>');
       });
       stream.pipe(res);
    }
}
```
我们还会针对前端的ajax请求做路由处理
```
//根据不同的路径调用不同的处理程序
var handle={};
handle["/api/get.json"]=requestHandlers.get;
handle["/api/post.json"]=requestHandlers.post;
```
```
function route(handle,pathname,req,res){
    if(typeof handle[pathname]==='function'){
        handle[pathname](req,res);
    }
    else{
        res.writeHead(404,{'Content-Type':'text/html'});
        res.write('<h1>404 Not Found</h1>');
        res.end();
    }
}
```
```
var pathname=decodeURI(url.parse(req.url).pathname);
if(path.extname(pathname)=='.json'){
    route(handle,pathname,req,res);
}
```
通过上面那么一折腾，原本的热情就消退了一半，这里我推荐一下express框架，官方文档在[这里](http://www.expressjs.com.cn/)

### Express
#### 安装和目录结构
首先安装express
```
npm install express --save-dev
```
安装express的应用生成器
```
npm install express-generator -g
```
使用它的应用生成器可以很快建立一个项目
```
express myapp
```
然后进入到该文件夹下，安装所有依赖
```
cd myapp
npm install
```
你的目录结构长这个样子

![](http://upload-images.jianshu.io/upload_images/1495096-d25565cc3ad97c09.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

bin是启动项，在bin/www里面你可以修改端口号
node_modules用来存放npm安装的模块
public里面用来存放资源
routes是路由控制，存放了所有的处理程序
views是视图文件，相当于html页，在routes中会使用类似
```
router.get('/', function(req, res, next) {
  res.render('index', { title: 'Express' });
});
```
这样的代码来替换模板引擎里的内容，有点像java的.jsp文件和asp.net的.asp文件
app.js用来配置express，你可以在这里进行路由分配和错误处理并按需加载中间件

#### 启动项目
```
npm start
```

![](http://upload-images.jianshu.io/upload_images/1495096-4d651e6318d58b6f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 修改代码
作为前端开发的你，也许会厌恶由后端直接将数据渲染在页面上（至少我是这样），我们通常使用ajax来进行前后端交互并自行处理后端的数据，因此整个views文件就可以直接delete掉了。

step1 更改目录结构
把public更名为client，新建server文件夹并把routes和app.js放到里面（views直接删除即可），这样一来整个项目结构就比较明显了，客户端和服务端分离。

![](http://upload-images.jianshu.io/upload_images/1495096-9537205523283645.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

step2 修改routes文件夹
routes里默认有index.js和users.js两个路由文件，其中index.js是负责模板渲染的
```
var express = require('express');
var router = express.Router();

/* GET home page. */
router.get('/', function(req, res, next) {
  res.render('index', { title: 'Express' });
});

module.exports = router;
```
而users.js是直接返回一个字符串
```
var express = require('express');
var router = express.Router();

/* GET users listing. */
router.get('/', function(req, res, next) {
  res.send('respond with a resource');
});

module.exports = router;

```
既然我们删掉了views，那么index.js的代码肯定要修改的。我删掉了这两个文件，并新建了get.js和post.js文件用来接收和处理前端的ajax请求
```
//get.js
module.exports=function(req,res){
  console.log(req.query.userName);
  res.json({code:200,data:"this is a get request"});
}
```
```
//post.js
module.exports=function(req,res){
    console.log(req.body.userName);
    console.log(req.body.password);
    res.json({code:200,data:"this is a post request"});
}
```
另外，我还新建了一个名为router.js的文件，用来分配路由（默认是在app.js文件中进行路由分配的，但是一旦请求过多，app.js会十分冗长）
```
var express=require('express');
var router=express.Router();

var get=require('./get');
var post=require('./post');

router.get('/get.json',get);
router.post('/post.json',post);
module.exports=router;
```
step3 修改app.js
对模块加载做如下修改
```
//修改前
var index = require('./routes/index');
var users = require('./routes/users');
//修改后
var router=require('./routes/router.js');
```
删掉视图引擎的设置
```
//删除下面的代码
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'jade');
```
对静态资源的加载做一些更改，默认是在"public"文件夹下读取并加载静态资源的，现在我们设置为先从根路径读取文件，如果不存在则从"client"中读取文件
```
//修改前
app.use(express.static(path.join(__dirname, 'public')));
//修改后
var rootDir=path.resolve(__dirname);
var projectDir=path.resolve(__dirname,'../','client');
app.use(express.static(rootDir));
app.use(express.static(projectDir));
```
替换默认的路由加载，改为上面require的router。/api是一个虚拟路径，这样每当我们访问/api/xxx.json就会通过路由来分配不同的控制器完成业务逻辑
```
//修改前
app.use('/', index);
app.use('/users', users);
//修改后
app.use('/api',router);
```
我们还需要让根路径加载首页面
```
app.get('/', function(req, res){
  res.sendFile(projectDir+'/index.html');
});
```
由于不需要渲染视图，因此错误页面的处理也要进行修改，我这里仅仅是把错误信息返回给前端
```
//修改前
app.use(function(err, req, res, next) {
  res.locals.message = err.message;
  res.locals.error = req.app.get('env') === 'development' ? err : {};
  res.status(err.status || 500);
  res.render('error');
});
//修改后
app.use(function(err, req, res, next) {
  res.locals.message = err.message;
  res.locals.error = req.app.get('env') === 'development' ? err : {};
  res.status(err.status || 500);
  res.json({code:500,msg:"error"});
});
```

#### 测试
我们需要编写前端代码进行测试，这里我使用angular简单地发送get和post请求
```
angular.module("indexApp",[])
.controller("IndexCtrl",function($scope,$http){
    $scope.get=function(){
        $http({
            method:"get",
            url:"/api/get.json",
            params:{userName:"sunnychuan"}
        }).then(function(res){
            if(res.data.code==200){
                console.log(res.data.data);
            }
            else if(res.data.code==500){
                console.log(res.data.msg);
            }
        })
    }
    $scope.post=function(){
        $http({
            method:"post",
            url:"/api/post.json",
            data:{userName:"sunnychuan",password:"qc"}
        }).then(function(res){
            if(res.data.code==200){
                console.log(res.data.data);
            }
            else if(res.data.code==500){
                console.log(res.data.msg);
            }
        })
    }
})
```
同样的，使用npm start启动项目，在url中输入localhost:3000

![](http://upload-images.jianshu.io/upload_images/1495096-95f59140377630af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击get按钮

![](http://upload-images.jianshu.io/upload_images/1495096-c76452e4513b8840.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![](http://upload-images.jianshu.io/upload_images/1495096-e4c897f0f0c7c1e2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击post按钮

![](http://upload-images.jianshu.io/upload_images/1495096-f69ba35095f8f03d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/1495096-e627f3d9e6e7e25d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

至此，基本的环境搭建成功，搭配mongodb会在之后的文章中介绍。