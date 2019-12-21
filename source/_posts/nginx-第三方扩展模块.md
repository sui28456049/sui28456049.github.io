---
title: nginx 第三方扩展模块
date: 2019-06-10 11:18:25
tags: Linux
category: Linux
---


官方文档: `http://nginx.org/en/docs/`

官方第三方扩展地址:`https://www.nginx.com/resources/wiki/modules/index.html`

`https://www.nginx.com/products/nginx/modules`

# nginx 内置常量
```
$arg_name	请求中的name参数
$args	请求中的参数
$binary_remote_addr	远程地址的二进制表示
$body_bytes_sent	已发送的消息体字节数
$content_length	HTTP请求信息里的"Content-Length"
$content_type	请求信息里的"Content-Type"
$document_root	针对当前请求的根路径设置值
$document_uri	与$uri相同; 比如 /test2/test.php
$host	请求信息中的"Host"，如果请求中没有Host行，则等于设置的服务器名
$hostname	机器名使用 gethostname系统调用的值
$http_cookie	cookie 信息
$http_referer	引用地址
$http_user_agent	客户端代理信息
$http_via	最后一个访问服务器的Ip地址。
$http_x_forwarded_for	相当于网络访问路径
$is_args	如果请求行带有参数，返回“?”，否则返回空字符串
$limit_rate	对连接速率的限制
$nginx_version	当前运行的nginx版本号
$pid	worker进程的PID
$query_string	与$args相同
$realpath_root	按root指令或alias指令算出的当前请求的绝对路径。其中的符号链接都会解析成真是文件路径
$remote_addr	客户端IP地址
$remote_port	客户端端口号
$remote_user	客户端用户名，认证用
$request	用户请求
$request_body	这个变量（0.7.58+）包含请求的主要信息。在使用proxy_pass或fastcgi_pass指令的location中比较有意义
$request_body_file	客户端请求主体信息的临时文件名
$request_completion	如果请求成功，设为"OK"；如果请求未完成或者不是一系列请求中最后一部分则设为空
$request_filename	当前请求的文件路径名，比如/opt/nginx/www/test.php
$request_method	请求的方法，比如"GET"、"POST"等
$request_uri	请求的URI，带参数
$scheme	所用的协议，比如http或者是https
$server_addr	服务器地址，如果没有用listen指明服务器地址，使用这个变量将发起一次系统调用以取得地址(造成资源浪费)
$server_name	请求到达的服务器名
$server_port	请求到达的服务器端口号
$server_protocol	请求的协议版本，"HTTP/1.0"或"HTTP/1.1"
$uri	请求的URI，可能和最初的值有不同，比如经过重定向之类的
```

# echo 扩展

`为NGINX HTTP服务器提供熟悉的shell样式命令`

## 安装

```
./configure --prefix=/usr/local/nginx --add-module=../echo-nginx-module  
```

注意:默认nginx输出类型`default_type  application/octet-stream;`要把输出类型注释掉



## 文档

`https://github.com/openresty/echo-nginx-module`



# GeoIP2模块
* ip 地址库下载 `https://dev.maxmind.com/geoip/geoip2/geolite2/`
* 地址: `git clone https://github.com/leev/ngx_http_geoip2_module`
* 安装: 
 
 ```
 ./configure --prefix=/usr/local/nginx --add-dynamic-module=../ngx_http_geoip2_module
 ```
  问题: 需要安装 libmaxminddb 库，这个是用来读取ip数据文件的。
```
wget https://github.com/maxmind/libmaxminddb/releases/download/1.3.2/libmaxminddb-1.3.2.tar.gz
./configure
make
make install
echo /usr/local/lib  >> /etc/ld.so.conf.d/local.conf
ldconfig

下载安装 ip 库
 gunzip GeoLite2-Country.mmdb.gz 解压
```

# PHP模块
Embedded php script language for nginx-module
* 地址: `https://github.com/rryqszq4/ngx_php`
* 安装: `https://github.com/rryqszq4/ngx_php7`
# js 模块
文档: `https://nginx.org/en/docs/njs/install.html`
yum install mercurial hg # 安装依赖

安装:
```
hg clone http://hg.nginx.org/njs
./configure --add-module=path-to-njs/nginx

 ./configure --prefix=/usr/local/nginx --add-dynamic-module=../ngx_http_geoip2_module --add-module=../echo-nginx-module \
   --add-module=../njs/nginx
```