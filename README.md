# webpack-flow

Build the Webpack project by step

## 目录

* 初始化项目

* 基本使用
    * 模块化
    * 使用 webpack cli 构建文件
    * 使用配置文件
    * Npm Scripts
    * Loader css
    * 分离样式表
    * tree shaking

* 优化开发环境
    * 增加 source-map
    * Dev-server
    * 模块热插拔 HMR
    * 使用 html-webpack-plugin 加速

* 代码分割
    * 多页面
    * 打包重复代码
    * 动态加载

* 引入 React 
   
* 上线
    * 分拆 config 文件
    * 代码压缩
    * 去除日志
    * gzip

## 初始化项目

mkdir webpack-demo && cd webpack-demo && npm init -y
npm i webapck --save-dev

mkdir src

vi src/index.js

```js
    var element = document.createElement('div');
    element.innerHTML = 'hello Webpack';
    document.body.appendChild(element);
```

vi index.html

```html
    <html>
      <head>
        <title>hello webpack</title>
      </head>
      <body>
        <script src="./src/index.js"></script>
      </body>
    </html>
```

## 分拆代码

src/hello.js

```js
    var element = document.createElement('div');
    element.innerHTML = 'Hello webpack';
    
    export default element
```

src/index.js

```js
    import Hello from './hello';
    document.body.appendChild(Hello);
```

## 使用 webpack cli 构建文件

`./node_modules/.bin/webpack src/index.js bundle.js`


## 使用配置文件

vi build/webpack.config.js

```js
    const path = require('path');
    
    module.exports = {
      entry: './src/index.js',
      output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, './dist')
      }
    };
```

`./node_modules/.bin/webpack --config ./webpack.config.js`


## NPM Scripts

```json
    {
      ...
      "scripts": {
        "build": "webpack"
      },
      ...
    }
```

## loader css

vi src/index.css

```css
    .red {
        color: red
    }
```

修改 hello.js

```js
    
    import './index.css';
    
    var element = document.createElement('div');
    element.innerHTML = 'Hello webpack';
    element.className = "red";
    export default element
```

修改 webpack.config.js

```js

    const path = require('path');

    module.exports = {
      entry: './src/index.js',
    
      output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, './dist')
      },
      
      module: {
        rules: [
            {
              test: /\.css$/,
              use: [
                'style-loader',
                'css-loader'
              ]
            }
          ]
      }
    };
    
```

注意看最后生成的bundle文件，加载CSS的所有脚手架都在里边。

## 分离样式表

使用 `extract-text-webpack-plugin` 插件

修改 webpack.config.js

```js
    const path = require('path');
    const ExtractTextPlugin = require("extract-text-webpack-plugin");
    
    module.exports = {
    
      entry: './src/index.js',
    
      output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, './dist')
      },
      
      module: {
        rules: [
            {
              test: /\.css$/,
              use:  ExtractTextPlugin.extract({
                fallback: "style-loader",
                use: "css-loader"
              })
            }
          ]
      },
    
      plugins: [
        new ExtractTextPlugin("styles.css"),
      ]
    };
```

## 搭建开发环境

### 增加 source-map  
修改 webpack.config.js

```js
    ...
    entry : {},
    devtool: 'source-map',
    ...
```

### 监听文件变化
修改 package.json

```json

    "scripts": {
        "build": "webpack",
        "watch": "webpack --watch"
     }
```

###  dev-server

webpack.config.js

```js
    devServer: {
        contentBase: './'
    },
```

package.json

```json
    "start": "webpack-dev-server --open",
```

### 模块热插拔 HMR

config.js

```js
    devServer: {
        contentBase: './',
        hot: true
    },
    
    plugins: [
        new webpack.HotModuleReplacementPlugin()
    ]
```

index.html

```js
    <script src="bundle.js"></script>
```

### 使用 html-webpack-plugin 优化

config.js

```js
    new HtmlWebpackPlugin({
      title: 'Hello webpack'
    }),
```

使用文件

```js
    new HtmlWebpackPlugin({
      title: 'Hot Module Replacement',
      filename: __dirname + '/index.html'
    }),
```

## 引入框架

### 多页面

新增文件

```shell
    vi about.html
    vi src/about.js
```

更新 config

```js

    entry: {
        index: './src/index.js',
        about: './src/about.js',
    },
    
    output: {
        filename: '[name].dist.js',
        path: __dirname + '/dist',
    },
    
    plugins: [
        new HtmlWebpackPlugin({
          title: 'Hot Module Replacement',
          filename: __dirname + '/index.html',
        }),
        new HtmlWebpackPlugin({
          title: 'Hot Module Replacement',
          filename: __dirname + '/about.html',
        }),
        new webpack.HotModuleReplacementPlugin()
    ]
```

### 打包重复代码

引入 React 

hello.js about.js

```js
    import React from 'react';
```

修改 config

```js
    new webpack.optimize.CommonsChunkPlugin({
      name: 'common'
    })
```

引入 common

```js
    <script src="common.dist.js"></script>
```

### 动态加载

修改 index.js

```js
    let button = document.createElement('button');
    button.innerHTML = '点我加载';
    
    button.onclick = async function() {
        const hello = await import('./hello.js').then(h => h);
        document.body.appendChild(hello.default);
    }
    
    document.body.appendChild(button);
```

## 引入 React

### 解析 JSX

修改 hello.js

```js
    import React, { Component } from 'react';
    export default class Hello extends Component {
        render (){
            return (
                <div>
                    <h1>Hello</h1>
                </div>
            )
        }
    }
```

修改配置

```js
    {
        test: /\.(js|jsx)$/,
        use: ['babel-loader'],
        exclude: /node_modules/
    },
```

vi .babelrc

```js
    {
        "presets": ["es2017", "react", "stage-0"]
    }
```

### 配置热加载


## 代码优化

### 非 CommonJS/AMD 文件
### 别名
### 预设置全局变量
### 引入dist文件
    
## 部署



