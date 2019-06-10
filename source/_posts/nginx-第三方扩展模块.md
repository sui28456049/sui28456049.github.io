---
title: nginx 第三方扩展模块
date: 2019-06-10 11:18:25
tags: Linux
category: Linux
---

官方文档: `http://nginx.org/en/docs/`

官方第三方扩展地址:`https://www.nginx.com/resources/wiki/modules/index.html`

`https://www.nginx.com/products/nginx/modules`

#  echo 扩展

`	为NGINX HTTP服务器提供熟悉的shell样式命令`

## 安装

```
./configure --prefix=/usr/local/nginx --add-module=../echo-nginx-module  
```

注意:默认nginx输出类型`default_type  application/octet-stream;`要把输出类型注释掉



## 文档

`https://github.com/openresty/echo-nginx-module`

