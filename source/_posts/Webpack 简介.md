---
title: Webpack 介绍
categories: 构建工具
tags:
- javascript
- Webpack
---



## 一、Webpack 是什么 

`Webpack` 是一种前端资源构建工具，一个静态模块打包器。在 `Webpack`看来，前端的所有资源文件（js/json/css/img/less/...）都会作为模块处理。它将根据模块的依赖关系进行静态分析，打包生成对应的静态资源（bundle）。

## 二、Webpack  五个核心概念

- **Entry**

  入口指示， Webpack 以哪个文件入口七点开始打包，分析构建内部依赖图

- **Output**

​       输出指示 Webpack 打包后的资源 bundles 输出到哪里去，以及如何命名

- **Loader**

​       Loader 让 Webpack 能够去处理那些非 JavaScript 文件

-  **Plugins**

  插件可以用于执行范围更广的任务。插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量等

- **Mode**

​       使用相应的模式的配置：`development` （开发模式）和 `production` （生产模式）



>`Webpack` 能处理 js/json 资源，不能处理css/img 等其他资源
>
>生产环境和开发环境将 ES6 模块化编译成浏览器能狗识别的模块化
>
>生产环境比开发环境多一个压缩 JS 代码

##### 安装及使用

```javascript
//安装
npm init
npm i webpack webpack-cli -D
npm i style-loader css-loader -D
//使用
webpack
```

配置 `webpack.config `文件：

```javascript
const path = require('path');

module.exports = {
  entry: './src/js/index.js', //入口起点文件开始打包
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js'
  } 
};
```



#### Webpack 打包样式资源

- 下载所需要的 `npm` 包

  ```javascript
  npm i css-loader style-loader less-loader less -D
  ```

- 在 `module`  配置项里进行配置，`test` 为匹配的正则文件，`use `为使用的插件包

```javascript
const path = require('path'); 
module.exports = {
  entry: './src/js/index.js', //入口起点文件开始打包
  output: {
    path: path.resolve(__dirname, 'dist'), //绝对路径 _dirname: 当前文件的目录绝对路径
    filename: 'bundle.js'
  },//打包后输出文件
  module: {
    //详细loader 配置
    rules: [{
      //匹配哪些文件，不同文件匹配不同loader处理
      test: /\.css$/,
      //使用哪些 loader 进行处理
      use: [
        'style-loader', //创建style标签，将js中的样式资源插入进去，添加到head中生效，执行顺序从下到上,从右到左依次执行
        'css-loader' //将css文件变成 common.js 加载到js中
      ]
    },{
      test: /\.less$/,
      //使用哪些 loader 进行处理
      use: [
        'style-loader', //创建style标签，将js中的样式资源插入进去，添加到head中生效，执行顺序从下到上,从右到左依次执行
        'css-loader', //将css文件变成 common.js 加载到js中
        'less-loader' //将less文件编译成css文件 需要下载less-loader和less
      ]
    }]
  },
  plugins: [],
  mode: 'development'
};
```



#### Webpack 打包 html 资源

- 下载 `html-webpack-plugin` 插件
- 引入 `html-webpack-plugin`

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin')
```

- 在 `plugins` 中配置使用

```javascript
 plugins: [
    new HtmlWebpackPlugin({
      template: './index.html' 
    })
  ],
```

> 引入 `index.html` 模板后 不需要在index.html 中手动引入外部 js 和 css，插件会自动引入打包输出的所有资源



#### Webpack 打包图片资源

- 下载 `url-loader` `file-loader` （处理 css url ）
- 下载 `html-loader` （处理 html 中 img 标签里的 url）
- 在 `loader` 里进行配置
- 图片进行 base 64处理
- 当只需要单独引用 `loader` 时候，可以直接用。多个需要用 `use` 进行处理
- 打包分类时可根据 `outputPath` 生成不同的文件夹整理

```javascript
{
    test: /\.(jpg|png|jpeg|ico)$/,
    loader: 'url-loader',//使用1个loader 直接写loader，需要下载url-loader, file-loader
    options: {
      limit: 8 * 1024 , //图片大小小于8KB，就会被base64处理，优点： 减少请求数量，缺点：图片体积会更大
      name: '[hash:10].[ext]' ,//給图片进行重命名， 取hash的前10位
      outputPath: 'img'
    }
  },{
    test: /\.html$/,
    loader: 'html-loader',
  }
```



#### Webpack 打包其他资源

```javascript
{
  //打包其他资源 (除了html/js/css资源以外的资源)
  exclude: /\.(css|js|html)$/,
  loader: 'file-loader'
}
```



#### Webpack 自动化打包

- 下载 `webpack-server`

- 配置

  ```javascript
    devServer: {
      contentBase: path.resolve(__dirname, 'dist'),
      //启动gzip压缩
      compress: true,
      //端口号
      port: 3000,
      //自动打开浏览器
      open: true
    }
  ```

> 开发服务器 deServer: 用来自动化 （自动化编译，自动打开浏览器，自动刷新浏览器）
>
> 特点： 智慧在内存中编译打包，不会有任何输出
>
> 启动： npx webpack-server



## 三、提取 css 成单独文件及压缩

##### 提取：

- 下载 `mini-css-extract-plugin` 插件

- 引入插件 

  ```javascript
  const MiniCssExtractPlugin = require('mini-css-extract-plugin')
  ```

- 使用插件

  ```javascript
  {
      test: /\.less$/,
      use: [
        // 'style-loader', //创建style标签，将js中的样式资源插入进去，添加到head中生效，执行顺序从下到上,从右到左依次执行
        MiniCssExtractPlugin.loader,  //这个loader 取代style-loader.作用：提取css 成单独文件
        'css-loader', //将css文件变成 common.js 加载到js中
        'less-loader' //将less文件编译成css文件 需要下载less-loader和less
      ]
  }, 
  
  plugins: [
      //html-webpack-plugin 插件  功能：默认创建一个空的html, 自动引入打包输出的所有资源（JS/CSS）
      new HtmlWebpackPlugin({
        template: './index.html'  //复制‘./index.html文件’ 并自动引入打包输出的所有资源（JS/CSS）
      }),
      //对输出的css文件重新命名
      new MiniCssExtractPlugin({
        filename: 'css/built.css'
      })
  ],
  ```

##### 压缩：

- 下载 `optmize-css-assets-webpack-plugin` 插件

- 引入插件

  ```javascript
  const OptmizeCssAssetsWebpackPlugin = require('optmize-css-assets-webpack-plugin')
  ```

- 使用插件

  ```javascript
  plugins: [
      //html-webpack-plugin 插件  功能：默认创建一个空的html, 自动引入打包输出的所有资源（JS/CSS）
      new HtmlWebpackPlugin({
        template: './index.html'  //复制‘./index.html文件’ 并自动引入打包输出的所有资源（JS/CSS）
      }),
      //对输出的 css 文件重新命名
      new MiniCssExtractPlugin({
        filename: 'css/built.css'
      })
      //压缩 css
      new OptmizeCssAssetsWebpackPlugin()
  ],
  ```



## 四、压缩 html 和 js

```javascript
mode: 'production'  生产环境下自动压缩 js
```

```javascript
plugins: [
    //html-webpack-plugin 插件  功能：默认创建一个空的html, 自动引入打包输出的所有资源（JS/CSS）
    new HtmlWebpackPlugin({
      template: './index.html'  //复制‘./index.html文件’ 并自动引入打包输出的所有资源（JS/CSS）
      minify: {
        // 移除空格
        collapseWhitespace: true, 
        // 移除注释
        removeComments: true,
      }
    }),
],
```



## 五、Webpack 性能优化

- 开发环境性能优化
- 生产环境性能优化

##### 开发环境性能优化

优化打包构建速度

**HMR**

优化代码调试

**source-map**

> 一种 提供源代码到构建后代码映射 技术（如果构建后代码出错了，通过映射可以追踪源代码错误）

##### 生产环境性能优化

优化打包构建速度

**oneOf、babel 缓存**

优化代码运行的性能  

**缓存（hash-chunkhash-contenthash**) 、 **tree shaking** 、**code split**

