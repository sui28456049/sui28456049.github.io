---
title: openresty从入门到放弃
date: 2019-12-07 11:56:45
tags: Linux
category: Linux
---

> openresty是一款结合了nginx和lua的全功能web服务器，既是一个中间件，也结合了一个后端解释器。所以，我们可以在nginx上用lua开发很多“有趣”的东西。

# 下载安装

 需要一个 nginx 第三方内容替换库
`https://github.com/yaoweibin/ngx_http_substitutions_filter_module`

```
cd /usr/local/src/
git clone https://github.com/yaoweibin/ngx_http_substitutions_filter_module
wget https://openresty.org/download/openresty-1.15.8.2.tar.gz #  下载
tar -xzvf openresty-1.15.8.2.tar.gz
cd openresty-1.15.8.2
#  编译安装
./configure --with-http_sub_module --with-pcre-jit --with-ipv6 --add-module=../ngx_http_substitutions_filter_module

make && make install
```

编译选项介绍 
* -—with-http_sub_module 附带http_sub_module模块，这是nginx自带的一个模块，用来替换返回的http数据包中内容。 
* --with-pcre-jit —with-ipv6 提供ipv6支持 
* --add-module=../ngx_http_substitutions_filter_module
（此处为你下载的ngx_http_substitutions_filter_module目录） 将刚才下载的http_substitutions_filter_module模块编译进去。
 http_substitutions_filter_module模块是http_sub_module的加强版，它可以用正则替换，并可以多处替换。
 
 默认在/usr/local/openresty
 
# 反向代理网站 
## 例子一
```
  server {
        listen       80;
        server_name  www.sui666.tk;
        index index.html index.htm index.php index;

        location / {
            proxy_pass https://dst_host; # dst_host为目标网站,eg:www.baidu.com
            proxy_cookie_domain dst_host www.sui666.tk;
            proxy_buffering off;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header User-Agent $http_user_agent;
            proxy_set_header referer "https://dst_host$request_uri";
        }
       
    }
```
* proxy_pass 是将请求交给上游处理，而这里的上游就是https://dst_host
 
* proxy_cookie_domain是将所有cookie中的domain替换掉成自己的domain，达到能够登陆的效果。
 
* proxy_buffering off用来关闭内存缓冲区。
 
* proxy_set_header是一个重要的配置项，利用这个项可以修改转发时的HTTP头。
* 比如，在登录网站后，修改资料的时候会验证referer，如果referer来自其它网站是会提示错误的。
* 所以我在这里用proxy_set_header将referer设置为上游域名下的地址，从而绕过检查。

## 代理百度
```
server {
        listen       80;
        server_name  www.sui666.tk;
        index index.html index.htm index.php index;

        location / {
            proxy_pass https://www.baidu.com; # nginx配置中也是可以将https降成http
            proxy_buffering off;
            proxy_cookie_domain www.baidu.com www.sui666.tk;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header User-Agent1 $http_user_agent;
            proxy_set_header Sui 'sui test';
            proxy_set_header Accept-Encoding ""; # 重要 在向上层服务器转发数据包的时候，设置proxy_set_header Accept-Encoding ””，这样后端服务器就不会压缩数据包了。
            proxy_set_header referer "https://www.baidu.com$request_uri";
            subs_filter www.baidu.com www.sui666.tk;
            subs_filter hao123 hao345;
            subs_filter 百度一下 百度一下喽;
			subs_filter </head> "<script>alert('suisuissui');</script></head>" # 前端的数据截取
        }
       
    }
```
在反代过程中，我们会常常和gzip打交道。
熟悉http协议的同学应该知道，如果浏览器发送的数据包头含有Accept-Encoding: gzip，即告诉服务器：“我可以接受gzip压缩过的数据包”。
这时后端就会将返回包压缩后发送，并包含返回头Content-Encoding: gzip，
浏览器根据是否含有这个头对返回数据包进行解压显示。
但gzip在反代中，会造成很大问题：subs_filter替换内容时，如果内容是压缩过的，明显就不能正常替换了

* 所以网上一般处理方式是，在向上层服务器转发数据包的时候，设置proxy_set_header Accept-Encoding ””，这样后端服务器就不会压缩数据包了。
* 有时候就算发送Accept-Encoding=""也不管用。各种header头设置缓存也无效
* 借助lua，通过lua脚本在数据包返回的时候解压缩gzip数据，并代替subs_filter进行字符串的替换。

## lua进行gzip解压与返回包修改
openresty在编译安装的时候就加入了lua支持，所以无需再对nginx进行改造。
但lua下对gzip进行解压，需要借助一个库：lua-zlib（https://github.com/brimworks/lua-zlib） 
lua是一个和C语言结合紧密的脚本语言，实际上lua-zlib就是一个C语言编写的库，
1 .`我们现在需要做的就是将其编译成一个动态链接库zlib.so，让lua来引用。`

```bash
git clone https://github.com/brimworks/lua-zlib.git
cd lua-zlib
cmake -DLUA_INCLUDE_DIR=/usr/local/openresty/luajit/include/luajit-2.1 -DLUA_LIBRARIES=/usr/local/openresty/luajit/lib -DUSE_LUAJIT=ON -DUSE_LUA=OFF
make && make install

cp zlib.so /usr/local/openresty/lualib/zlib.so
```
执行cmake来生成编译配置文件。LUA_INCLUDE_DIR指定luajit的include文件夹，LUA_LIBRARIES指定luajit的lib文件夹。
USE_LUAJIT=ON和USE_LUA=OFF指定我们使用的是luajit而不是lua
nginx的配置文件 加入，
"body_filter_by_lua_file /usr/local/openresty/luasrc/repl.lua; "
这句话告诉nginx我需要把返回包的body交给repl.lua处理。
....


# 参考
* https://wooyun.js.org/drops/openresty+lua%E5%9C%A8%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86%E6%9C%8D%E5%8A%A1%E4%B8%AD%E7%9A%84%E7%8E%A9%E6%B3%95.html

# 题外话
少折腾..  nginx 自带 ngxscript也可以运行 js,php,python 虚拟机,用到再说吧.