---
title: phpstorm xdebug本地调试与远程调试
date: 2019-04-05 16:23:14
tags: php
category: php
---

# 安装xdebug

我用`brew install php72-xdebug`安装不上

使用`pecl install xdebug`安装


# phpstorm配置Xdebug 步过 步入 步出意思

步过，就是遇到方法，不进入，直接下一行

步入，就是遇到函数会进入函数

步出，就是运行到退出本函数、返回上一级的下一行


# 本地调试

## 本地php.ini配置文件

```
[xdebug]
#/usr/local/Cellar/php@7.2/7.2.18/pecl/20170718/xdebug.so
zend_extension="xdebug.so"
xdebug.remote_enable = 1
xdebug.remote_handler = dbgp
xdebug.remote_host = 127.0.0.1
xdebug.remote_port = 9001
xdebug.idekey = PHPSTORM
```

## phpstorm配置

![1](/uploads/phpstorm1.png)
![1](/uploads/phpstorm2.png)
![1](/uploads/phpstorm3.png)
![1](/uploads/phpstorm4.png)

* Start URL 可以设置为入口文件`/index.php`
* 记得点击绿色的监听`电话筒`按钮

# 远程调试





# ssh远程端口转发调试(远程调试xdebug调试)

## 服务器php.ini设置
```
zend_extension=xdebug.so
xdebug.remote_enable=On
xdebug.remote_port=11955
;xdebug.idekey=PHPSTORM
```
如果使用宝塔,开放11955端口

## phpstorm客户端操作
![1](/uploads/xdebug1.png)
注意:(我不配public 目录也可以)这里要配置2个映射,不然单一入口文件找不到入口(具体参考:
https://www.jetbrains.com/help/phpstorm/validating-the-configuration-of-the-debugging-engine.html)
![1](/uploads/xdebug2.png)
![1](/uploads/xdebug3.png)
设置本地目录和服务器目录程序对应

## ssh客户端操作

1.客户端php安装xdebug(不是必须)
2.确保xdebug监听端口运行(phpstorm配置的xdebug端口)

```
lsof -i:9100
COMMAND   PID USER   FD   TYPE            DEVICE SIZE/OFF NODE NAME
phpstorm 1115  sui  485u  IPv4 0x272f4a4103ca923      0t0  TCP *:hp-pdl-datas
```
3.ssh转发

```
ssh -NT -R 11955:127.0.0.1:9100 root@119.45.5.209

# ssh -NT -R 11955:127.0.0.1:9100 root@ip
# HOST 为远程服务器,可以替换为你的 比如 root@1.1.1.1
# 这样就实现了远程xdebug端口11955到本地9100的映射。
```
 phpstorm监听的9100端口转发到远程服务器上的11955 实现远程调试
> 调试的时候定义一个入口文件,index.php/dev.php ,点phpstorm 甲壳虫,刷新浏览器就可以.





```
http://120.27.3.185
https://cloud.tencent.com/developer/article/1561767
```


# 超时设置



```
　　1. apache module的情况下:

　　　修改配置文件 httpd/conf.d/fcgid.conf

　　　FcgidIOTimeout 3600

　　2.nginx , php-fpm的情况下:

　　　　修改配置文件 php-fpm.conf

　　　　request_terminate_timeout = 3600
```
