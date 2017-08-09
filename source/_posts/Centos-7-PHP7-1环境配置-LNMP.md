---
title: ' Centos 7 PHP7.1环境配置 LNMP'
date: 2017-08-09 13:56:53
tags: php
---
> 首先更新系统软件 yum update  
  安装gcc编译器 yum install gcc && yum install gcc-c++

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
