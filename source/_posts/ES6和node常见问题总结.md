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

# ES6 Promise用法

Promise是一个构造函数，自己身上有all、reject、resolve这几个方法，原型上有then、catch等同样方法。

```js
var p = new Promise(function(resolve, reject){
    //做一些异步操作
    setTimeout(function(){
        console.log('执行完成');
        resolve('随便什么数据');
    }, 2000);
});
```

Promise的构造函数接收一个参数，是函数，并且传入两个参数：resolve，reject，分别表示异步操作执行成功后的回调函数和异步操作执行失败后的回调函数。

其实这里用“成功”和“失败”来描述并不准确，按照标准来讲，resolve是将Promise的状态置为fullfiled，reject是将Promise的状态置为rejected。

只是new了一个对象，并没有调用它，我们传进去的函数就已经执行了,这是需要注意的一个细节。所以我们用Promise的时候一般是包在一个函数中，在需要的时候去运行这个函数

```js
function runAsync(){
    var p = new Promise(function(resolve, reject){
        //做一些异步操作
        setTimeout(function(){
            console.log('执行完成');
            resolve('随便什么数据');
        }, 2000);
    });
    return p;            
}
runAsync()
```

上述做了什么???

return出Promise对象，也就是说，执行这个函数我们得到了一个Promise对象,Promise对象上有then、catch方法

```
runAsync().then(function(data){
    console.log(data);
    //后面可以用传过来的数据做些其他操作
    //......
});
```

在异步操作执行完后，用链式调用的方式执行回调函数,类似与php $this->db链式操作.

## 链式操作的用法

```js
runAsync1()
.then(function(data){
    console.log(data);
    return runAsync2();
})
.then(function(data){
    console.log(data);
    return runAsync3();
})
.then(function(data){
    console.log(data);
});



function runAsync1(){
    var p = new Promise(function(resolve, reject){
        //做一些异步操作
        setTimeout(function(){
            console.log('异步任务1执行完成');
            resolve('随便什么数据1');
        }, 1000);
    });
    return p;            
}
function runAsync2(){
    var p = new Promise(function(resolve, reject){
        //做一些异步操作
        setTimeout(function(){
            console.log('异步任务2执行完成');
            resolve('随便什么数据2');
        }, 2000);
    });
    return p;            
}
function runAsync3(){
    var p = new Promise(function(resolve, reject){
        //做一些异步操作
        setTimeout(function(){
            console.log('异步任务3执行完成');
            resolve('随便什么数据3');
        }, 2000);
    });
    return p;            
}
```

在then方法中，你也可以直接return数据而不是Promise对象，在后面的then中就可以接收到数据了，比如我们把上面的代码修改成这样：

```js
runAsync1()
.then(function(data){
    console.log(data);
    return runAsync2();
})
.then(function(data){
    console.log(data);
    return '直接返回数据';  //这里直接返回数据
})
.then(function(data){
    console.log(data);
});
```

## reject用法
失败的回调函数,reject的作用就是把Promise的状态置为rejected，这样我们在then中就能捕捉到，然后执行“失败”情况的回调。

```js
function getNumber(){
    var p = new Promise(function(resolve, reject){
        //做一些异步操作
        setTimeout(function(){
            var num = Math.ceil(Math.random()*10); //生成1-10的随机数
            if(num<=5){
                resolve(num);
            }
            else{
                reject('数字太大了');
            }
        }, 2000);
    });
    return p;            
}

getNumber()
.then(
    function(data){
        console.log('resolved');
        console.log(data);
    }, 
    function(reason, data){
        console.log('rejected');
        console.log(reason);
    }
);
```

## catch用法

不会报错,捕捉异常

```js
getNumber()
.then(function(data){
    console.log('resolved');
    console.log(data);
})
.catch(function(reason){
    console.log('rejected');
    console.log(reason);
});
```

## all的用法

Promise的all方法提供了`并行执行异步`操作的能力

```
Promise
.all([runAsync1(), runAsync2(), runAsync3()])
.then(function(results){
    console.log(results);
});
```
用Promise.all来执行，all接收一个数组参数，里面的值最终都算返回Promise对象。这样，三个异步操作的并行执行的，等到它们都执行完后才会进到then里面。

那么，三个异步操作返回的数据哪里去了呢？都在then里面呢，all会把所有异步操作的结果放进一个数组中传给then，就是上面的results。

# 箭头函数

```js
// 箭头函数
let fun = (name) => {
    // 函数体
    return `Hello ${name} !`;
};

// 等同于
let fun = function (name) {
    // 函数体
    return `Hello ${name} !`;
};
```

箭头函数省去了function关键字，采用箭头=>来定义函数。函数的参数放在=>前面的括号中，函数体跟在=>后的花括号中。

关于箭头函数的参数：
① 如果箭头函数没有参数，直接写一个空括号即可。
② 如果箭头函数的参数只有一个，也可以省去包裹参数的括号。
③ 如果箭头函数有多个参数，将参数依次用逗号(,)分隔，包裹在括号中即可。

```js
// 没有参数
let fun1 = () => {
    console.log(111);
};

// 只有一个参数，可以省去参数括号
let fun2 = name => {
    console.log(`Hello ${name} !`)
};

// 有多个参数
let fun3 = (val1, val2, val3) => {
    return [val1, val2, val3];
};
```

## 箭头函数里面的函数体

① 如果箭头函数的函数体只有一句代码，就是简单返回某个变量或者返回一个简单的JS表达式，可以省去函数体的大括号{ }。

```js
let f = val => val;
// 等同于
let f = function (val) { return val };

let sum = (num1, num2) => num1 + num2;
// 等同于
let sum = function(num1, num2) {
  return num1 + num2;
};
```

② 如果箭头函数的函数体只有一句代码，就是返回一个对象，可以像下面这样写

```js
// 用小括号包裹要返回的对象，不报错
let getTempItem = id => ({ id: id, name: "Temp" });

// 但绝不能这样写，会报错。
// 因为对象的大括号会被解释为函数体的大括号
let getTempItem = id => { id: id, name: "Temp" };
```

③ 如果箭头函数的函数体只有一条语句并且不需要返回值（最常见是调用一个函数），可以给这条语句前面加一个void关键字

```js
let fn = () => void doesNotReturn();
```

## 常见用法

1. 简化函数

```js
// 例子一
// 正常函数写法
[1,2,3].map(function (x) {
  return x * x;
});

// 箭头函数写法
[1,2,3].map(x => x * x);

// 例子二
// 正常函数写法
var result = [2, 5, 1, 4, 3].sort(function (a, b) {
  return a - b;
});

// 箭头函数写法
var result = [2, 5, 1, 4, 3].sort((a, b) => a - b);
```

## 与普通函数区别

箭头函数不会创建自己的this（重要！！深入理解！！）

`箭头函数不会创建自己的this，所以它没有自己的this，它只会从自己的作用域链的上一层继承this。`

箭头函数没有自己的this，它会捕获自己在定义时（注意，是定义时，不是调用时）所处的外层执行环境的this，并继承这个this值。
所以，箭头函数中this的指向在它被定义的时候就已经确定了，之后永远不会改变。

```js
var id = 'Global';

function fun1() {
    // setTimeout中使用普通函数
    setTimeout(function(){
        console.log(this.id);
    }, 2000);
}

function fun2() {
    // setTimeout中使用箭头函数
    setTimeout(() => {
        console.log(this.id);
    }, 2000)
}

fun1.call({id: 'Obj'});     // 'Global'

fun2.call({id: 'Obj'});     // 'Obj'
```

上面这个例子，函数fun1中的setTimeout中使用普通函数，2秒后函数执行时，这时函数其实是在全局作用域执行的，所以this指向Window对象，this.id就指向全局变量id，所以输出'Global'。
但是函数fun2中的setTimeout中使用的是箭头函数，这个箭头函数的this在定义时就确定了，它继承了它外层fun2的执行环境中的this，而fun2调用时this被call方法改变到了对象{id: 'Obj'}中，所以输出'Obj'。


eg2:

```js
var id = 'GLOBAL';
var obj = {
  id: 'OBJ',
  a: function(){
    console.log(this.id);
  },
  b: () => {
      console.log(this.id);
  }
};

obj.a();    // 'OBJ'
obj.b();    // 'GLOBAL'
```

上面这个例子，对象obj的方法a使用普通函数定义的，普通函数作为对象的方法调用时，this指向它所属的对象。所以，this.id就是obj.id，所以输出'OBJ'。
但是方法b是使用箭头函数定义的，箭头函数中的this实际是继承的它定义时所处的全局执行环境中的this，所以指向Window对象，所以输出'GLOBAL'。（这里要注意，定义对象的大括号{}是无法形成一个单独的执行环境的，它依旧是处于全局执行环境中！！）

箭头函数继承而来的this指向永远不变（重要！！深入理解！！）

上面的例子，就完全可以说明箭头函数继承而来的this指向永远不变。对象obj的方法b是使用箭头函数定义的，这个函数中的this就永远指向它定义时所处的全局执行环境中的this，
即便这个函数是作为对象obj的方法调用，this依旧指向Window对象。

# ES7(async、await)异步特性

## 什么是async、await？

有一种特殊的语法可以更舒适地与promise协同工作，它叫做async/await

async顾名思义是“异步”的意思，async用于声明一个函数是异步的。而await从字面意思上是“等待”的意思，就是用于等待异步完成。并且await只能在async函数中使用

通常async、await都是跟随Promise一起使用的。为什么这么说呢？因为async返回的都是一个Promise对象同时async适用于任何类型的函数上。这样await得到的就是一个Promise对象(如果不是Promise对象的话那async返回的是什么 就是什么)；

await得到Promise对象之后就等待Promise接下来的resolve或者reject。

## Async functions

async关键字,它被放置在一个函数前面

```js
async function f() {
    return 1
}
```

函数前面的async一词意味着一个简单的事情：这个函数总是返回一个promise.
如果代码中有return <非promise>语句，JavaScript会自动把返回的这个value值包装成promise的resolved值。

```
async function f() {
    return 1
}
f().then(alert) // 1
```
上面的代码返回resolved值为1的promise

所以，async确保了函数返回一个promise，即使其中包含非promise。
还有另一个关键词await，只能在async函数里使用

## Await关键字

```js
// 只能在async函数内部使用
let value = await promise
```
关键词await可以让JavaScript进行等待，直到一个promise执行并返回它的结果，JavaScript才会继续往下执行。


以下是一个promise在1s之后resolve的例子：

```js
async function f() {
    let promise = new Promise((resolve, reject) => {
        setTimeout(() => resolve('done!'), 3000)
    })
    let result = await promise // 直到promise返回一个resolve值（*）阻塞态
    console.log(result) // 'done!' 
}
f()
```

await字面上使得JavaScript等待，直到promise处理完成，

## 注意

await不能工作在顶级作用域
那些刚开始使用await的人们老是忘记这一点，那就是我们不能将await放在代码的顶层，那样是行不通的

```
async function showAvatar() {
    // read our JSON
    let response = await fetch('/article/promise-chaining/user.json')
    let user = await response.json()
    
    // read github user
    let githubResponse = await fetch(`https://api.github.com/users/${user.name}`)
    let githubUser = await githubResponse.json()
    
    // 展示头像
    let img = document.createElement('img')
    img.src = githubUser.avatar_url
    img.className = 'promise-avatar-example'
    documenmt.body.append(img)
    
    // 等待3s
    await new Promise((resolve, reject) => {
        setTimeout(resolve, 3000)
    })
    
    img.remove()
    
    return githubUser
}
showAvatar()
```

# 覆盖组件样式方法

1.使用>>> 
```
>>> .uni-navbar__content{background: linear-gradient(top, #E1B2FE,#E2B1FD);box-shadow: none;} 

```
2. 使用/deep/

```
/deep/ .uni-navbar__content{background: linear-gradient(top, #E1B2FE,#E2B1FD);box-shadow: none;} 
```
