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

## 

