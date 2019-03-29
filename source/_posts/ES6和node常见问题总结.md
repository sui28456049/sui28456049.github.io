---
title: ES6和Node常见问题总结
date: 2019-03-17 07:47:30
tags: 前端
category: 前端
---

本人前端JS遇见知识点总结，遇见啥用到啥学啥，所以比较琐碎。

# 如何引入文件/模块

有时候需要封装一些公用的common,lib类似的公共文件,在其他地方对公共中定义的属性和方法进行引用

建立一个lib.js 文件，代码为下

```jsss
export function square(x) {
 return x * x;
}
export function diag(x, y) {
 return sqrt(square(x) + square(y));
}
```

## import引用（推荐）

```js
import { square, diag } from 'lib';
console.log(square(11)); // 121
console.log(diag(4, 3));
```

## require方式

```js
import * as lib from 'lib';
square = lib.square;
```

通常比较习惯用第一种。然后用import就可以得到这个数组或则参数。但是import只能用于静态导入，就是必须在文件开始的时候，在最上层就写好。而require就可以实现动态加载。

# hash模式和history模式

hash 地址带 `#` 
history 更美观,刷新404,需要nginx/apache配合

history 模式下 nginx配置

参考网址: https://router.vuejs.org/zh/guide/essentials/history-mode.html#%E5%90%8E%E7%AB%AF%E9%85%8D%E7%BD%AE%E4%BE%8B%E5%AD%90

```
location / {
  try_files $uri $uri/ /index.html;
}
```

## 原理

前端路由的核心，就在于 —— 改变视图的同时不会向后端发出请求


* hash —— 即地址栏 URL 中的 # 符号（此 hash 不是密码学里的散列运算）。比如这个 URL：http://www.abc.com/#/hello，hash 的值为 #/hello。它的特点在于：hash 虽然出现在 URL 中，但不会被包括在 HTTP 请求中，对后端完全没有影响，因此改变 hash 不会重新加载页面。

* history —— 利用了 HTML5 History Interface 中新增的 pushState() 和 replaceState() 方法。（需要特定浏览器支持）这两个方法应用于浏览器的历史记录栈，在当前已有的 back、forward、go 的基础之上，它们提供了对历史记录进行修改的功能。只是当它们执行修改时，虽然改变了当前的 URL，但浏览器不会立即向后端发送请求。


* 因此可以说，hash 模式和 history 模式都属于浏览器自身的特性，Vue-Router 只是利用了这两个特性（通过调用浏览器提供的接口）来实现前端路由.

## 总结

1 hash 模式下，仅 hash 符号之前的内容会被包含在请求中，如 http://www.abc.com，因此对于后端来说，即使没有做到对路由的全覆盖，也不会返回 404 错误。
2 history 模式下，前端的 URL 必须和实际向后端发起请求的 URL 一致，如 http://www.abc.com/book/id。如果后端缺少对 /book/id 的路由处理，将返回 404 错误。Vue-Router 官网里如此描述：“不过这种模式要玩好，还需要后台配置支持……所以呢，你要在服务端增加一个覆盖所有情况的候选资源：如果 URL 匹配不到任何静态资源，则应该返回同一个 index.html 页面，这个页面就是你 app 依赖的页面。”
3 结合自身例子，对于一般的 Vue + Vue-Router + Webpack + XXX 形式的 Web 开发场景，用 history 模式即可，只需在后端（Apache 或 Nginx）进行简单的路由配置，同时搭配前端路由的 404 页面支持。

# 快速建立Vue项目(老爱忘...)

首先安装vue-cli

```
npm install -g vue-cli
```

安装成功后可以输入vue查看相关命令的使用，输入`vue list`可以查看vue可以用的模板

```
  Available official templates:

  ★  browserify - A full-featured Browserify + vueify setup with hot-reload, linting & unit testing.
  ★  browserify-simple - A simple Browserify + vueify setup for quick prototyping.
  ★  pwa - PWA template for vue-cli based on the webpack template
  ★  simple - The simplest possible Vue setup in a single HTML file
  ★  webpack - A full-featured Webpack + vue-loader setup with hot reload, linting, testing & css extraction.
  ★  webpack-simple - A simple Webpack + vue-loader setup for quick prototyping.
```

一般都用`webpack模板`

```
vue init webpack projectName
```