---
title: composer搭建一个 mvc 框架
date: 2019-12-28 13:58:24
tags: php
category: php
---

* symfony/http-foundation(HttpFoundation组件针对HTTP协议定义了一个面向对象层（ 使用 Request&& Respones)


# 搭建路由

这里我用的是 fast route , 地址 `(https://github.com/nikic/FastRoute)`

 注意: 这里需要隐藏 index.php, 不然 需要全写路径`/index.php/test` 才能找到路径
 
 nginx 隐藏 index.php
 ```
 location / {
       try_files $uri $uri/ /index.php?$args;  #  交给 index.php 里面的路由处理
     }
 ```

# 
#
