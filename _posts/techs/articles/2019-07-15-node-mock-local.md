---
title: 使用node搭建本地mock服务
date: 2019-07-15 15:40:35
permalink: /docs/techs/node-mock-local
key: node-mock-local
sharing: true
show_author_profile: true
show_subscribe: true
sidebar:
  nav: tech-nav
categories:
  - 前端
  - 技术文章
tags:
  - mock
---



在实际开发中，往往是前后端同时开发，后端通过写好接口文档，前端进行对接。但是并没有数据，前端只能自己造假数据进行渲染。
<!--more-->
而造假数据的一般都是使用`mock.js`或者是在线的mock服务，比如：[easy-mock](https://www.easy-mock.com/)、[fastmock](https://www.fastmock.site/)，但是这些在线mock往往服务不稳定经常挂掉。当然我们也可以使用easy-mock部署到自己的服务器。但是也是坑比较多。我们可以在本地使用`mock.js`来拦截请求，通过`mock`假数据返回，但是这样的拦截在network里面是获取不到的，因为压根没有发起请求。为了模拟真实的请求，这里使用`node.js`搭配`express`框架搭建本地mock服务。具体`mock.js`的使用请参考[官网教程](http://mockjs.com/)。

# 使用node搭建本地mock服务

使用`mock.js`和`node.js`搭建模拟后端服务设计。使用node搭建本地服务器，实现真实请求。mock造假数据响应请求。
功能点：

      - 一键开启mock服务
      - 拆分不同模块api
      - 修改文件后自动重启服务 （nodemon）

## node服务搭建

1. 安装`express`

`npm i express --save-dev`

2. 安装`mockjs`

`npm i mockjs --save-dev`

3. 新建目录文件夹

在src目录下新建以下结构的文件夹
```javascript
├─mocks
|   ├─modules          //路由模块
|   |      └login.js
|   |      └user.js
|   ├─index.js        //node服务主文件
```
4.`index.js`文件
需要配置跨域才能够通过请求
```javascript
let express = require('express')//引入express框架

let app = express()
app.use(function (req, res, next) { //配置跨域
 res.header("Access-Control-Allow-Origin", "*");
 res.header('Access-Control-Allow-Methods', 'PUT, GET, POST, DELETE, OPTIONS');
 res.header("Access-Control-Allow-Headers", "X-Requested-With");
 res.header('Access-Control-Allow-Headers', 'Content-Type');
 next();
});

let login = require('./modules/login') //引入不同模块的api
let user = require('./modules/user')

app.use('/api',login)
app.use('/api',user)



app.listen('8090', () => {
 console.log('监听端口 8090')
})
```
5.不同业务场景api举例
使用路由拆分不同的模块api
`login.js`
```javascript
let app = require('express').Router();  
let Mock = require('mockjs')

//获取验证码
app.post('/login/codeImage/', (req, res) => {
 res.json(Mock.mock({
  "code": 0,
  "data": {
  }, 
  "message": "成功", 
  "timestamp": "1594018370", 
  "type": "success"
 }))
})

module.exports = app
```
`user.js`
```javascript
let app = require('express').Router()
let Mock = require('mockjs')

app.post('/userInfo', (req, res) => {
 res.json(Mock.mock({
  "code": 0,
  "data": {
  },
  "message": "成功",
  "timestamp": "1594018370",
  "type": "success"
 }))
})

module.exports = app
```
## 服务启动
在命令行进入到`mocks`目录下，运行`node index`命令即可启动服务。
## 端口监听
在`vue.config.js`文件代理中,将代理服务改为我们的node服务端口即可。
```javascript
proxy: { //配置跨域
  "/api": {
    target: "http://localhost:8090",
		//...
  },
```
现在运行`server`服务能够发现真实的服务请求了，并且返回的数据是我们`mock`的。


## 扩展
以上已经能够模拟真实请求，返回`mock`数据了。但是会发现一个问题，就是我们只要修改了mock文件夹里面的文件，必须重新运行命令，重启node服务。未免太麻烦了。这里我们安装nodemon插件。
`npm i nodemon --save-dev`
引入后，我们使用`nodemon mocks/index`命令来启动node服务，可以发现在我们修改来mocks文件夹里面的文件，node服务会自动重启。


每次输入命令也是很烦的，我们可以在`package.json`文件里面像配置serve服务一样的命令，配置指令。
`"mock": "nodemon ./src/mocks/index"`，添加一行这条命令。在vscode 或webstorm里面就有一键启动的指令。


完...




