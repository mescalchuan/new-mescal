---
layout: post
title: Webpack工程化配置
date: 2017-11-15 13:32:20 +0300
catalog: true
description: 
header-img: /img/mescal/8.jpg # Add image post (optional)
tags: 
    - 博客
    - 技术博客
    - webpack
categories: [技术博客]
ascription: technology
author: # Add name author (optional)
---
本文重点讲述如何一步一步搭建webpack工程化配置，这里涉及到一些常用插件的使用以及node.js的文件操作。上篇文章介绍到，webpack配置成了两种模式，有些人喜欢将两种模式的代码放到不同的配置文件中，而我更偏爱全部集成到webpack.config.js里，因为这里有一些配置是通用的。我将代码分为三块：基本配置、开发模式配置、生产模式配置。本文建议你有一定的webpack基础，因为我不会花费大量篇幅去讲解每个loader、插件的具体用法。

### 基本配置

#### 核心一：多入口文件
如果你的项目是单页应用，可能最终打包后仅仅有一个js文件、一个css文件、一个html文件。在这种情况，webpack配置起来并没有太大难度。但是一旦项目有多入口（比如多页应用），那么配置起来就比较麻烦了。
你可以手写多个入口文件，但是这样做肯定不灵活：
```
entry:{
    entry1:'xxx1.js',
    entry2:'xxx2.js',
    entry3:'xxx3.js',
    ...
}
```
这样的话每次你创建了新的入口文件都需要更改webpack.config.js。倒不如换一种思想：既然webpack是基于node.js的，直接规定好入口文件的目录结构然后遍历文件夹自动生成入口文件配置。
```
var jsPath = path.resolve(__dirname, 'entry');
var files = fs.readdirSync(jsPath);
var entry = {};
files.forEach(function(filename) {
    var stats = fs.statSync(path.join(jsPath, filename));
    if (stats.isDirectory()) {
        var entryJSKey = filename + '/' + 'main.js'.split('.js')[0];
        entry[entryJSKey] = path.join(jsPath, filename, '/' + 'main.js');
    }
})
```
```
var webpackConfig = {
    entry: entry
}
```

#### 核心二：判断当前开发环境
我们来判断一下当前的开发环境，process.env.NODE_ENV可以获取到命令行中输入的NODE_ENV的值，我们将其定义为production：
```
var DefinePlugin = webpack.DefinePlugin;
var isDevelopment = process.env.NODE_ENV !== 'production';
```

现在来配置一下输入路径。为了方便引用，我将输出路径定义为与入口文件同级，文件的命名与入口文件相同，后缀取决于当前的环境。
```
var webpackConfig = {
    entry: entry,
    output: {
        path: jsPath,
        filename: isDevelopment ? '[name].__bundle.js' : '[name].bundle.js',
    },
    plugins: [
        new webpack.DefinePlugin({
            'process.env': {
                NODE_ENV: JSON.stringify(process.env.NODE_ENV),
            }
        })
    ]
}
```

#### 核心三：提取公共文件
实际项目中有很多公共的文件，将这些文件全都打包到每一个入口文件里明显不合适，因此我们使用CommonsChunkPlugin插件将公共文件和入口文件分离打包。
```
var CommonsChunkPlugin = webpack.optimize.CommonsChunkPlugin;
var commonModule1 = path.resolve(__dirname, 'common' + '/app');
//替换掉上面的var entry = {}
var entry = {
    vendor: [commonModule1]
}
```
```
var webpackConfig = {
    entry: entry,
    externals: isDevelopment ? {} : externals,
    plugins: [
        new CommonsChunkPlugin({
            name: ['vendor'],
            filename: isDevelopment ? 'vendor.__bundle.js' : 'vendor.bundle.js',
            minChunks: Infinity
        })
    ]
}
```

#### 核心四：是否需要将框架打包进来
webpack默认情况下会将入口文件（包括它的子模块）依赖的所有模块全部打包进来，这会导致打包后的文件十分庞大，这在开发环境下可以接受，但是生产环境下就不允许了。通常的操作是：开发环境下一次性全部打包，生产环境下框架、库等比较庞大的第三方模块不进行打包，转而使用cdn。
```
var externals = {
    'angular': 'angular',
    'react': 'React',
    'react-dom': 'ReactDOM'
}

var webpackConfig = {
    ...
    externals: isDevelopment ? {} : externals
}
```

Externals对象的key是框架的名字，value是你import时的变量名。举个例子，cdn上的react名字为react.js（或react.min.js），key就为"react"。在代码中我使用`import React from 'react'`，因此value是“React”。上述代码只是举例说明，实际上很少有项目会同时需要使用react和angular。

#### 核心五：加载器
Babel-loader是最常用的，style-loader和css-loader会根据环境的不同分别配置。你可以根据自己的项目加载更多的loader，比如url-loader、file-loader、sass-loader等。
```
var webpackConfig = {
    ...
    module: {
        rules: [{
            test: /\.js$/,
            exclude: /node_modules/,
            use: {
                loader: 'babel-loader',
                options: {
                    presets: ['es2015', 'stage-2']
                }
            }
        }]
    }
    ...
}
```

### 开发环境配置
在开发环境下我们需要：将css打包到js中；为了方便调试，需要映射打包后的代码和源代码；支持热更新；确保编译的代码没有错误，若出现错误将其记录；编译成功后自动打开页面。
#### 创建服务器
得益于webpack-dev-server，我们可以像node.js一样创建服务器。我们将服务器的内容定义到pages中。在服务运行后，打包后的文件就会出现在内存里，路径参考自contentBase，也就是pages。
```
var htmlPath = path.resolve(__dirname, 'pages');
var webpackConfig = {
    ...
    devServer: {
        hot: true,
        inline: true,
        progress: true,
        contentBase: htmlPath,
        port: 3000,
        stats: {
            colors: true
        }
    }
    ...
}
```

#### 在基本配置上添加开发环境配置
我们先添加css模块的加载：
```
if (isDevelopment) {
    var cssLoader = {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
    };
    webpackConfig.module.rules.push(cssLoader);
}
```
试想一下，在没有文件映射的情况下进行开发是多么恐怖的一件事：你的代码都被打包到了最终的js文件里，由于js混合了框架、库、css、babel转码，可以说一旦出现错误是很难定位的。为此，我们需要实现打包后的文件与源文件之间的内容映射，出现问题后可以直接定位到源文件的相应位置。
```
webpackConfig.devtool = 'source-map';
```
我们还要支持热更新、无错保证、自动打开页面：
```
var OpenBrowserPlugin = require('open-browser-webpack-plugin');
var HotModuleReplacementPlugin = webpack.HotModuleReplacementPlugin;
var NoEmitOnErrorsPlugin = webpack.NoEmitOnErrorsPlugin;
//假设默认开启的页面位于./pages/entryPages
webpackConfig.plugins = webpackConfig.plugins.concat([
    new HotModuleReplacementPlugin(),
    new NoEmitOnErrorsPlugin(),
    new OpenBrowserPlugin({
        url: 'http://localhost:3000/entryPages/index.html'
    })
]);
```

### 生产环境配置
在生产环境下我们需要：压缩代码；分离css和js；我们甚至可以将打包后的文件自动引入到html中。

#### 分离CSS和JS
首先是分离css和js，这里需要用到extract-text-webpack-plugin这个插件。
```
if (isDevelopment) {
    ...
}
else {
    var cssLoader = {
        test: /\.css$/,
        use: ExtractTextPlugin.extract({
            fallback: 'style-loader',
            use: 'css-loader'
        })
    };
    webpackConfig.module.rules.push(cssLoader);
    webpackConfig.plugins = webpackConfig.plugins.concat([
        new ExtractTextPlugin('main.bundle.css', {
            allChunks: false
        })
    ])
}
```

目测文件的分离是成功了，css被提取到了js中。但是你会发现，整个输出目录里只有一个main.bundle.css，但是我不同的入口文件依赖了不同的css。一我的假设是所有import进来的css被合并到了main.bundle.css中，打开css文件后我发现，main.bundle.css里面只有最后一个入口文件导入的css，而之前的css内容全部被覆盖了。

假设现在有两个入口js：
```
//main1.js
import '../css/main1.css';

//main1.css
body: { background: red }
```
```
//main2.js
import '../css/main2.css';

//main2.css
body: { background: blue }
```
最终的main.bundle.css只有main2.css里面的内容：
```
//main.bundle.css
body: { background: blue }
```

原因是webpack在分离css的时候参考自entry的配置，从中提取css文件，依据`new ExtractTextPlugin('main.bundle.css',{...})`的第一个参数生成提取后的css文件。由于文件的名字均为main.bundle.css，文件会被依次替换，导致的结果就是提取出来的css只有最后一个entry所依赖的内容。

解决方案与output相同，我们通过node.js遍历文件目录自动生成了诸如
```
{
    "page1/main": "./entry/page1/main.js",
    "page2/main": "./entry/page2/main.js",
    "page3/main": "./entry/page3/main.js",
}
```
的入口文件配置，我们可以使用[name]来获取到entry里面的key，将css命名设置为key，这样一来webpack就会根据key值在page1、2、3...的下面生成main.bundle.css了。
```
webpackConfig.plugins = webpackConfig.plugins.concat([
    new ExtractTextPlugin('[name].bundle.css', {
        allChunks: false
    })
])
```
#### 代码压缩
接下来我们将代码进行压缩，使用到了UglifyJsPlugin这个插件，被抽离的css文件默认情况下不会被压缩，需要额外用到optimize-css-assets-webpack-plugin这个插件。
```
webpackConfig.plugins = webpackConfig.plugins.concat([
    new UglifyJsPlugin({
        minimize: true,
        output: {
            comments: false,
        },
        compress: {
            warnings: false
        }
    }),
    new ExtractTextPlugin('[name].bundle.css', {
        allChunks: false
    }),
    new OptimizeCSSPlugin()
]);
```
#### 自动引入打包后的文件
最后，谈一下如何实现html自动引入打包后的文件。在webpack中html-webpack-plugin插件可以自动生成html页面，我们可以在为其指定模板为入口页面。为了实现生成多个html页面，需要创建多个HtmlWebpackPlugin对象，这一步我将其放在了文件操作里面。每个页面的名称与pages里面的各html相同，这样一来，生成的html页就会已原页面为模板，添加打包后的文件并覆盖原页面。
```
var HtmlWebpackPlugin = require('html-webpack-plugin');
var jsPath = path.resolve(__dirname, 'entry');
var files = fs.readdirSync(jsPath);
var entry = {};
files.forEach(function(filename) {
    var stats = fs.statSync(path.join(jsPath, filename));
    if (stats.isDirectory()) {
        var entryJSKey = filename + '/' + 'main.js'.split('.js')[0];
        entry[entryJSKey] = path.join(jsPath, filename, '/' + 'main.js');
        if (!isDevelopment) {
            var template = path.resolve(__dirname, 'pages', filename, 'index.html')
            var htmlPlugin = {
                filename: template,
                template: template,
                chunks: [],
                inject: true,
                chunksSortMode: 'manual',
                xhtml: true,
                showErrors: true,
                minify: false
            };
            htmlPlugin.chunks = ['vendor', entryJSKey];
            htmlPluginArr.push(new HtmlWebpackPlugin(htmlPlugin));
            }
    }
})
```

上面的配置有两个关键点：1.指定每个页面需要自动引入的模块名称。默认情况下，新生成的页面会把所有打包后的文件全部引入，在多页面应用下是不可取的，我们期望每个页面只引入与本页面有关的打包文件。每个页面打包后的js名字都是相同的，但是路径是不同的，因此我们定义每个页面需要引入的js名称为`filename + '/' + 'main.js'.split('.js')[0]`，然后在chunks里指定需要加载的模块`htmlPlugin.chunks = ['vendor', entryJSKey]`。2.保证js的引入顺序。Vendor作为全局的通用代码，它需要在main之前被引入。Vendor和main应该都被引入在body的最后面。因此，我们将inject设置为true（body）,将 chunksSortMode设置为manual，确保chunks数组的顺序`['vendor', entryJSKey]`。

### 运行
Webpack的配置已经完成，现在其实就可以应用到实际项目中了。
#### 使用Webpack命令
我们可以通过以下三种命令执行webpack配置：
```
//开启服务器并执行开发模式配置
webpack-dev-server
//打包开发模式配置
NODE_ENV=development webpack
//打包生产模式配置
NODE_ENV=production webpack
```

下面我们来精简一下上面三个命令，使用到了cross-env这个插件`npm install cross-env`。得益于package.json，我们更改一下script字段：
```
//package.json
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "webpack-dev-server",
    "build": "NODE_ENV=production webpack",
    "dev": "NODE_ENV=development webpack"
  }
```

现在直接运行`npm start`，你会发现服务成功启动了。但是运行`npm run build`和`npm run dev`在windows下会报“NODE_ENV不是内部或外部命令”。为了解决该问题，cross-env就派上用场了：
```
//package.json
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "webpack-dev-server",
    "build": "cross-env NODE_ENV=production webpack",
    "dev": "cross-env NODE_ENV=development webpack"
  }
```

再次运行`npm run build`和`npm run dev`，打包成功。

后续有时间的话可以将上述配置封装成cli，更加方便开发人员使用。

#### 11月23日更新（新增mock服务）

Mock服务可以拦截指定的请求，返回使用者定义的数据来实现前后端分离式开发。通过写代码的方式配置mock的接口并不是很方便，因此我推荐使用mock2easy-middleware这个中间件。你可以访问[文档](https://www.npmjs.com/package/mock2easy)来查看具体的配置项。
```
//安装
npm install mock2easy-middleware --save-dev

//webpack.config.js
var mock2easy = require('mock2easy');
//mock服务器配置
var mockConfig = {
    port: 3005,
    lazyLoadTime: 3000,
    database: 'mock2easy',
    doc: 'doc',
    ignoreField: [],
    interfaceSuffix: '.json',
    preferredLanguage: 'en'
};
mock2easy(mockConfig, function (app) {
    app.listen(mockConfig.port, function () {
        console.log('mockServer has started , see : localhost:' + mockConfig.port);
    });
});
```
由于当前的服务器地址为webpack-dev-server中的3000，我们需要将所有的请求都代理到mock服务器上（3005）并允许https请求。
```
devServer: {
    ...
    proxy:{
        '/*.json':{
            target:'http://localhost:3005'
            secure:false
         }
    }
}
```