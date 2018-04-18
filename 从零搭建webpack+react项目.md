# 从零搭建webpack+react项目
本文将记录如何从零开始搭建一个webpack+react的项目，在开始之前请确保你安装了node.js。友情链接：[node.js 安装包地址](https://nodejs.org/en/)
## npm init
初始化项目，创建一个package.json。
## npm install webpack --save
安装webpack
## npm install react --save
安装react
## 在根目录下创建一个命名为build的文件夹
在build文件夹下，我们会放一些我们的配置文件、一些webpack的config文件、以及其他的我们在工程里面需要用到的脚本文件。
## 在根目录下创建一个命名为client的文件夹
这个文件夹下放前端应用的文件
## 在client目录下新建一个命名为app.js的js文件
app.js作为应用的入口
## 在client目录下新建一个命名为App.jsx的jsx文件
声明整个应用的页面上的内容
## 在bulid目录下新建一个命名为webpack.config.js的js文件
先配置好入口和出口文件

```
const path = require('path')

module.exports = {
    entry: {
        app: path.join(__dirname, '../client/app.js')
    },
    output: {
        filename: '[name].[hash].js',
        path: path.join(__dirname, '../dist'),
        publicPath: '/public'
    }
}
```
## 在package.json的"scripts"代码里，添加一行命令：

```
"build": "webpack --config build/webpack.config.js"
```
使webpack按照webpack.config.js进行打包
## 在app.js里随便添加一行代码

```
alert('123');
```
## 运行npm run build命令
你就会发现生成了dist文件夹，dist文件下有一个app.[hash值].js文件，这个文件的最后部分可以看到 alert('123');
## npm install react-dom --save
安装react-dom
## 写一个简单的App组件
在App.jsx添加如下代码：

```
import React from 'react';

export default class App extends React.Component {
    render() {
        return (
            <h1>Hello, echo</h1>
        );
    }
}
```
## 渲染App组件
在app.js添加如下代码：

```
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App.jsx';

ReactDOM.render(
    <App/>,
    document.body
);
```
## 安装babel-loader并配置
实际上，上述jsx代码需要为它配置loader转译。
- npm install babel-loader --save-dev
- npm install babel-core --save-dev
- 在webpack.config.js的module.exports的对象里添加如下代码：

```
module: {
    rules: [
        {
            test: /\.(js|jsx)$/,
            loader: 'babel-loader'
        }
    ]
}
```
## 配置jsx环境
在上述配置好babel-loader之后运行npm run build发现会报错，其中的jsx代码并不能被编译，因为babel默认运行es6代码。

解决办法是在根目录下创建一个名为 *.babelrc* 的文件，加上如下代码：

```
{
    "presets": [
        ["es2015", {"loose": true}],
        "react"
    ]
}
```
还需要为其配置里安装所需要的插件：

*npm install babel-preset-es2015 babel-preset-es2015-loose babel-preset-react --save-dev*

此时，再执行npm run build就发现可以成功编译了。
## 在浏览器中显示
- 首先，需要安装html-webpack-plugin，npm install html-webpack-plugin --save-dev
- 其次在webpack.config.js中引入此插件：

```
const HTMLPlugin = require('html-webpack-plugin');
```
- 在module对象下面添加如下代码：

```
plugins: [
    new HTMLPlugin()
]
```
此时，再运行npm run build，会发现，dist目录除了生成js文件外还生成了一个index.html文件。将这个html用浏览器打开，发现报错了，说app.xxx.js文件没有找到，是因为没有做路径映射。

解决方法是，修改webpack.config.js的publicPath：

```
publicPath: ''
```
为保证js文件也要被正确编译，将webpack.config.js的rules的代码修改为：

```
rules: [
    {
        test: /.jsx$/,
        loader: 'babel-loader'
    },
    {
        test: /.js$/,
        loader: 'babel-loader',
        exclude: [
            path.join(__dirname, '../node_modules')
        ]
    }
]
```
这里的exclude表示编译时不需要编译哪些文件。

此时，删掉dist目录，再运行npm run build，将index.html用浏览器打开，会发现页面可以显示：Hello,echo 了。
## 配置服务端打包
1. 在client目录下新建一个名为server.entry.js的文件，为服务端的入口文件，加上代码为：

```
import React from 'react';
import App from './App.jsx';

export default <App/>
```
2. 在build目录下，新建一个名为webpack.config.server.js的文件，用来作为webpack打包服务端代码的配置文件，加上代码为：

```
const path = require('path');

module.exports = {
    target: 'node', //代表打包出来的内容的执行环境
    entry: {
        app: path.join(__dirname, '../client/server.entry.js')
    },
    output: {
        filename: 'server.output.js',
        path: path.join(__dirname, '../dist'),
        publicPath: '',
        libraryTarget: 'commonjs2' //使用模块的方案，AMD,CMD....
    },
    module: {
        rules: [
            {
                test: /.jsx$/,
                loader: 'babel-loader'
            },
            {
                test: /.js$/,
                loader: 'babel-loader',
                exclude: [
                    path.join(__dirname, '../node_modules')
                ]
            }
        ]
    }
}
```
3. 为区别，将webpack.config.js重命名为webpack.config.client.js
4. 安装rimraf，npm install rimraf --save-dev，rimraf是以包的形式包装rm -rf命令，用来删除文件和文件夹的，不管文件夹是否为空，都可删除。
5. 重新配置npm run build的命令，修改package.json的"scripts"的代码为：

```
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build:client": "webpack --config build/webpack.config.client.js",
    "build:server": "webpack --config build/webpack.config.server.js",
    "clear": "rimraf dist",
    "build": "npm run clear && npm run build:client && npm run build:server"
  },
```
这样，npm run build的命令执行的内容是：先删除dist文件夹，然后打包客户端文件，最后打包服务端文件。

你可以试着运行npm run build，看看效果。
## 建立服务端渲染
1. 在根目录下新建一个名为server的文件夹，用来放置服务端的代码。
2. 安装express，原来作为服务端的框架，npm install express --save
3. 在server目录下新建一个名为server.js的js文件，加上代码为：

```
const express =require('express');
const ReactSSR = require('react-dom/server');
const serverOutput = require('../dist/server.output').default;
//因为server.output.js默认导出App，
//而commonjs2规范的require导入的是server.output的整个部分，
//并不是它的默认导出，所以需要在后面加上.default

const app = express();

app.get('*', function(req, res) {
    const appString = ReactSSR.renderToString(serverOutput);
    res.send(appString);
})

app.listen(3333, function() {
    console.log('server is listening on 3333')
})
```
4. 在package.json配置npm start的命令：

```
"start": "node server/server.js"
```
使其启动是的服务端。

此时，运行npm start， 用浏览器打开[http://localhost:3333/](http://localhost:3333/)，就可以看到：Hello,echo 了。
## 完善服务端渲染
在http://localhost:3333/这个页面，可以看到服务端返回内容的是<h1 data-reactroot="">Hello, echo</h1>，而实际中我们并不会返回如此简单的代码，而且这个页面并没有引用客户端的js。

因此，我们需要把服务端渲染的内容插到dist下的index.html，然后将整个html的内容返回到浏览器端，这样才算是走通服务端渲染的整个流程。具体做法是：
1. 在client文件夹下新建一个名为template.html的html模板文件，加上代码：

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" 
    content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>cnode..</title>
  </head>
  <body>
      <div id="root"><app></app></div>
      <!-- <app></app>这部分内容是在服务端渲染时将它替换掉,客户端渲染也需要替换 -->
  </body>
</html>
```
2. 这时，App组件也不需要挂载在body了，修改app.js的render代码为：

```
ReactDOM.render(
    <App/>,
    document.getElementById('root')
);
```
3. 在webpack.config.client.js里使用上述template.html。将plugins的代码改为：

```
plugins: [
    new HTMLPlugin({
        template: path.join(__dirname, '../client/template.html')
    })
]
```
这样的话dist目录下打包生成的html，会以template.html作为模板，并插入生成的js。
3. 在server端读取打包生成的html，将server.js的代码更改为：

```
const express =require('express');
const ReactSSR = require('react-dom/server');
const fs = require('fs');
const path = require('path');  //在使用到路径时，一般引入path，保证绝对路径，避免出错
const serverOutput = require('../dist/server.output').default;
//因为server.output.js默认导出App，
//而commonjs2规范的require导入的是server.output的整个部分，
//并不是它的默认导出，所以需要在后面加上.default

const app = express();

const template = fs.readFileSync(path.join(__dirname, '../dist/index.html'), 'utf8'); 
//同步读取文件
//指定以utf8的格式读进来，将之变成String

app.get('*', function(req, res) {
    const appString = ReactSSR.renderToString(serverOutput);
    res.send(template.replace('<app></app>', appString));  //替换<app></app>
})

app.listen(3333, function() {
    console.log('server is listening on 3333')
})
```
4. 在http://localhost:3333/这个页面我们发现js请求返回的也是html，然而这是错误的，这时我们需要给server.js加上一行代码：

```
app.use('/public', express.static(path.join(__dirname, '../dist')));
//给静态文件指定对应的请求返回
```
然后，将webpack.config.client.js和webpack.config.server.js的publicPath改回：

```
publicPath: '/public',
```
这样可以方便服务端区分返回的静态内容和要渲染的内容，只要是/public开头的就全部返回静态文件。

此时，重新npm run build和npm start，发现返回的js是正确的了。

除此之外，发现浏览器报了一个warning，意思是用ReactDOM的hydrate方法替代render方法，
因为React 期望在服务器和客户端之间渲染的内容是相同的，因此修改app.js的render代码为：

```
ReactDOM.hydrate(
    <App/>,
    document.getElementById('root')
);
```
## 配置自动编译和自动启动server
每次我们更改代码都需要重新去编译和启动服务，非常繁琐。可以使用webpack提供的webpack dev server和Hot module replacement，Hot module replacement可以免去改动之后需要刷新浏览器的动作。
- 安装webpack dev server，npm install webpack-dev-server --save-dev
- 安装cross-env，npm install cross-env --save-dev。cross-env是运行跨平台设置和使用环境变量的脚本。
- 在package.json配置webpack dev server命令：

```
"dev:client": "cross-env NODE_ENV=development webpack-dev-server --config build/webpack.config.client.js",
```
- 更改webpack.config.client.js的代码为：

```
const path = require('path');
const HTMLPlugin = require('html-webpack-plugin');

const isDev = process.env.NODE_ENV === 'development';//判断是否是开发环境

const config = {
    entry: {
        app: path.join(__dirname, '../client/app.js')
    },
    output: {
        filename: '[name].[hash].js',
        path: path.join(__dirname, '../dist'),
        publicPath: '/public'
    },
    module: {
        rules: [
            {
                test: /.jsx$/,
                loader: 'babel-loader'
            },
            {
                test: /.js$/,
                loader: 'babel-loader',
                exclude: [
                    path.join(__dirname, '../node_modules')
                ]
            }
        ]
    },
    plugins: [
        new HTMLPlugin({
            template: path.join(__dirname, '../client/template.html')
        })
    ]
}

if (isDev) {
    config.devServer = {
      host: '0.0.0.0',  //指定使用一个 host。默认是 localhost。如果你希望服务器外部可访问，指定如左
      port: '8888',    //指定要监听请求的端口号
      contentBase: path.join(__dirname, '../dist'),
      //contentBase告诉服务器从哪里提供内容。只有在你想要提供静态文件时才需要。
      //默认情况下，将使用当前工作目录作为提供内容的目录，但是你可以修改为其他目录：
      hot: true,  //启用 webpack 的模块热替换特性(Hot module replacement)
      overlay: {  //在编译器错误或警告时，在浏览器中显示显示其信息。
        errors: true  //只显示编译错误
      }
    }
  }

  module.exports = config
```
