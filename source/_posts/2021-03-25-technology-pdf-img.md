---
layout: post
title: 前端 PDF 预览方案
date: 2021-03-25 10:15:20 +0300
description:
catalog: true 
header-img: /img/2.jpg # Add image post (optional)
tags: 
    - 博客
    - 技术博客
    - javascript
    - vue
categories: [技术博客]
ascription: technology
author: # Add name author (optional)
---
### 产品需求描述
后端返回 pdf 文件链接，前端预览，要求不允许用户下载、复制、打印。

### 初步方案
1. 浏览器支持 pdf 文件预览功能，通过 window.open 的方式打开新的链接，效果如下：

![](https://upload-images.jianshu.io/upload_images/1495096-571b621b562bc937.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

问题：浏览器提供的下载、打印控件以及复制内容、右键下载等操作无法干预

2. 以 iframe 的方式加载文件，并禁用 iframe 的右键：
```html
<iframe ref="iframe" :src="pdfUrl" />
```
> 网上找到的方案大多为 document.oncontextmenu = function() { return false; }，但实测发现该方法仅适用于子页面内容没加载之前，如果资源加载完成则右键操作由子页面本身控制。

思路：在 iframe 加载成功后，为子页面注册对应的事件处理函数。

```js
let iframe = this.$refs.iframe
iframe.onload = () => {
    window.frames[0].contentDocument.oncontextmenu = () => false
}
```

问题：提示跨域

![](https://upload-images.jianshu.io/upload_images/1495096-ed6c54c19480dc5d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

原因分析：网页地址与资源链接的域名不一致，导致 iframe 跨域。

解决方案：由后端配置同源头解决跨域问题，但使用 iframe 无法解决用户复制文字的问题。

3. 使用 embed 标签，禁止右键：

```html
<embed :src="pdfUrl" enableContextMenu="false" />
<!-- 或者 -->
<embed :src="pdfUrl" oncontextmenu="window.event.returnValue=false" />
```

问题：仅在音视频资源下生效

### 思考

#### 分析
以上方案无法解决问题的原因在于利用了浏览器的默认特性，且这些特性是无法干预的。因此需要转换思路，将不可控的特性转换为已有的可控特性。

#### 思路
利用图片无法选择复制的特性，将 pdf 转成图片，并限制用户无法右键保存。

### 解决方案

基于 [pdf.js](https://mozilla.github.io/pdf.js/) 将 pdf 按页转换为一张张图片，通过 img 标签渲染，并禁用右键和图片拖拽。

#### Step1 读取 pdf 内容
```js
window.pdfjsLib.getDocument(pdfUrl)
```

由于资源链接不在本地，pdf.js 会报跨域的错误。

![](https://upload-images.jianshu.io/upload_images/1495096-f680f8408d6ef254.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

方案：参考[该链接](https://blog.csdn.net/l_ai_yi/article/details/82388497)，需要手动修改 pdf.js 的逻辑，并要求服务端配合解决跨域的问题。

修改 pdf.js 逻辑不利于后期升级和维护，因此我们换一种思路：基于 ajax 请求。

```js
axios.request({
    url: imgUrl,
    type: 'get',
    responseType: 'blob'
}).then(res => {
    window.pdfjsLib.getDocument(res.data)
})
```

成功解决跨域问题，并返回了 blob 对象，但在初始化 pdf.js 时报了如下错误：

![](https://upload-images.jianshu.io/upload_images/1495096-0068d147331e034b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

查询源码得知，pdf.js 不支持读取 blob 对象，因此需要将 blob 转为 url：
```js
window.pdfjsLib.getDocument(window.URL.createObjectURL(res.data))
```

#### Step2 解析文件，渲染到 canvas
调用 pdf.js 的 api 进行解析：
```js
window.pdfjsLib.getDocument(window.URL.createObjectURL(res.data)).promise.then(pdf => {
    // 解析第一页
    pdf.getPage(1).then(page => {
        let scale = 1
        let viewport = page.getViewport({ scale })
    })
})
```

渲染到 canvas：

```js
pdf.getPage(1).then(page => {
    let scale = 1
    let viewport = page.getViewport({ scale })
    let canvas = this.$refs.canvas
    let context = canvas.getContext('2d')
    canvas.width = viewport.width
    canvas.height = viewport.height
    let renderContext = {
        canvasContext: context,
        viewport: viewport
    }
    page.render(renderContext)
})
```

#### Step3 渲染图片

```html
<img :src="pdfUrl" />
```

```js
page.render(renderContext)
this.pdfUrl = canvas.toDataURL('image/png')
```

#### Step4 禁止右键和复制

```html
<img :src="pdfUrl" :draggable="false" oncontextmenu="return false;" />
```
#### Step5 将 pdf 的每一页转换为图片

上述步骤已经完成大体逻辑，但在 step2 中只是将 pdf 的第一页解析成了图片。实际需求需要解析每一页，然后通过轮播的方式显示图片。因此需要做以下改造：

```html
<Carousel v-if="pdfImgsShow">
    <CarouselItem v-for="(item, index) in pdfImgs" :key="index">
        <img :src="item" :draggable="false" oncontextmenu="return false;" />
    </CarouselItem>
</Carousel>
```
```js
window.pdfjsLib.getDocument(window.URL.createObjectURL(res.data)).promise.then(async pdf => {
    for(let i = 1; i <= pdf.numPages; i++) {
        let page = await pdf.getPage(i)
        let scale = 1
        let viewport = page.getViewport({ scale })
        let canvas = this.$refs.canvas
        let context = canvas.getContext('2d')
        canvas.width = viewport.width
        canvas.height = viewport.height
        let renderContext = {
            canvasContext: context,
            viewport: viewport
        }
        page.render(renderContext)
        this.pdfImgs.push(canvas.toDataURL('image/png'))
    }
    this.pdfImgsShow = true
})
```
如下图所示，已成功生成图片：

![](https://upload-images.jianshu.io/upload_images/1495096-4713f2b75d75ffbb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


但当 pdf 页数大于 1 时，控制台会报如下错误：

![](https://upload-images.jianshu.io/upload_images/1495096-e08f0634203c9404.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

查询源码得知，page.render 方法是异步函数，在循环体内部调用 render 方法会导致同时存在多个未执行完的 render，引发上述错误。

解决方法：
```js
await page.render(renderContext).promise
this.pdfImgs.push(canvas.toDataURL('image/png'))
```

#### Step6 调整清晰度

实际测试发现，canvas 导出的图片清晰度较差。
查询资料得知是因为 dpi 的问题，参考[该文章](https://blog.csdn.net/qq_45351419/article/details/113882420)，调整 canvas 画布的大小：
```js
let UNITS = 2
canvas.width = Math.floor(viewport.width * UNITS)
canvas.height = Math.floor(viewport.height * UNITS)
let renderContext = {
    transform: [UNITS, 0,0, UNITS, 0, 0],
    canvasContext: context,
    viewport: viewport
}
```

#### 完整代码
```html
<Carousel v-if="pdfImgsShow">
    <CarouselItem v-for="(item, index) in pdfImgs" :key="index">
        <img :src="item" :draggable="false" oncontextmenu="return false;" />
    </CarouselItem>
</Carousel>
<canvas ref="canvas" style="display: none;" />
```

```js
axios.request({
     url: imgUrl,
     type: 'get',
     responseType: 'blob'
}).then(res => {
    window.pdfjsLib.getDocument(window.URL.createObjectURL(res.data)).promise.then(async pdf => {
        let UNITS = 2
        for(let i = 1; i <= pdf.numPages; i++) {
            let page = await pdf.getPage(i)
            let scale = 1
            let viewport = page.getViewport({ scale })
            let canvas = this.$refs.canvas
            let context = canvas.getContext('2d')
            canvas.width = Math.floor(viewport.width * UNITS)
            canvas.height = Math.floor(viewport.height * UNITS)
            let renderContext = {
                transform: [UNITS, 0,0, UNITS, 0, 0],
                canvasContext: context,
                viewport: viewport
            }
            await page.render(renderContext).promise
            this.pdfImgs.push(canvas.toDataURL('image/png'))
            context.clearRect(0, 0, viewport.width, viewport.height)
            this.pdfImgsShow = true
       }
    })
})
```