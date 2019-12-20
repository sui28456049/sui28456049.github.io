---
title: HAProxy学习笔记
date: 2019-01-04 10:35:00
tags: Linux
category: Linux
---

# 引言

HAproxy可以代理四层七层应用,而varnish web cache 作用于七层


许多东西不比重复写: http://www.cnblogs.com/f-ck-need-u/p/7576137.html#haproxy


proxy配置部分是haproxy最重要的配置部分，包含下面四种配置段：

defaults []:设置frontend/backend/listen配置段的默认值。
frontend :配置监听客户端连接的套接字。
backend :配置haproxy所代理的后端服务器组。
listen:定义一个完整的前端和后端代理，但后端可以不定义。所以有时候等价于frontend+backend。它常用于绑定前后端1对1的情况。
所有代理的名称只能使用大写字母、小写字母、数字、-(中线)、_(下划线)、.(点号)和:(冒号)。此外，ACL名称会区分字母大小写。

目前，有两种主流的代理模式：tcp代理(即所谓的4层代理)和http代理(即所谓的7层代理)。在4层代理模式下，haproxy简单的在两端进行双向转发。在7层代理模式下，haproxy会对协议进行分析，可以根据协议来允许、阻塞、切换、增加、修改和移除request或response中的属性内容。



# 默认配置
haproxy命令行检查配置文件语法是否正确:`haproxy -f /etc/haproxy/phaproxy.cfg -c`
```
#---------------------------------------------------------------------
# Example configuration for a possible web application.  See the
# full configuration options online.
#
#   http://haproxy.1wt.eu/download/1.4/doc/configuration.txt
#
#---------------------------------------------------------------------

#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    #
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    #
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000

#---------------------------------------------------------------------
# main frontend which proxys to the backends
#---------------------------------------------------------------------
frontend  main *:5000
    acl url_static       path_beg       -i /static /images /javascript /stylesheets
    acl url_static       path_end       -i .jpg .gif .png .css .js

    use_backend static          if url_static
    default_backend             app

#---------------------------------------------------------------------
# static backend for serving up images, stylesheets and such
#---------------------------------------------------------------------
backend static
    balance     roundrobin
    server      static 127.0.0.1:4331 check

#---------------------------------------------------------------------
# round robin balancing between the various backends
#---------------------------------------------------------------------
backend app
    balance     roundrobin
    server  app1 127.0.0.1:5001 check
    server  app2 127.0.0.1:5002 check
    server  app3 127.0.0.1:5003 check
    server  app4 127.0.0.1:5004 check
```

# 小试牛刀

## 监听所有的80端口

配置文件代理模式为http，frontend定义的是监听在前端所有网卡的80端口上，此文件中只定义了一个后端服务器组backend，该backend只包含一台监听在127.0.0.1:8000的服务器。在haproxy的术语中，frontend表示的是监听套接字，用于等待客户端的连接。

```
global
        daemon
        maxconn 256

defaults
        mode http
        timeout connect 5000ms
        timeout client 50000ms
        timeout server 50000ms

frontend http-in
        bind *:80
        default_backend web_servers

backend web_servers
        server server1 127.0.0.1:8000 maxconn 32
```
如果使用listen配置方式替换backend和frontend，则更简单，以下是等价配置

```
global
        daemon
        maxconn 256

 defaults
        mode http
        timeout connect 5000ms
        timeout client 50000ms
        timeout server 50000ms

 listen http-in
        bind *:80
        server server1 127.0.0.1:8000 maxconn 32
```

## 动静分离

```sh
global
        daemon
        maxconn 256

defaults
        mode http
        timeout connect 5000ms
        timeout client 50000ms
        timeout server 50000ms

frontend http-in
        bind *:80
        acl url_static path_beg /static /images /img /css /viedo /download  
        acl url_static path_end .gif .png .jpg .css .js .bmp                
        acl host_www   path /index.html                                     
        acl url_dynamic path_end .php .php5                                 
        acl host_www    hdr_beg(Host) -i m.zxmrtop.com                
        use_backend static  if url_static
        use_backend dynamic if url_dynamic
        use_backend www if host_www

backend static
        server server1 121.65.111.138:8888 maxconn 32
backend dynamic
        server server1 47.52.222.88:8888 maxconn 32
backend www
        server server1 47.52.222.88:8080 maxconn 32
```


## 反向代理mysql

```sh
global
        maxconn 4096
        daemon
        chroot      /var/lib/haproxy
        pidfile     /var/run/haproxy.pid
        #debug
        #quiet
        user haproxy
        group haproxy
 
defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull
        log 127.0.0.1 local0
        retries 3
        option redispatch
        maxconn 2000
        #contimeout      5000
        #clitimeout      50000
        #srvtimeout      50000
        timeout http-request    10s
        timeout queue           1m
        timeout connect         10s
        timeout client          1m
        timeout server          1m
        timeout http-keep-alive 10s
        timeout check           10s
 
listen  admin_stats 0.0.0.0:8888
        mode        http
        stats uri   /dbs
        stats realm     Global\ statistics
        stats auth  admin:admin
 
listen  proxy-mysql 0.0.0.0:3307
        mode tcp
        balance leastconn # 新的连接请求被派发至具有最少连接数目的后端服务器；此算法是动态的，可以在运行时调整其权重
       # balance roundrobin
        option tcplog
        # option mysql-check user haproxy #在mysql中创建无任何权限用户haproxy，且无密码
        server MySQL1 47.52.222.25:3306 check weight 1 maxconn 2000
        server MySQL2 47.52.222.26:3306 check weight 1 maxconn 2000
        option tcpka
```

对于MySQL的负载来说，更建议采用MySQL协议感知的程序来实现，例如mysqlrouter，proxysql，maxscale，mycat等等数据库中间件。

#  转发 tcp(ssrr) 流量
 一个 vps 转发到另一个 vps 上
 ```
 global
  
 defaults
 	log	global
 	mode	tcp
 	option	dontlognull
         timeout connect 5000
         timeout client  50000
         timeout server  50000
  
 frontend ss-in
     bind *:10086
     default_backend ss-out
  
 backend ss-out
     server server1 要转发的服务器ip maxconn 20480
 ```





手册查询: https://www.cnblogs.com/f-ck-need-u/p/8502593.html#1-1-














