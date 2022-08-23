## 1. node.js介绍
>- node.js是Server side javascript runtime，即服务端的js运行时。js运行在浏览器内核中，node.js可理解为js的运行环境，可以在node中运行JS代码。
>- JS由ECMAScript、DOM、BOM三部分组成，node中只能运行ECMAScript,无法使用BOM和DOM。
>- node将浏览器JS引擎（chrome的V8引擎）搬到了服务器端，增加了一些提供文件、网络之类操作的API。
>- node.js内置http服务器（PHP需要Apache才能运行）,虽然文件较小（大概十几M ）并发量超乎想象。

NodeJS的优势：
>- 1、Nodejs语法完全是js语法，只要你懂js基础就可以学会Nodejs后端开发;
>- 2、NodeJs超强的高并发能力;
>- 3、实现高性能服务器;
>- 4、开发周期短、开发成本低、学习成本低;

因此，基于上述原因，NodeJS的应用面特别广，基于对比微软Clipchamp与剪映Capcut均使用nodejs的情况，本文介绍如何基于NodeJS环境搭建支持Cors的web服务器(Cors是wasm多线程基础)；

其中，本文也有参考最近比较流行的Express框架

Express 是一个简洁而灵活的node.js Web应用框架, 提供了一系列强大特性帮助你创建各种Web 应用，和丰富的HTTP 工具。 使用Express 可以快速地搭建一个完整功能的网站。

## 2. 环境搭建
   以下以Mac平台为例：
### 2.1 安装node

  在官网(https://nodejs.org/en/download/) 下载Mac版的可执行程序就行了，然后按顺序将其安装即可！  
![](files://files)

### 2.2 验证node是否安装成功
  版本号存在即表示安装成功了：
  
### 2.3 安装cnpm
  cnpm主要用于项目管理，必须安装
  
```
  sudo npm install -g cnpm --registry=https://registry.npm.taobao.org
```

### 2.4 安装express
   安装express框架，用于构建项目；
   
```
  sudo npm install -g express
```
## 3. 使用openssl生成密钥
    在任一路径(建议在本项目的ssl目录下，也可以使用附带的密钥)生成所需的密钥，其中证书信息可以随便填或者留空，只有Common Name要根据你的域名填写。

Common Name (e.g. server FQDN or YOUR name) []: wes.test.com

```
cd <path-to-nginx>
mkdir ssl
 
# 使用openssl生成密钥privkey.pem
openssl genrsa -out privkey.pem 2048
 
# 使用密钥生成证书server.pem
openssl req -new -x509 -key privkey.pem -out server.pem -days 3650
```

## 4.使用方式

```
# 启动项目
cnpm start
```

## 5. 项目目录

+ bin/
 - wes ----------------------------项目入口文件
+dist/
 - js/ -----------------------------js文件
 - css/ ---------------------------css文件
+node_modules/ ---------项目依赖文件夹，cnpm intall后生成
+routes/ -------------------路由配置文件夹(可添加其他页面的路由配置)
 - index.js -----------------------首页的路由配置
+ssl/ ----------------------HTTPs证书文件
 - priv.key
 - server.pem
+views/ -------------------模板文件(可添加其他页面)
-index.ejs ----------------------首页的模板
+app.js --------------------存放的Express项目中最基本的配置信息
+package.json ------------项目依赖文件

## 6. 文件解析
 ### 6.1 wes
   文件为web服务的启动配置，核心是使用HTTPS证书配置https服务器.
 
```
// web服务的管理入口
 
var express = require('express');
var path    = require('path');
var index   = require('./routes/index');                // 引入index.js路由配置文件
 
var app = express();                                    // 用express创建一个app应用
// view engine setup
app.set('views',       path.join(__dirname, 'views'));  // 指定视图文件夹 views/
app.set('view engine', 'ejs');                          // 指定视图引擎 ejs
// 全局跨域设置
app.all('*', (req, res, next) => {
    res.header('Access-Control-Allow-Origin',  '*');
    res.header('Cross-Origin-Embedder-Policy', 'require-corp');
    res.header('Cross-Origin-Opener-Policy',   'same-origin');
    next();
})
 
// 设置静态资源路径
app.use(express.static(path.join(__dirname, 'dist')));  // 指定公共资源文件夹 为public/
// 路由规则
app.use('/', index);                                    // 当路径为'/'，即'https://localhost:443/'时，匹配路由配置index.js
 
// 匹配404，即路径未匹配时
app.use(function (req, res, next) {
    var err     = new Error('Not Found');
    err.status  = 404;
    next(err);
});
 
module.exports  = app;
```
### 6.2 app.js

```
// web服务的管理入口
 
var express = require('express');
var path    = require('path');
var index   = require('./routes/index');                // 引入index.js路由配置文件
 
var app = express();                                    // 用express创建一个app应用
// view engine setup
app.set('views',       path.join(__dirname, 'views'));  // 指定视图文件夹 views/
app.set('view engine', 'ejs');                          // 指定视图引擎 ejs
// 全局跨域设置
app.all('*', (req, res, next) => {
    res.header('Access-Control-Allow-Origin',  '*');
    res.header('Cross-Origin-Embedder-Policy', 'require-corp');
    res.header('Cross-Origin-Opener-Policy',   'same-origin');
    next();
})
 
// 设置静态资源路径
app.use(express.static(path.join(__dirname, 'dist')));  // 指定公共资源文件夹 为public/
// 路由规则
app.use('/', index);                                    // 当路径为'/'，即'https://localhost:443/'时，匹配路由配置index.js
 
// 匹配404，即路径未匹配时
app.use(function (req, res, next) {
    var err     = new Error('Not Found');
    err.status  = 404;
    next(err);
});
 
module.exports  = app;
```

## 7.关于NODEFS

最初准备搭建Node.js的Web服务器是对Emscripten的“NODEFS支持本地文件系统”(仅能在Node环境下)的认知有误解，因此想构建一个能正在本地持久化存储、且空间不受限的FS(区别于IDBFS)



   而实际上：

NODEFS并不是给web服务器使用的，即使是使用node.js搭建的服务器，它的环境依旧不是NODE！！
NODE环境是指使用node命令启动的程序，如node fs.js;
mount可以指定目录，"."表示的是当前路径;
NODEFS如果能用于web端进行本地文件操作，明显和web的安全策略相悖, 因此不能是作用于web服务器！！


## 8. 问题解决
### 8.1 未安装express


       没有安装express，Node.js是运行环境，express是项目的框架，需要通过npm安装express包才能使用，否则在request("express")时找不到文件；

### 8.2 deprecated警告

      当前未发现该警告不能设置response的Header域(主要用于跨域），暂时忽略！


### 8.3 mime-type错误

   content-type有很多种，如text/html;application/json等，这里主要是将js文件的content-type设置错误，导致其不能执行(当前项目采用的express框架没有此问题)



### 8.4 服务器拒绝，跨域拒绝

  主要是对express实例的路由设置，必须放在最后，顺序放错就会出现下述报错，


```
app.all('*', (req, res, next) => {
      res.header('Access-Control-Allow-Origin', '*');
      res.header('Cross-Origin-Embedder-Policy', 'require-corp');
      res.header('Cross-Origin-Opener-Policy', 'same-origin');
      next();
})
 
// 设置静态资源路径
app.use(express.static(path.join(__dirname, 'dist'))); // 指定公共资源文件夹 为public/
// 路由规则
app.use('/', index); 
```
![](https://github.com/AngenimyWhu/Docs/blob/main/docs/files/imgs/image2022-8-22_14-11-53.png)

### 9. Demo
   see 