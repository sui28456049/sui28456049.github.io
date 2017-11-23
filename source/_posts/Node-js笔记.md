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
# main.js

var hello = require('./hello');
hello.world();

# hello.js

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

# 直接获得这个对象了
/main.js 
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



