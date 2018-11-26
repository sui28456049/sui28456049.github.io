---
title: 搭建Google镜像站
date: 2018-11-26 18:57:25
tags: 娱乐
category: 娱乐
---

# 搭反向代理需要的条件

1. 一台可以上网的服务器或者VPS

2. 一个域名网上有很多地方可以申请免费的域名，比如Freenom，可以免费申请.tk、.ml、.cf、.ga等后缀的顶级域名，最长一年免费，到期可以免费续期。

3. 域名的SSL证书,自签

# 下载第三方模块

## substitutions

为了在反代时，更好的替换原网页中的信息，需要在编译时增加一个第三方模块：substitutions 扩展

```bash
git clone https://github.com/yaoweibin/ngx_http_substitutions_filter_module
```

## ngx_http_google_filter_module

```bash
便捷配置反代Google的第三方模块
```


# 安装并编译Nginx

```bash
apt-get update
apt-get install libpcre3 libpcre3-dev
apt-get install zlib1g zlib1g-dev openssl libssl-dev
apt-get install libxml2 libxml2-dev libxslt1-dev
apt-get install libgd-dev libgeoip-dev
apt-get install -y gcc g++ make automake


wget http://nginx.org/download/nginx-1.9.3.tar.gz && tar -zxvf nginx-1.9.3.tar.gz && cd ~/nginx-1.9.3/

./configure --prefix=/usr/local/nginx --with-http_ssl_module  --add-module=../ngx_http_google_filter_module  --add-module=../ngx_http_substitutions_filter_module

sudo make && sudo make install

```

# 配置nginx

自签的证书当做ssl证书

```bash
scp sui.key sui.crt sui.key root@149.28.46.168:/usr/local/nginx/ssl/
```

```bash
 server {
        server_name  lucky.sui.cn;
        resolver 8.8.8.8;
        listen 443;
        ssl on;
        ssl_certificate /usr/local/nginx/ssl/sui.crt;
        ssl_certificate_key /usr/local/nginx/ssl/sui.key;

     #forbid spider
     if ($http_user_agent ~* "qihoobot|Baiduspider|Googlebot|Googlebot-Mobile|Googlebot-Image|Mediapartners-Google|Adsbot-Google|Feedfetcher-Google|Yahoo! Slurp|Yahoo! Slurp China|YoudaoBot|Sosospider|Sogou spider|Sogou web spider|MSNBot|ia_archiver|Tomato Bot")
     {
         return 403;
     }

     # 编译时加了 ngx_http_google_filter_module 模块，location的设置就非常简单
     location / {
         google on;
     }
   }
```