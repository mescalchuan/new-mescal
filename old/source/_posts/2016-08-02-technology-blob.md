---
layout: post
title: Blob对象
date: 2016-08-02 13:32:20 +0300
description: 
cover: /img/mescal/13.jpg # Add image post (optional)
tags: 
     - 博客
     - 技术博客
     - javascript
categories: [技术博客]
ascription: technology
author: # Add name author (optional)
---
前几天写公司的SpreadJS控件的测试用例时遇到一个需求，不通过input标签的type=file的形式来实现本地文件选择，而是直接指定自己想要上传的文件（因为模拟ui行为的测试会占用大量的时间，影响测试效率）。一开始我认为这是不可能实现的，因为浏览器的安全性原因，js是不能直接访问系统文件的（node.js除外）。ie浏览器支持使用FileSystemObject接口来访问本地文件，但是其他浏览器均没有实现。初步方案是利用后端语言去访问系统文件然后转换成base64格式返回给前端，前端将base64转成file文件。
#### 一、后端操作
后端语言可以很轻松地去访问本地文件，我将文件对象存入到byte数组中并转换成了base64的字符串传给前端。后端代码就不写了，很简单。
<br>
#### 二、前端操作
我想通过后端的base64字符串来生成File对象的实例。这里就需要了解input标签形式的文件上传的底层原理。
#### 1.input生成文件对象的原理
查阅w3c的文档得知：在 HTML 文档中` <input type="file"> `标签每出现一次，一个 FileUpload 对象就会被创建。

该元素包含一个文本输入字段，用来输入文件名，还有一个按钮，用来打开文件选择对话框以便图形化选择文件。当用户选择了一个或多个文件点击"打开"时，js会生成一个FileList数组来保存用户所选择的文件。FileList包含了一个或多个File对象。可以通过`document.getElemenById("uploadFile").files`来进行访问。处于安全性的考虑，File对象不会暴露出文件的内容，它只提供了一些只读的属性来描述修改时间、文件类型、长度、名字等。

![](http://upload-images.jianshu.io/upload_images/1495096-0895fa2984313aad.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<br>
#### 2.创建File对象
js有一个底层的Blob对象，它是一个包含有只读原始数据的类文件对象。File对象就继承自Blob对象并扩展了系统文件的支持。

Blob对象的构造函数（来自MDN）
```
Blob Blob(
  [optional] Array parts,
  [optional] BlobPropertyBag properties
);
```
第一个参数是一个数组，里面可以是DOMString，也可以是TypeArray或者Blob。
这里需要解释一下TypedArray：在JS语言中，数值只有一种称为Number的类型，而不像C语言或底层CPU指令那样区分是整型还是浮点型，是有符号的还是无符号的，是32位的还是64位的。Typed Array的主要是为了弥补JS处理二进制格式数据的不足，利用Typed Array可以非常方便地操作二进制的数据。TypedArray包括Int32Array、Uint8Array、Float32Array等，也可以是连续的内存缓冲区ArrayBuffer或者工具类DataView。
```
var myFile=new Blob([u8arr],{type : 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'});
```
第一个参数是一个Uint8Array的TypedArray类型，它是一个8比特的无符号的int数组，通过base64转换而来。第二参数是该文件的mime类型，不同的文件类型可以参考w3c的mime手册。
<br>
#### 3.base64的转换
还剩最后一个问题，如何把base64转成Uint8Array?
js提供了window.atob(base64)的方式对base64进行解码。再通过charCodeAt()的方式转成Uint类型。代码如下
```
var bytes=window.atob(base64),      //解码  
    n=bytes.length,
    u8arr=new Uint8Array(n),
while(n--){
    u8arr[n]=bytes.charCodeAt(n);   //将编码转换成Unicode编码
}
```
我对这一操作做了封装
```
//假设有一个xlsx文件
function createFile(urlData){
     var bytes=window.atob(urlData),
         n=bytes.length,
         u8arr=new Uint8Array(n);
     while(n--){
          u8arr[n]=bytes.charCodeAt(n);
     }
     return new Blob([u8arr],{type:'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'}});
}
var str="";   //str接收来自后端返回的base64
var myFile=createFile(str);
```
现在就大功告成了。前端的文件对象已经成功生成。
<br>
#### 4.最终测试
我在代码中创建一个链接去下载这个文件
```
var a = document.createElement("a");
a.href =window.URL.createObjectURL(myFile); //下载路径指向这个文件对象 
a.download = "SunnyChuan.xlsx"; 
a.click();    //指定页面自动下载文件
document.body.appendChild(a);
```
页面打开后会自动下载文件

![](http://upload-images.jianshu.io/upload_images/1495096-63f4eab0b989b958.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们把文件打开测试一下整个流程中有没有数据损失

![](http://upload-images.jianshu.io/upload_images/1495096-d471e0364e320580.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从图中可以看出文件完成无损地传到了前端。

这里有一个小插曲:用ie浏览器是无法下载的，因此需要做hack:、
```
if(navigator.appVersion.toString().indexOf(".NET")>0){
     window.navigator.msSaveBlob(myFile, "SunnyChuan.xlsx");
}
else{
        var a = document.createElement("a");
        a.href =window.URL.createObjectURL(myFile); 
        a.download = "SunnyChuan.xlsx"; 
        a.click(); 
        document.body.appendChild(a);
}  
```
附上完整的代码
```
function createFile(urlData,fileType){
     var bytes=window.atob(urlData),
         n=bytes.length,
         u8arr=new Uint8Array(n);
     while(n--){
          u8arr[n]=bytes.charCodeAt(n);
     }
     return new Blob([u8arr],{type:fileType});
}
var str="";   //str接收来自后端返回的base64
var fileType="";//指定mime
var myFile=createFile(str,fileType);
if(navigator.appVersion.toString().indexOf(".NET")>0){
     window.navigator.msSaveBlob(myFile, "SunnyChuan.xlsx");
}
else{
        var a = document.createElement("a");
        a.href =window.URL.createObjectURL(myFile); 
        a.download = "SunnyChuan.xlsx"; 
        a.click(); 
        document.body.appendChild(a);
}  
```
#### 三、总结
js一般是不可以直接访问本地文件的，但是可以通过一些间接的方式去实现。js的文件操作一般用于下载文件以及保存图片。我实习的公司就是通过将产品内部的图片转成base64，在需要使用的时候再转换回图片，这种方式不需要产品附带图片文件，产品最终只有一个js文件，极大地减少了产品尺寸。