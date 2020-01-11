---
title: 初探 express
date: 2020-01-11 09:31:23
tags: node
category: node
---

express 是一个轻量级 http 服务框架,相比 thinkphp,laravel,symonfy, 我感觉很小很简单(不知道简单不简单).上手做个小项目试一试.


# 安装 express 生成器


通过应用生成器工具 express-generator 可以快速创建一个应用的骨架。

```bash
 npm install express-generator -g
```


```
 express -h

  Usage: express [options] [dir]

  Options:

    -h, --help          输出使用方法
        --version       输出版本号
    -e, --ejs           添加对 ejs 模板引擎的支持
        --hbs           添加对 handlebars 模板引擎的支持
        --pug           添加对 pug 模板引擎的支持
    -H, --hogan         添加对 hogan.js 模板引擎的支持
        --no-view       创建不带视图引擎的项目
    -v, --view <engine> 添加对视图引擎（view） <engine> 的支持 (ejs|hbs|hjs|jade|pug|twig|vash) （默认是 jade 模板引擎）
    -c, --css <engine>  添加样式表引擎 <engine> 的支持 (less|stylus|compass|sass) （默认是普通的 css 文件）
        --git           添加 .gitignore
    -f, --force         强制在非空目录下创建
```

*  生成一个应用`myapp`

```
express -v twig myapp  # twig 模板,习惯,php 也经常使用.jsp/asp 使用 ejs, 直接上手
```


# 路由

```
# app.js 引用路由
app.use('/', indexRouter);
app.use('/users', usersRouter);

app.get('/sui',function(req, res){
  res.send('echo suisuisui');
});

app.post('/sui', function (req, res) {
  res.send('POST request to the homepage')
})

app.get('/sui/article/:id', function (req, res) {
  res.send(req.params)
})
```

#  详情: http://www.expressjs.com.cn/guide/routing.html

# 数据库


# 自动热加载

*  安装 nodemon `cnpm install -g  nodemon`
```
"scripts": {
    "start": "nodemon node ./bin/www"
  },
```

# 跨域配置

```js
app.all('*', function (req, res, next) {
  res.header('Access-Control-Allow-Origin', '*');
  res.header('Access-Control-Allow-Headers', '*');
  res.header('Access-Control-Allow-Methods', '*');
  next();
});

```