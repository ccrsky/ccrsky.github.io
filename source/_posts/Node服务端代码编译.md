---
title: Node服务端代码编译
date: 2019-03-06 19:08:37
tags:
  - python
  - 运维

category:
  - python
---

使用 webpack 打包 node 服务端项目

### 背景

通常我们开发 node web 项目时也期望类似 java web(线上只需要一个混淆后的 jar 包)需要线上分发一个最小的部署包，通过 webpack 可以轻松实现这一点。

源码目录结构

```
build
src
├── app.js
├── config
│   ├── app.js
│   ├── filter.js
│   ├── log.js
│   └── static.js
├── controllers
│   └── index.js
├── models
│   └── index.js
├── routers
│   └── index.js
├── test
│   └── test.js
├── utils
│   ├── debug.js
│   ├── env.js
│   ├── index.js
│   ├── log.js
└── views
    ├── 403.html
    ├── 404.html
    ├── 500.html
    ├── index.html
    └── layout
```

### 安装依赖

```shell
npm install -D @babel/core webpack webpack-cli webpack-node-externals babel-cli babel-loader babel-preset-env clean-webpack-plugin copy-webpack-plugin
```

### webpack 配置

```js
const path = require("path");
const CleanWebpackPlugin = require("clean-webpack-plugin");
const nodeExternals = require("webpack-node-externals");
const CopyWebpackPlugin = require("copy-webpack-plugin");

module.exports = {
  target: "node",
  entry: {
    app: "./src/app.js"
  },
  externals: [nodeExternals()],
  // mode: process.env.NODE_ENV,
  mode: "development",
  output: {
    filename: "[name].js",
    path: path.resolve(__dirname, "./build"),
    libraryTarget: "commonjs"
  },
  node: {
    __dirname: true // 需要处理代码里的变量，不然会被转掉
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: __dirname + "node_modules",
        include: __dirname + "src",
        use: {
          loader: "babel-loader",
          options: {
            presets: ["env"]
          }
        }
      }
    ]
  },
  plugins: [
    new CleanWebpackPlugin(),
    new CopyWebpackPlugin([
      {
        from: path.join(__dirname, "src/views"),
        to: path.join(__dirname, "build/views")
      }
    ])
  ]
};
```

### 编译代码

```json
{
  "name":"node-web-demo",
  "version:"0.1.0",
  "scripts": {
    "compile": "webpack"
  }
}
```

### 运行服务

```shell
# 打包生成的文件都在build目录下，执行如下命令开启服务
node build/app.js
```
