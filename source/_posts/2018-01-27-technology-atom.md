---
layout: post
title: 优雅地配置Atom
date: 2018-01-27 13:32:20 +0300
description: 
catalog: true
header-img: /img/5.jpg # Add image post (optional)
tags: 
    - 博客
    - 技术博客
    - 工具
categories: [技术博客]
ascription: technology
author: # Add name author (optional)
---
从最初的sublime text，到webstorm，后来又转战visual studio code，直到现在的atom，就个人使用体验来看是越来越舒适的。之前一直在使用sublime text和webstorm，后来尝试了一把vs code，发现其插件安装非常方便，主题也很优雅，于是就将vs code作为常用开发工具。最近vs code经常出现智能提示消失的现象，特别是当代码中有语法错误之后，除非重启，否则就跟用记事本没什么区别，可能是插件本身的问题，等过一阵子再尝试。昨天花了整整一个下午的时间体验atom，它的插件安装和vs code一样方便，社区也很活跃，下面是我最终的配置结果：

![atom](http://upload-images.jianshu.io/upload_images/1495096-fc5ef474d3f88148.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 基本配置
前往[atom官网](https://atom.io/)下载最新版本：

![](http://upload-images.jianshu.io/upload_images/1495096-ea76ed55986ebc76.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

安装成功后，根据个人喜好做一些基本配置。我个人喜欢将tab缩进长度设置为4，这样代码看起来不是那么紧凑。

![](http://upload-images.jianshu.io/upload_images/1495096-d7e6b3cb4148f94b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Atom默认是不显示缩进线的，你需要手动勾选`show indent guide`。
![](http://upload-images.jianshu.io/upload_images/1495096-4f1c57ac4c68c700.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 主题

推荐`atom material`和`seti-ui`，但我个人更喜欢`atom material`这种扁平化的风格，编辑器嘛就使用默认的`one dark`，两者搭配起来使用效果更好。直接在`settings -> install`中输入关键字，然后点击安装即可。

![主题安装](http://upload-images.jianshu.io/upload_images/1495096-bccc9a2f38837ecf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
安装成功后，在`settings -> themes -> ui theme`中选择`atom msterial`即可切换主题。

#### 插件
这里罗列了一些经常用到的插件，参考了kompasim的[atom-plugins](https://github.com/kompasim)。插件的安装方法与主题相同，每个插件的具体配置都在github上有详细说明。
1. `atom-beautify` 格式化代码
2. `atom-ternjs` es5、es6、node、jQuery等代码自动补全
3. `pigments` 颜色代码片段的背景色以该颜色显示，效果如图：

![](http://upload-images.jianshu.io/upload_images/1495096-93b2ab8711d4b38c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4. `minimap` 实现sublime text的代码预览，效果如图：
![](http://upload-images.jianshu.io/upload_images/1495096-fee436966feb56e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

5. `autocomplete-modules` 模块自动补全。这个在es6开发中经常用到，会智能显示当前路径下的模块，搭配`autocomplete-paths`一起使用，效果如图：
![](http://upload-images.jianshu.io/upload_images/1495096-15eb37a6942a6aeb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

6. `autocomplete-paths` 路径智能提示，它的默认项目最大文件数为2000，当超过这个数量时插件不再运行。目前的前端项目2000+的文件已经再正常不过了（包含了node_modules），可以在`autocomplete-paths`的设置中修改：

![](http://upload-images.jianshu.io/upload_images/1495096-13a941a8e8e13508.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

不过并不建议修改该默认值，如果你的电脑性能不是很高的话重启atom后会十分卡顿。
7. `file-icons` 为文件添加小图标，效果如图：

![](http://upload-images.jianshu.io/upload_images/1495096-13776b2743800479.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

8. `atom-html-preview` 预览html页面
9. `js-hyperclick` ctrl+鼠标左键跳转到变量定义处，它依赖于其他插件，当出现提示框时点击确认让其自动安装即可。
10. `linter` 基本的错误检查，推荐在其之上安装更精准的错误检查插件
11. `linter-eslint` js错误检查，比`linter-jshint`更容易配置和实用。推荐将.eslintrc放到`c:\Users\用户名\`下，在`linter-eslint`的设置中配置路径：

![](http://upload-images.jianshu.io/upload_images/1495096-fcd1def768721329.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

eslint的配置请参考[官方文档](https://eslint.org/docs/user-guide/configuring)
```
//.eslintrc
{
    "env": {
        "browser": true,
        "node": true,
        "commonjs": true,
        "es6": true
    },
    "parserOptions": {
        "ecmaVersion": 6,
        "sourceType": "module",
        "ecmaFeatures": {
            "jsx": true
        }
    }
}
```
![](http://upload-images.jianshu.io/upload_images/1495096-9592016bab9c6911.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

目前的配置并不支持es7语法：

![](http://upload-images.jianshu.io/upload_images/1495096-497ea35c4bb739c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

需要在.eslintrc中添加`"parser": "babel-eslint",`，然后linter-eslint会抛出如下错误：

![](http://upload-images.jianshu.io/upload_images/1495096-06fc9f9d9c9e2ca8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
我们点开错误信息，进入到linter-eslint文件中，安装`babel-eslint`：

![](http://upload-images.jianshu.io/upload_images/1495096-3c2efcabae8b0959.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
cd /c/Users/qinchuana/.atom/packages/linter-eslint
npm install babel-eslint --save
```
重启atom，一切ok了。
12. `terminal-plus` 内嵌控制台
13. `highlight-selected` 高亮显示相同的单词，效果如图：

![](http://upload-images.jianshu.io/upload_images/1495096-75229de63e42469b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


14. `docblockr` 快速编写注释文档
15. `language-babel` jsx自动编译
16. `language-javascript-jsx` 支持jsx语法
17. `emmet-jsx-css-modules` jsx中的css emmet
18. `atom-react-autocomplete` react的智能提示
19. `atom-react-es6-snippets` 快速生成es6写法的react片段
20. `react-native-snippets` 快速生成react native片段
21. `atom-react-native-style` 快速书写rn样式，效果如图：

![](http://upload-images.jianshu.io/upload_images/1495096-0c39980283a721ab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

22. `activate-power-mode` 一个特效插件，当连击数达到一定值后每敲一次键盘都会有颗粒特效和震动，效果如图：
![](http://upload-images.jianshu.io/upload_images/1495096-4f4c7fecf351593a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)








