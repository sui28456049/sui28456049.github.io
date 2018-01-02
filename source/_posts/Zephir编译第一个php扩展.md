---
title: Zephir编译第一个php扩展
date: 2018-01-02 14:36:12
tags: php
category: php
---

# 安装Zephir

```bash
git clone https://github.com/phalcon/zephir.git

cd zephir  

./install -c   # -c 全局安装
```

检测是否安装成功,若不成功安装php-zephir-parser

```bash
zephir help  
```

# 安装 php-zephir-parser

若报错提示缺少这个,安装php-zephir-parser

```bash
sudo yum install php-devel gcc make re2c autoconf #安装依赖

git clone git://github.com/phalcon/php-zephir-parser.git
cd php-zephir-parser
sudo ./install
```

添加至php.ini,重启

```bash
[Zephir Parser]
extension=zephir_parser.so

service php-fpm restart #重启php
```

# 开发第一个php扩展

```bash
zephir init tools

cd tools

sudo vim tools/tools.zep
```

## tools.zep

```php
namespace Tools;
 
class Tools
{
 
    public static function sui($data)
    {
        echo "<pre>";
        print_r($data); die();
    }
 
}
```

## 编译安装

```bash
zephir build
```

添加至php.ini,重启

```bash
[Zephir Parser]
extension=tools.so

service php-fpm restart #重启php
```

## 测试运行

```php
$data = ['test'=>666,'test666'=>333];

\Tools\Tools::sui($data);
```

