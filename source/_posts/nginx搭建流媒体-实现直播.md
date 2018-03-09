---
title: 'nginx搭建流媒体,实现直播'
date: 2018-03-09 11:07:41
tags: Linux
category: Linux
---


# 编译nginx安装rtmp模块

rtmp下载地址:

https://github.com/arut/nginx-rtmp-module

```bash
sudo ./configure --add-module=../nginx-rtmp-module-master
sudo make
sudo make install
```


# 视频点播

编辑`nginx.conf`文件

```
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}


rtmp {                #RTMP服务
    server 
    {
        listen 1935;  #//服务端口 
        chunk_size 4096;   #//数据传输块的大小

        application video       #应用名称
	    {
	        play /home/sui/video/; #//视频文件存放位置。
	    }
    }
}
```

# 点播

/home/sui/video 目录放个视频文件

用obs软件播放


...

# 完结  ......


发现一篇写的很不错的博客,后续懒得我就不写了...

```
http://blog.csdn.net/kingroc/article/details/50839994
```








