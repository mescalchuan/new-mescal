---
layout: post
title: React Native之巧用TextInput
date: 2018-01-19 13:32:20 +0300
description: 
cover: /img/mescal/6.jpg # Add image post (optional)
tags: 
    - 博客
    - 技术博客
    - react-native
categories: [技术博客]
ascription: technology
author: # Add name author (optional)
---
近日使用rn开发电商app遇到一个再简单不过的需求：买家申请退款，填写申请单，其中有一个多行文本输入框用于填写退款理由，如图所示：

![](http://upload-images.jianshu.io/upload_images/1495096-969a3263e2eafdf3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

看似简单的需求，然而用rn开发还是会遇到问题。
#### 页面搭建
两个View和一个TextInput足矣，外层View白色背景色，内层View提供边框。如果你在安卓环境下运行项目会发现TextInput下面有一条黑线，将underlineColorAndroid设置为true既可解决。同时，将paddingVertical设置为0解决TextInput文字不垂直居中的问题。
```
<View style = {styles.container} >
    <View style = {styles.inputContainer} >
        <TextInput
            placeholder = {'输入退款说明'} 
            placeholderTextColor = {'#BBBBBB'}
            underlineColorAndroid = {'transparent'} 
            style = {{paddingVertical: 0, paddingLeft: 5, fontSize: 16}}
         />
    </View>
</View>
```
运行结果如图所示：

![](http://upload-images.jianshu.io/upload_images/1495096-3f58b17349a4ee10.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 问题
我们更改一下TextInput的背景色，发现此时TextInput的高度为一行文字的高度，并且，当输入较长的内容后，文字仍然排成一行而不是单起一行。

![](http://upload-images.jianshu.io/upload_images/1495096-cdb26eaea5123f2e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

既然这样，我们将TextInput的高度设置为其外层的容器高度，并指定其为多行文本：
```
<TextInput
    placeholder = {'输入退款说明'} 
    placeholderTextColor = {'#bbbbbb'}
    underlineColorAndroid = {'transparent'} 
    multiline
    style = {{paddingVertical: 0, paddingLeft: 5, fontSize: 16, height: 105}}
/>
```
效果如图：

![](http://upload-images.jianshu.io/upload_images/1495096-c621fd4b77fe63b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

现在，整个内层容器都是文本框了，但是文本框的内容却是垂直居中的，很明显与设计不符。通过翻阅rn的源码发现，TextInput是原生代码实现的，前端将justifyContent设置为flex-start仍然不能解决该问题。产品经理也肯定不会接受这种解决方案。

#### 解决方案
思路就是动态修改TextInput的高度，其高度由内容行数决定，这样就能保证内容始终是紧靠顶端的。然后，为TextInput的外层容器绑定onPress事件，当点击容器时自动聚焦TextInput，形成“整个容器都是TextInput”的假象。

```
constructor(props) {
    super(props);
    this.state = {
        height: 30
    }
}

cauculateHeight(e) {
    const height = e.nativeEvent.contentSize.height > 30 ? e.nativeEvent.contentSize.height : this.state.height;
    this.setState({height});
}
```
```
<TouchableOpacity 
    activeOpacity = {1}
    style = {styles.inputContainer} 
    onPress = {() => this.TextInput.focus()} 
>
    <TextInput
        placeholder = {'输入退款说明'} 
        placeholderTextColor = {'#bbbbbb'}
        underlineColorAndroid = {'transparent'} 
        multiline
        ref = {textInput => this.TextInput = textInput}
        onContentSizeChange = {e => this.cauculateHeight(e)}
        style = {[{paddingVertical: 0, paddingLeft: 5, fontSize: 16}], {height: this.state.height}]}
    />
</TouchableOpacity>
```

![](http://upload-images.jianshu.io/upload_images/1495096-451e5da7219e5ce1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/1495096-7997458637d44c77.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当内容超过容器高度后，超出的部分被隐藏了，给TextInput添加maxHeight样式解决该问题（maxHeight值为容器高度）。
```
<TextInput
    ...
    style = {[{paddingVertical: 0, paddingLeft: 5, fontSize: 16, maxHeight: 105}], {height: this.state.height}]}
/>
```

![](http://upload-images.jianshu.io/upload_images/1495096-5ac6da860fe48aba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)










