---
title: ' Centos 7 PHP7.1环境配置 LNMP'
date: 2017-08-09 13:56:53
tags: php
---
> 首先更新系统软件 yum update  
  安装gcc编译器 yum install gcc && yum install gcc-c++

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=32303044&auto=1&height=66"></iframe>

#### 编译安装php7.1.8

##### 1.下载php7源码包

```bash
$ cd /root & wget -O php7.tar.gz http://am1.php.net/get/php-7.1.8.tar.gz/from/this/mirror
```
##### 2.解压源码包
```bash
$ tar -xzvf php7.tar.gz
```
##### 3.进入PHP目录
```bash
$ cd php-7.1.8
```
##### 4.安装php依赖包
```bash
$ yum install libxml2 libxml2-devel openssl openssl-devel bzip2 bzip2-devel libcurl libcurl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel gmp gmp-devel libmcrypt libmcrypt-devel readline readline-devel libxslt libxslt-devel
```
##### 5.编译配置，这一步我们会遇到很多configure error，我们一一解决，基本都是相关软件开发包没有安装导致
```bash
$ ./configure \
--prefix=/usr/local/php \
--with-config-file-path=/etc \
--enable-fpm \
--with-fpm-user=nginx  \
--with-fpm-group=nginx \
--enable-inline-optimization \
--disable-debug \
--disable-rpath \
--enable-shared  \
--enable-soap \
--with-libxml-dir \
--with-xmlrpc \
--with-openssl \
--with-mcrypt \
--with-mhash \
--with-pcre-regex \
--with-sqlite3 \
--with-zlib \
--enable-bcmath \
--with-iconv \
--with-bz2 \
--enable-calendar \
--with-curl \
--with-cdb \
--enable-dom \
--enable-exif \
--enable-fileinfo \
--enable-filter \
--with-pcre-dir \
--enable-ftp \
--with-gd \
--with-openssl-dir \
--with-jpeg-dir \
--with-png-dir \
--with-zlib-dir  \
--with-freetype-dir \
--enable-gd-native-ttf \
--enable-gd-jis-conv \
--with-gettext \
--with-gmp \
--with-mhash \
--enable-json \
--enable-mbstring \
--enable-mbregex \
--enable-mbregex-backtrack \
--with-libmbfl \
--with-onig \
--enable-pdo \
--with-mysqli=mysqlnd \
--with-pdo-mysql=mysqlnd \
--with-zlib-dir \
--with-pdo-sqlite \
--with-readline \
--enable-session \
--enable-shmop \
--enable-simplexml \
--enable-sockets  \
--enable-sysvmsg \
--enable-sysvsem \
--enable-sysvshm \
--enable-wddx \
--with-libxml-dir \
--with-xsl \
--enable-zip \
--enable-mysqlnd-compression-support \
--with-pear \
--enable-opcache
```

###### configure error:

1.configure: error: xml2-config not found. Please check your libxml2 installation.

解决：
```
$ yum install libxml2 libxml2-devel
```
2.configure: error: Cannot find OpenSSL's <evp.h>

解决：
```
$ yum install openssl openssl-devel
```
3.configure: error: Please reinstall the BZip2 distribution

解决：
```
$ yum install bzip2 bzip2-devel
```
4.configure: error: Please reinstall the libcurl distribution - easy.h should be in <curl-dir>/include/curl/

解决：
```
$ yum install libcurl libcurl-devel
```
5.If configure fails try --with-webp-dir=<DIR> configure: error: jpeglib.h not found.


解决：
```
$ yum install libjpeg libjpeg-devel
```
6.If configure fails try --with-webp-dir=<DIR>

checking for jpeg_read_header in -ljpeg... yes

configure: error: png.h not found.

解决：
```
$ yum install libpng libpng-devel
```
7.If configure fails try --with-webp-dir=<DIR>

checking for jpeg_read_header in -ljpeg... yes

checking for png_write_image in -lpng... yes

If configure fails try --with-xpm-dir=<DIR>

configure: error: freetype-config not found.

解决：
```
$ yum install freetype freetype-devel
```
8.configure: error: Unable to locate gmp.h

解决：
```
$ yum install gmp gmp-devel
```
9.configure: error: mcrypt.h not found. Please reinstall libmcrypt.

解决：
```
$ yum install libmcrypt libmcrypt-devel
```
如果显示无可用的包:
```bash
$ yum  install epel-release  //扩展包更新包
$ yum  update //更新yum源
$ yum install libmcrypt libmcrypt-devel #继续安装
```
10.configure: error: Please reinstall readline - I cannot find readline.h

解决：
```
$ yum install readline readline-devel
```
11.configure: error: xslt-config not found. Please reinstall the libxslt >= 1.1.0 distribution

解决：
```
$ yum install libxslt libxslt-devel
```
##### 6.编译与安装
```bash
$ make && make install
```
这里要make好久，要耐心一下

##### 7.添加 PHP 命令到环境变量
```bash
$ vim /etc/profile
```
在末尾加入

PATH=$PATH:/usr/local/php/bin

export PATH

要使改动立即生效执行
```bash
$ ./etc/profile
```
或 
```bash
$ source /etc/profile
```
查看环境变量
```bash
$ echo $PATH
```
查看php版本
```
$ php -v
```
##### 8.配置php-fpm
```bash
$ cp php.ini-production /etc/php.ini
```
```bash
$ cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
$ cp /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf
```
```bash 
$ cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
```
```bash
$ chmod +x /etc/init.d/php-fpm
```
##### 9.启动php-fpm
```bash
$ /etc/init.d/php-fpm start
```

$ /etc/init.d/php-fpm start
#### 安装Nginx
##### 1.安装nginx源
```bash
$ yum localinstall http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```
##### 2.安装nginx
```bash
$ yum install nginx
```
##### 3.启动nginx
```
$ service nginx start
```
Redirecting to /bin/systemctl start  nginx.service

##### 4.访问http://你的ip/
```
如果成功安装会出来nginx默认的欢迎界面
```
#### 安装MySQL5.7.*
##### 1.安装mysql源
```bash
$ yum localinstall  http://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
```
##### 2.安装mysql
```bash
$ yum install mysql-community-server
```
确认一下mysql的版本，有时可能会提示mysql5.6
##### 3.安装mysql的开发包，以后会有用
```bash
$ yum install mysql-community-devel
```
##### 4.启动mysql
```bash
$ service mysqld start
```
Redirecting to /bin/systemctl start  mysqld.service

##### 5.查看mysql启动状态
```bash
$ service mysqld status
```
出现pid

证明启动成功

##### 6.获取mysql默认生成的密码
```bash
$ grep 'temporary password' /var/log/mysqld.log
```
##### 7.换成自己的密码
```bash
$ mysql -uroot -p
```
##### 8. 更换密码
```mysql
mysql>  ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass666!';
```
注意:本地可以连接MySQL服务器,远程就不行.
```
mysql -uroot -p
use mysql;
Grant all on *.* to 'root'@'%' identified by 'root用户的密码' with grant option;
flush privileges;
然后用以下命令查看哪些用户和host可以访问，%代表任意ip地址

select user,host from user;
```
#### php和Nginx互通
##### 修改文件

```bash
$ vim /etc/nginx/conf.d/sui.conf
```
##### 配置文件
```
server 
{
    listen       80;
    server_name  localhost;
    #charset koi8-r;
    access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.php index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    location ~ \.php$ {
        root           /usr/share/nginx/html;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```
#### 编译安装php扩展
##### 1.查看自己服务器对象php版本
```
php -v
```
##### 2.根据版本下载PHP源代码
```
$ wget -O php7.tar.gz http://am1.php.net/get/php-7.1.8.tar.gz/from/this/mirror
```
##### 3、解压源码压缩包
```
$ tar -zxvf php-7.1.8.tar.gz
```
##### 4、进入源码中的ext/pcntl目录
```
cd pphp-7.1.8/ext/pcntl/
```
##### 5、运行 phpize 命令
```
phpize
```
如果报错:
Cannot find autoconf. Please check your autoconf installation and the
$PHP_AUTOCONF environment variable. Then, rerun this script.

注:如果报错,写绝对路径:sudo /usr/local/php/bin/phpize

```bash
yum install m4 
yum install autoconf
```
##### 6、运行 configure命令
```
./configure
```
##### 7、makefile
```
make && make install
```
##### 8.配置ini文件
```
通过运行 php --ini查找php.ini文件位置，然后在文件中添加extension=pcntl.so
```
##### 也可以使用pecl 安装
```
pecl install memcached
```
