---
layout: post
title:  初探微信小程序之小川天气
date:   2018-08-29 13:32:20 +0300
description: 
catalog: true
header-img: /img/xiaochuan_weather.jpg # Add image post (optional)
tags: 
    - 博客
    - 技术博客
    - 小程序
categories: [技术博客]
ascription: technology
author: # Add name author (optional)
---
微信小程序可以说是当下很火热的应用了。在好奇心的作用下，通读了一遍之后便开始上手撸了一个很简单的天气应用——小川天气。

![小川天气](https://upload-images.jianshu.io/upload_images/1495096-6895c3ecc8731c6c.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![首页](https://upload-images.jianshu.io/upload_images/1495096-6b52de2058465306.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![生活指数](https://upload-images.jianshu.io/upload_images/1495096-097faac85ddac982.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![导航](https://upload-images.jianshu.io/upload_images/1495096-fb080aab4403782b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![城市选择](https://upload-images.jianshu.io/upload_images/1495096-1239aa459b6252ae.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

目前已经发布正式版本，你可以直接在微信中搜索`小川天气`来使用。

小程序的文档已经很完善了，通读一遍后上手写代码并不是很难，下面主要讲一下本人在开发微信小程序的一些感触（坑）~

#### SetData
类似于`React`的`setState`，用于更改状态，但小程序做得更加智能，你完全可以直接修改数组中的某一项的值而不必再克隆一份出来：
```
//小程序
this.setData({
    "arr[0]": "new value"
})
//react
let _arr = JSON.parse(JSON.stringify(this.state.arr));
_arr[0] = "new value";
this.setState({
    arr: _arr
})
```

修改对象也是如此：
```
//小程序
this.setData({
    "obj.name": "mescal-chuan"
})
//react
let _obj = JSON.parse(JSON.stringify(this.state.obj));
_obj.name = "mescal-chuan";
this.setState({
    obj: _obj
})
```
#### 页面中绑定函数
使用`bind`指令进行函数绑定：
```
 <view bindtap="openModal"></view>
```
看起来和`vue`差不多，然而如果你想给这个函数传递一些参数就比较恶心了：
```
 <view bindtap="openModal" data-index="{{index}}"></view>
//js
openModal() {
    const currentIndex = e.currentTarget.dataset.index;
    this.setData({
      currentIndex
    })
    wx.showModal({
        title: config.lifeStyleTitle(activity[currentIndex].type),
        content: activity[currentIndex].txt,
        showCancel: false
    })
}
```


#### 页面通信
以本应用为例，用户在城市选择页选择新的城市之后，需要通知首页刷新数据。在单页应用中我们可以使用`redux`或`vuex`这类状态管理库实现状态的全局化；在使用`dcloud`这类`webapp`开发时，我们可以用它们自己内部定制的事件传递来实现跨页面通信。那么在微信小程序中，你可以获取一个页面的实例，然后直接调用该页面内部的方法。
```
//city.js
selectCity(e) {
    const keyIndex = e.currentTarget.dataset.key;
    const childIndex = e.currentTarget.dataset.child;
    const pages = getCurrentPages();
    const homePage = pages[0];
    //调用首页的方法来刷新首页数据
    homePage.getNewCity(null, keyIndex, childIndex);
    wx.navigateBack();
}
```

#### 指令中使用函数
很遗憾，一个页面内部的函数只能用在事件绑定中，你必须使用`wxs`来实现在页面中访问内部函数。它是小程序的一套脚本语言，配合我们的`wxml`一起完成页面构建。

```
//index.vue
<template>
    <p>{{getDateText(index)}}</p>
</template>
<script>
    export default {
        ...
        methods: {
            getDateText(index) {
                return xxx;
            }
        }
        ...
    }
</script>
```

```
//index.wxs
var getDateText = function(index) {
    return xxx;
}
module.exports = {
    getDateText: getDateText
}
//index.wxml
<wxs src="./index.wxs" module="forecast"/>
<text>{{forecast.getDateText(index)}}</text>
```
#### UI
小程序的组件到目前为止还不是很完善，一些比较常用的组件需要自己实现，比如：抽屉、模态框、Loading等。当然，`github`上也有一些比较成熟的UI库供你直接使用，比如本项目中用到的[小程序版echarts](https://github.com/ecomfe/echarts-for-weixin)。

#### 总结
目前为止遇到的坑大致如此，由于只是开发了一个简单应用，所涉及到的知识只是皮毛，后续有时间会继续开发一个复杂应用。