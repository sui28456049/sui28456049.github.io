---
title: varnish Web加速器
date: 2019-01-03 14:03:20
tags: Linux
category: Linux
---

# 简介

VarnishCache是一个Web应用加速器，也是一个知名的反向代理程序。你可以把它安装在任何http服务器的前端，同时对它进行配置来缓存内容。

工具文档: https://jefferywang.gitbooks.io/varnish_4_1_doc_zh/content/preface.html

# 基本操作

默认配置文件: /etc/varnish/default.vcl  /etc/varnish/varnish.params

/etc/varnish/default.vcl  ：主配置文件
/etc/varnish/varnish.params ：服务器参数配置文件

varnish 默认监听 127.0.0.1:6081,修改配置文件`/etc/varnish/varnish.params`的`VARNISH_LISTEN_PORT=80`监听端口为80

重载配置文件:service varnish reload
重启: service varnish restart

# VCL——varnish配置语言

参考: https://jefferywang.gitbooks.io/varnish_4_1_doc_zh/content/chapter4.html

# VCL中请求和响应对象

1：req.：由客户发来的http请求相关

req.http.：拿到请求报文各个首部值

例如：req.http.Cookie 拿到请求报文中的Cookie首部的值

2：resp.：由varnish响应给client的http响应报文

resp.http. ：响应报文的各个首部

例如：set resp.http.X-Cache = "HIT from " + server.hostname 给X-Cache首部赋值

3：bereq.：由varnish向backend主机发出的http请求 （backend request：后端主机的请求报文）

bereq.http. ：向backend主机发送的请求报文的对象

4：beresp.：由backend主机发来的http响应报文 

beresp.http. ：由backend主机发来的http的响应报文的对象

5：obj. ：存储在缓存空间中的缓存对象属性

------------------------------------------------

1：req
req.method：请求的方法
req.url：请求的URL
2：bereq
bereq.http.HEADERS：varnish的请求报文首部
bereq.request：请求方法
bereq.url：请求的URL
bereq.proto：协议版本
bereq.backend：指明要调用的后端主机
3：beresp
beresp.proto：后端主机响应给varnish的协议版本
beresp.status：响应状态码
beresp.backend.name：后端主机的主机名
beresp.http.HEADERS：后端主机响应报文的首部
beresp.ttl：后端主机响应内容ttl值
4：obj
obj.hits：对象在缓存中命中的次数
obj.ttl：对象的ttl值
5：server：
server.ip：主机的IP
server.hostname：主机名


# 我的配置

```sh
#
# This is an example VCL file for Varnish.
#
# It does not do anything by default, delegating control to the
# builtin VCL. The builtin VCL is called when there is no explicit
# return statement.
#
# See the VCL chapters in the Users Guide at https://www.varnish-cache.org/docs/
# and http://varnish-cache.org/trac/wiki/VCLExamples for more examples.

# Marker to tell the VCL compiler that this VCL has been adapted to the
# new 4.0 format.
vcl 4.0;
import directors;

# Default backend definition. Set this to point to your content server.
backend default {
    .host = "47.52.222.83";
    .port = "8080";
}

#############################（一）################################
backend web1 {
.host = "47.52.222.83";
.port = "8888";
# 指定做健康状态检查的URL，既然给了这个URL，一定需要让后端主机的站点目录里面有这个URL，如果没有varnish将不会将请求调度到这台主机上
.probe = {
    .url = "/";
    .timeout = 2s;  # 指定检查的超时时长
    .interval = 4s; # 检查每次检查的时间间隔
    .window = 5;   # 指定一个检测多少次
    .threshold = 1; # 指定最少成功多少次认为后端主机有效
    }
}

backend web2 {
.host = "121.65.111.238";
.port = "8888";
.probe = {
    .url = "/";
    .timeout = 2s;
    .interval = 4s;
    .window = 5;
    .threshold = 1;
    }
}
#############################（二）################################
# 定义负载均衡调度器
sub vcl_init {
    # 创建一个调度器对象,directors.round_robin()指定了轮询调度算法
    new static = directors.round_robin();
    static.add_backend(web1);
    static.add_backend(web2);
}

sub vcl_recv {
    #url重写，告诉后端服务器真实的请求者，安全避免重复添加，还可定义在记录日志中
    if (req.restarts == 0) {
        if (req.http.X-Fowarded-For) {
            set req.http.X-Forwarded-For = req.http.X-Forwarded-For + "," + client.ip;
        } else {
            set req.http.X-Forwarded-For = client.ip;
        }
    }         
    # 指定varnish接受到的请求，如果缓存没有命中，直接全部发往后端的static主机组
    set req.backend_hint = static.backend();

    # 处理虚拟域名
    if (req.http.host ~ "m.zxmrtop.com") {
       set req.backend_hint = default;
    }


    # 如何请求方法是pri,直接返回synth子程序
    if (req.method == "PRI") {
        /* We do not support SPDY or HTTP/2.0 */
        return (synth(405));
    }   

    # 如果不是一下七种方法的任意一个,直接返回pipe子程序
    if (req.method != "GET" &&
            req.method != "HEAD" &&
            req.method != "PUT" &&
            req.method != "POST" &&
            req.method != "TRACE" &&
            req.method != "OPTIONS" &&
            req.method != "PATCH" &&
            req.method != "DELETE") {
        /* Non-RFC2616 or CONNECT which is weird. */
        return (pipe);
    }

    # 如果不是get或者head方法,我们就不用去查缓存,直接交由后端主机处理就是
    if (req.method != "GET" && req.method != "HEAD") {
        /* We only deal with GET and HEAD by default */
        return (pass);
    }

    # 如果客户端的请求报文中的http首部包含认证信息或cookie信息,也直接交由后端主机处理,这是保密的静态内容
    # 由于在haproxy对于动态请求都会在用户的浏览器中设置cookie信息，因此，下面这项先将其注释掉，以免请求都不能查询缓存
    #if (req.http.Authorization || req.http.Cookie) {
    # /* Not cacheable by default */
    # return (pass);
    #}


    # 否则其他的请求都通过hash子程序程处理
    return (hash);

}

#############################（四）################################
# 对后端主机响应的响应报做修改
sub vcl_backend_response {

    # 如何发往后端主机的请求报文的http首部中没有s-maxage参数，那么做两个if判断
    # 1：如果请求的是css样式代码，设置响应回来的资源对象的ttl值为7200s，并且删除Set-Cookie参数
    # 2：如果请求的是jpg图片，设置响应回来的资源对象的ttl值为7200s，并且删除Set-Cookie参数
    # 其他的请求都交由deliver子程序处理
  if (beresp.http.cache-control !~ "s-maxage"){
        # 这里要写为bereq.url，如果写成beresp.url会报错
        if (bereq.url ~ "(?i)\.css$") {
            set beresp.ttl = 7200s;
            unset beresp.http.Set-Cookie;
        }
        if (bereq.url ~ "(?i)\.js$") {
            set beresp.ttl = 7200s;
            unset beresp.http.Set-Cookie;
        }
        if (bereq.url ~ "(?i)\.jpg$") {
            set beresp.ttl = 7200s;
            unset beresp.http.Set-Cookie;
        }
        if (bereq.url ~ "(?i)\.png$") {
            set beresp.ttl = 7200s;
            unset beresp.http.Set-Cookie;
        }
    }

    return (deliver);     
}

# 这一条程序是最后一个子程序
# 在deliver子历程中可以修改响应报文http的首部
sub vcl_deliver {
    # 如果客户端的请求被缓存命中,那么加一个http的首部参数，明确告诉用户请求的资源是从缓存加载的
    if (obj.hits>0)
    {
      set resp.http.X-Cache = "HIT from " + server.ip;
    }else{
      set resp.http.X-Cache = "MISS from " + server.ip;
    }
     # Remove some headers: PHP version
        unset resp.http.X-Powered-By;
     # Remove some headers: Apache version & OS
        unset resp.http.Server;
        unset resp.http.X-Drupal-Cache;
        unset resp.http.X-Varnish;
        unset resp.http.Via;
        unset resp.http.Link;
        return (deliver);
}

```
例子2:
```sh
vcl 4.0;
 
# 后端服务器配置
backend default {
  .host = "127.0.0.1"; # 后端服务器的域名或 IP
  .port = "8080";                    # 端口
  .connect_timeout = 600s;
  .first_byte_timeout = 600s;
  .between_bytes_timeout = 600s;
  .max_connections = 128;
}
 
acl purge {
    "localhost";
    "127.0.0.1";
}
 
# vcl_recv 表示 Varnish 收到客户端请求的时候
sub vcl_recv {
  # 当 HTTP 方法是 Purge 时，检查来源 IP，如果 IP 有效，则进行 Purge 操作
  if (req.method == "PURGE") {
    if (!client.ip ~ purge) {
      return(synth(405, "This IP is not allowed to send PURGE requests."));
    }
    return (purge);
  }
 
  # 不缓存有密码控制的内容和 Post 请求
  if (req.http.Authorization || req.method == "POST") {
    return (pass);
  }
 
  # 不缓存管理员页面和预览页面
  if (req.url ~ "wp-(login|admin)" || req.url ~ "preview=true") {
    return (pass);
  }
 
  # 不缓存已登录用户的内容
  if (req.http.Cookie ~ "wordpress_logged_in_") {
    return (pass);
  }
 
  # 清除 cookie，因为 WordPress 会根据用户 cookie 在评论框中直接输出昵称
  unset req.http.cookie;
 
  # 进行 hash 操作，见下面的定义
  return (hash);
}
 
sub vcl_pipe {
        return (pipe);
}
 
sub vcl_pass {
        return (fetch);
}
 
# 定义用于缓存的键
sub vcl_hash {
  # 这里使用 URL 做为键，如果是多域名站点，则需要使用 req.http.host + req.url
  hash_data(req.url);
  return (lookup);
}
 
# 处理后端服务器的响应
sub vcl_backend_response {
  # 删掉一些没有用的项
  unset beresp.http.X-Powered-By;
  unset beresp.http.x-mod-pagespeed;
 
  # 对于图片之类的静态内容，删掉 cookie 并且设置浏览器缓存时间为一个月
  if (bereq.url ~ "\.(css|js|png|gif|jp(e?)g|swf|ico|txt|eot|svg|woff)") {
    unset beresp.http.cookie;
    set beresp.http.cache-control = "public, max-age=2700000";
  }
 
  # 不缓存管理员页面和预览页面
  if (bereq.url ~ "wp-(login|admin)" || bereq.url ~ "preview=true") {
    set beresp.uncacheable = true;
    set beresp.ttl = 30s;
    return (deliver);
  }
 
  # 这一段很重要，在用户提交评论的同时，立即清空该页面的缓存，这样用户可以加载到最新的页面
  if (bereq.url == "/wp-comments-post.php") {
    ban("req.url == " + regsub(beresp.http.Location, "^http(s)?://bb\.mf8\.biz(/.*/)$
  }

  # 不缓存 Post 请求和有密码的内容
  if ( bereq.method == "POST" || bereq.http.Authorization ) {
    set beresp.uncacheable = true;
    set beresp.ttl = 120s;
    return (deliver);
  }
 
  # 只缓存正常的响应和 404
  if ( beresp.status != 200 && beresp.status != 404 ) {
    set beresp.uncacheable = true;
    set beresp.ttl = 120s;
    return (deliver);
  }
 
  unset beresp.http.set-cookie;
 
  # 默认缓存时间是 24 小时
  set beresp.ttl = 24h;
  set beresp.grace = 30s;
  return (deliver);
}
 
sub vcl_deliver {
  return (deliver);
}
sub vcl_init {
        return (ok);
}
sub vcl_fini {
        return (ok);
}
```

# varnish工具介绍

`对于缓存服务器，修改了配置，千万不能重启，重启就会清空所有的内存`

## varnishadm

### 获取帮助
varnishadm -h
### 登入到varnishadm的命令行接口
varnishadm -S /etc/varnish/secret -T 127.0.0.1:6082
### 获取命令帮助
help### 装载指定的配置文件
vcl.load <configname> <filename>
例如：vcl.load config1 default.vcl
### 使用指定的配置文件
vcl.use <configname>
例如：vcl.use config1
### 删除配置文件
vcl.discard <configname>
例如：vcl.discard boot
### 列出所有可用的配置文件
vcl.list：
例如：vcl.list
### 显示配置文件的内容
vcl.show [-v] <configname>
例如：vcl.show config1
### 列出后端可用的主机
backend.list

## varnishstat

查看varnish服务器的状态信息
