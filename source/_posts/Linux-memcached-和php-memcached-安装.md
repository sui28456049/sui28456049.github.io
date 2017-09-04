---
title: Linux memcached 和php memcached 安装
date: 2017-09-04 14:28:35
tags: php
---
> 注意:这里所有软件都是默认位置,需自行判断软件位置(--prefix=/usr/local)

## Linux(memcached安装)
### libevent 安装

memcached 依赖于 libevent

```
wget https://github.com/libevent/libevent/releases/download/release-2.1.8-stable/libevent-2.1.8-stable.tar.gz
1. ./configure 
2.  make && make test
3.  make install
``` 

### memcached 安装

#### 下载  wget http://memcached.org/latest                   
#### 1.   ./configure --prefix=/usr/local/memcached           
#### 2.    make && make test                                   
#### 3.   sudo make install

### 检查状态

#### ps aux | grep memcached
#### telnet localhost 11211   

## Php(memcached扩展安装)

php的扩展memcached，因为该扩展是依赖libmemcached的API

### libmemcached 安装

#### 下载wget https://launchpad.net/libmemcached/1.0/1.0.18/+download/libmemcached-1.0.18.tar.gz
####   ./configure -prefix=/usr/local/libmemcached --with memcached
####   make  && make install 

### memcached 扩展安装

####  ./configure --with-php-config=/usr/bin/php-config --with-libmemcached-dir=/usr/local/libmemcached 
####   make && make install  

### 添加 php.ini

#### vim /etc/php.ini 添加 extension=memcached.so
#### 重启php-fpm    service php-fpm restart
####  查看模块       php -m