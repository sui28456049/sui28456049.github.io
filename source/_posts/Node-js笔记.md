---
title: Node.js笔记
date: 2017-11-23 19:40:23
tags: Node.js
category: Node.js
---

最近在学习webpack,发现有许多node的命令,以前粗浅的学习的一下node,发现都忘光了.作为一个php后端选手,本不应该学习,注重后端,但是js越发展生态越好,还是有必要好好学习一下的.拥抱node.js.特此文章,以备后来查看所需.

# 简介

Node.js 是一个基于Chrome JavaScript 运行时建立的一个平台。
Node.js是一个`事件驱动I/O服务端JavaScript环境`，基于Google的V8引擎.

# npm 使用

NPM是随同NodeJS一起安装的包管理工具(类似于php的composer,Python的pip,解决依赖问题的)

## 全局安装和局部安装

```bash
npm install express          # 本地安装
npm install express -g   # 全局安装
```

本地安装
1. 将安装包放在 ./node_modules 下（运行 npm 命令时所在的目录），如果没有 node_modules 目录，会在当前执行 npm 命令的目录下生成 node_modules 目录。
2. 可以通过 require() 来引入本地安装的包。

全局安装
1. 将安装包放在 /usr/local 下或者你 node 的安装目录。
2. 可以直接在命令行里使用。

## 安装模块

```bash
npm install express
```
## 卸载模块

```bash
npm uninstall express
```
## 更新模块

```bash
npm update express
```
## 创建模块

```bash
npm init # 一路回车即可
```

## 查看安装信息

```bash
npm list -g     # 查看所有全局安装的模块

npm list grunt  # 查看模块版本号
```

## *使用package.json

dependencies - 依赖包列表。如果依赖包没有安装，npm 会自动将依赖包安装在 node_module 目录下。

```bash
npm install http-server --save   # 把http-server 写进依赖
```

## 使用淘宝 NPM 镜像(加速)

```bash
npm install -g cnpm --registry=https://registry.npm.taobao.org

cnpm install express
```

# Node.js模块系统(一切皆模块)

一个 Node.js 文件就是一个模块，这个文件可能是JavaScript 代码、JSON 或者编译过的C/C++ 扩展。

Node.js 提供了`exports` 和` require` 两个对象，其中 exports 是模块公开的接口，require 用于从外部获取一个模块的接口，即所获取模块的 exports 对象。

```js

// main.js
var hello = require('./hello');
hello.world();

// hello.js

exports.world = function() 
{
  console.log('Hello World');
}

# 运行 node main.js
```

有时候我们只是想把一个对象封装到模块中

```js

//hello.js 
function Hello() 
{ 
 var name; 
    this.setName = function(thyName) { 
       name = thyName; 
  }; 
   this.sayHello = function() { 
     console.log('Hello ' + name); 
  }; 
}; 
module.exports = Hello;

// 直接获得这个对象了
//main.js 
var Hello = require('./hello'); 
hello = new Hello(); 
hello.setName('BYVoid'); 
hello.sayHello(); 

```

模块接口的唯一变化是使用 module.exports = Hello 代替了exports.world = function(){}。 在外部引用该模块时，其接口对象就是要输出的 Hello 对象本身，而不是原先的 exports。

# Node.js Buffer(缓冲区)

JavaScript 语言自身只有字符串数据类型，没有二进制数据类型。

但在处理像TCP流或文件流时，必须使用到二进制数据。因此在 Node.js中，定义了一个 Buffer 类，该类用来创建一个专门存放二进制数据的缓存区。

在 Node.js 中，Buffer 类是随 Node 内核一起发布的核心库。Buffer 库为 Node.js 带来了一种存储原始数据的方法，可以让 Node.js 处理二进制数据，每当需要在 Node.js 中处理I/O操作中移动的数据时，就有可能使用 Buffer 库。原始数据存储在 Buffer 类的实例中。一个 Buffer 类似于一个整数数组，但它对应于 V8 堆内存之外的一块原始内存。

## 创建buffer
```js
//创建长度为 10 字节的 Buffer 实例：
var buf = new Buffer(10);

//通过给定的数组创建 Buffer 实例：
var buf = new Buffer([10, 20, 30, 40, 50]);

//通过一个字符串来创建 Buffer 实例
var buf = new Buffer("www.w3cschool.cn", "utf-8");
```

## 操作buffer

```js
var buf =new Buffer(30);
len = buf.write('sui sui sui learn node.js');
console.log("写入的字节数 : "+ len);

//从缓冲区读取数据
buf.toString('utf8');

//buffer转为json对象
buf.toJSON()

.....很多API,需要什么查阅官网手册吧....    

```

# Node.js Stream(流)

Sream 是一个抽象接口，Node 中有很多对象实现了这个接口。例如，对http 服务器发起请求的request 对象就是一个 Stream，还有stdout（标准输出）。

## Node.js，Stream 有四种流类型：
1. Readable - 可读操作。
2. Writable - 可写操作。
3. Duplex - 可读可写操作.
4. Transform - 操作被写入数据，然后读出结果。

所有的 Stream 对象都是 EventEmitter 的实例。常用的事件有：
* data - 当有数据可读时触发。
* end - 没有更多的数据可读时触发。
* error - 在接收和写入过程中发生错误时触发。
* finish - 所有数据已被写入到底层系统时触发。

```js
var fs = require("fs");
var data = '';

// 创建可读流
var readerStream = fs.createReadStream('input.txt');

// 设置编码为 utf8。
readerStream.setEncoding('UTF8');

// 处理流事件 --> data, end, and error
readerStream.on('data', function(chunk) {
   data += chunk;
});

readerStream.on('end',function(){
   console.log(data);
});

readerStream.on('error', function(err){
   console.log(err.stack);
});

console.log("程序执行完毕");
```
# Node.js 文件系统

Node.js 文件系统封装在 fs 模块是中，它提供了文件的读取、写入、更名、删除、遍历目录、链接等POSIX 文件系统操作。
与其他模块不同的是，fs 模块中所有的操作都提供了异步的和 同步的两个版本，例如读取文件内容的函数有异步的 fs.readFile() 和同步的 fs.readFileSync()。

```js
var fs = require('fs'); 
    fs.readFile('content.txt', 'utf-8', function(err, data) { 
    if (err) { 
       console.error(err); 
  } else { 
     console.log(data); 
   } 
}); 
```

# Express框架

安装 Express 并将其保存到依赖列表中

```bash
$ npm install express --save
```

几个重要的模块是需要与 express 框架一起安装的：

* body-parser - node.js 中间件，用于处理 JSON, Raw, Text 和 URL 编码的数据。

* cookie-parser - 这就是一个解析Cookie的工具。通过req.cookies可以取到传过来的cookie，并把它们转成对象。

* multer - node.js 中间件，用于处理 enctype="multipart/form-data"（设置表单的MIME编码）的表单数据。

```bash
$ npm install body-parser --save
$ npm install cookie-parser --save
$ npm install multer --save
```

## 第一个 Express 框架实例

```js
//express_demo.js 文件
var express = require('express');
var app = express();

app.get('/', function (req, res) {
   res.send(suisuisui);
})

var server = app.listen(8081, function () {

  var host = server.address().address
  var port = server.address().port

  console.log("应用实例，访问地址为 http://%s:%s", host, port)

})
```

### Request 对象 
Request 对象 - request 对象表示 HTTP 请求，包含了请求查询字符串，参数，内容，HTTP 头部等属性。常见属性有：
req.app：当callback为外部文件时，用req.app访问express的实例
req.baseUrl：获取路由当前安装的URL路径
req.body / req.cookies：获得「请求主体」/ Cookies
req.fresh / req.stale：判断请求是否还「新鲜」
req.hostname / req.ip：获取主机名和IP地址
req.originalUrl：获取原始请求URL
req.params：获取路由的parameters
req.path：获取请求路径
req.protocol：获取协议类型
req.query：获取URL的查询参数串
req.route：获取当前匹配的路由
req.subdomains：获取子域名
req.accpets（）：检查请求的Accept头的请求类型
req.acceptsCharsets / req.acceptsEncodings / req.acceptsLanguages
req.get（）：获取指定的HTTP请求头
req.is（）：判断请求头Content-Type的MIME类型

### Response 对象

Response 对象 - response 对象表示 HTTP 响应，即在接收到请求时向客户端发送的 HTTP 响应数据。常见属性有：
res.app：同req.app一样
res.append（）：追加指定HTTP头
res.set（）在res.append（）后将重置之前设置的头
res.cookie（name，value [，option]）：设置Cookie
opition: domain / expires / httpOnly / maxAge / path / secure / signed
res.clearCookie（）：清除Cookie
res.download（）：传送指定路径的文件
res.get（）：返回指定的HTTP头
res.json（）：传送JSON响应
res.jsonp（）：传送JSONP响应
res.location（）：只设置响应的Location HTTP头，不设置状态码或者close response
res.redirect（）：设置响应的Location HTTP头，并且设置状态码302
res.send（）：传送HTTP响应
res.sendFile（path [，options] [，fn]）：传送指定路径的文件 -会自动根据文件extension设定Content-Type
res.set（）：设置HTTP头，传入object可以一次设置多个头
res.status（）：设置HTTP状态码
res.type（）：设置Content-Type的MIME类型

## 路由

创建 express_demo2.js 文件

```js
var express = require('express');
var app = express();

//  主页输出 "Hello World"
app.get('/', function (req, res) {
   console.log("主页 GET 请求");
   res.send('Hello GET');
})


//  POST 请求
app.post('/', function (req, res) {
   console.log("主页 POST 请求");
   res.send('Hello POST');
})

//  /del_user 页面响应
app.delete('/del_user', function (req, res) {
   console.log("/del_user 响应 DELETE 请求");
   res.send('删除页面');
})

//  /list_user 页面 GET 请求
app.get('/list_user', function (req, res) {
   console.log("/list_user GET 请求");
   res.send('用户列表页面');
})

// 对页面 abcd, abxcd, ab123cd, 等响应 GET 请求
app.get('/ab*cd', function(req, res) {   
   console.log("/ab*cd GET 请求");
   res.send('正则匹配');
})


var server = app.listen(8081, function () {

  var host = server.address().address
  var port = server.address().port

  console.log("应用实例，访问地址为 http://%s:%s", host, port)

})
```

很简单,比php的路由差很多,很轻松~~

## 静态文件

Express 提供了内置的中间件 express.static 来设置静态文件如：图片， CSS, JavaScript 等。
你可以使用 express.static 中间件来设置静态文件路径。将图片， CSS, JavaScript 文件放在 public 目录下,就放在public/images/logo.png一张图片

```js
var express = require('express');
var app = express();

app.use(express.static('public'));

app.get('/', function (req, res) {
   res.send('Hello World');
})

var server = app.listen(8081, function () {

  var host = server.address().address
  var port = server.address().port

  console.log("应用实例，访问地址为 http://%s:%s", host, port)

})
```

```
在浏览器中访问 http://127.0.0.1:8081/images/logo.png,即可得到logo
```

## GET方法

### 一个表单

```html
<html>
<body>
<form action="http://127.0.0.1:8081/process_get" method="GET">
First Name: <input type="text" name="first_name">  <br>
 
Last Name: <input type="text" name="last_name">
<input type="submit" value="Submit">
</form>
</body>
</html>
```

### 服务端

```js
var express = require('express');
var app = express();
 
app.use(express.static('public'));
 
app.get('/index.htm', function (req, res) {
   res.sendFile( __dirname + "/" + "index.htm" );
})
 
app.get('/process_get', function (req, res) {
 
   // 输出 JSON 格式
   var response = {
       "first_name":req.query.first_name,
       "last_name":req.query.last_name
   };
   console.log(response);
   res.end(JSON.stringify(response));
})
 
var server = app.listen(8081, function () {
 
  var host = server.address().address
  var port = server.address().port
 
  console.log("应用实例，访问地址为 http://%s:%s", host, port)
 
})
```

## POST方法

```html
<html>
<body>
<form action="http://127.0.0.1:8081/process_post" method="POST">
First Name: <input type="text" name="first_name">  <br>
 
Last Name: <input type="text" name="last_name">
<input type="submit" value="Submit">
</form>
</body>
</html>
```

```js
var express = require('express');
var app = express();
var bodyParser = require('body-parser');
 
// 创建 application/x-www-form-urlencoded 编码解析
var urlencodedParser = bodyParser.urlencoded({ extended: false })
 
app.use(express.static('public'));
 
app.get('/index.htm', function (req, res) {
   res.sendFile( __dirname + "/" + "index.htm" );
})
 
app.post('/process_post', urlencodedParser, function (req, res) {
 
   // 输出 JSON 格式
   var response = {
       "first_name":req.body.first_name,
       "last_name":req.body.last_name
   };
   console.log(response);
   res.end(JSON.stringify(response));
})
 
var server = app.listen(8081, function () {
 
  var host = server.address().address
  var port = server.address().port
 
  console.log("应用实例，访问地址为 http://%s:%s", host, port)
 
})
```