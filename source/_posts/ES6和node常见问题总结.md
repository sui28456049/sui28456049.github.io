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