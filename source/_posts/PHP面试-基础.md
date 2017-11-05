---
title: PHP面试 - 基础
date: 2015-10-07 11:10:44
tags: 面试
category: 面试
---

# php 引用变量

引用变量: 在php中引用意味着用不同的名字访问同一个变量内容

例一:
```php
// 定义一个变量
$a = range(0, 1000);
var_dump(memory_get_usage());

// 定义变量b，将a变量的值赋值给b
$b = &$a;
var_dump(memory_get_usage());

// 对a进行修改
$a = range(0, 1000);
var_dump(memory_get_usage());
```
例二

```php
// unset 只会取消引用，不会销毁空间
$a = 1;

$b = &$a;

unset($b);

echo $a. "\n";
```
面试题:

```php
$data = ['a', 'b', 'c'];

foreach ($data as $key=>$val)
{
    $val = &$data[$key];
    var_dump($data);
}

var_dump($data);
```


# 常量及数据类型
## php 字符串的定义方式及各自区别
单双引号
Heredoc 类似于双引号
Newdoc  类似于单引号

## 哪些情况认为false

0,0.0,'','0',false,array,null

# 运算符考察点

常用运算符优先级:
递增/递减 > ! > 算术运算符 > 大小比较 > (不)相等比较 > 引用 > 位运算符(^) > 位运算(|) > 逻辑与 > 逻辑或 > 三目 > 赋值 > and > xor > or

```php
$a = 0;
$b = 0;

if ($a = 3 > 0 || $b = 3 > 0) 
{
 // a = true  b= 0;
 var_dump($a);
    $a++;
    $b++;
    echo $a. "\n";  //1
    echo $b. "\n"; //1
}

```

# 流程控制考察点

## php中如何优化多个if....elseif 语句?
1.概率大的放在第一个if
2.使用switch语句

# 超全局变量

•$GLOBALS•$_SERVER•$_GET•$_POST•$_FILES•$_COOKIE•$_SESSION•$_REQUEST•$_ENV

客户端地址:$_SERVER['REMOTE_ADDR']
服务器地址:$_SERVER['SERVER_ADDR']

# 自定义函数及内部函数
 *   内置函数

```php
$count = 5;
function get_count()
{
    static $count;  
    return $count++;
}

echo $count;   //5
echo ++$count;     //6

var_dump(get_count()) ; // null
echo get_count(); // 1
```


例子2
```php
$var1 = 5;
$var2 = 10;

function foo(&$my_var)
{
    global $var1; // 5
    $var1 += 2;   // 7
    $var2 = 4;    // 4
    $my_var += 3; // 8
    return $var2; //4
}

$my_var = 5;
echo foo($my_var). "\n"; //4
echo $my_var. "\n";     // 8
echo $var1;             //7
echo $var2;            // 10
$bar = 'foo';            
$my_var = 10;
echo $bar($my_var). "\n"; //4
```
# 文件目录相关
## 名称相关
basename(), dirname(),pathinfo().....
## 目录读取
opendir(),readdir(),closedir(),rewinddir().....
## 目录创建
mkdir()
## 目录删除
rmdir()

```php
// 将文件的内容读取出来，在开头加入Hello World
// 将拼接好的字符串写回到文件当中

$file = './hello.txt';

$handle = fopen($file, 'r');

$content = fread($handle, filesize($file));

$content = 'Hello World'. $content;

fclose($handle);

$handle = fopen($file, 'w');

fwrite($handle, $content);

fclose($handle);
```
## 遍历目录
```php
$dir = './test';

// 打开目录
// 读取目录当中的文件
// 如果文件类型是目录，继续打开目录
// 读取子目录的文件
// 如果文件类型是文件，输出文件名称
// 关闭目录
//

function loopDir($dir)
{
    $handle = opendir($dir);

    while(false!==($file = readdir($handle)))
    {
        if ($file != '.' && $file != '..')
        {
            echo $file. "\n";
            if (filetype($dir. '/'. $file) == 'dir')
            {
                loopDir($dir. '/'. $file);
            }
        }
    }
}

loopDir($dir);
```
# 正则表达式相关
基础及熟练运用

# 面向对象相关

1. php类权限修饰符
2.面向对象的多态 抽象类定义 接口的定义
3.魔术方法
4.常见设计模式 :工厂模式,单例模式 注册树模式 适配器模式 观察者模式 策略模式

# 会话控制技术 

简述cookie和session的区别以及各自的工作机制,存储位置,简述cookie优缺点

Sessino存储
session_set_save_handler() Mysql Memcache Redis 等


# 网络协议考察点

## HTTP 协议状态码

## OSI 七层模型

物理层-数据链路层-网络层-传输层-会话层-表示层-应用层
物理层: 建立 断开 物理连接
数据链路层: 建立逻辑连接 进行硬件地址寻址 差错校验等功能
网络层: 进行逻辑地址寻址, 寻找不同网络之间的路径选择
传输层: 定义传输数据的协议端口号,以及流控和差错校验(协议:tcp udp,数据包一旦离开网卡即进入网络传输层)
会话层: 建立 管理 终止会话
表示层: 数据的表示 安全 压缩
应用层: 网络服务与最终用户的一个接口

## HTTP协议的工作特点及工作原理
 工作特点: 
 1.基于B/S模式
 2.通信开销小 简单快速 传输成本低
 3.使用灵活 可使用超文本协议传输协议
 4.无状态
工作原理:
客户端发送请求给服务器,创建一个TCP连接,指定端口号,默认80,连接到服务器后,服务器监听浏览器请求,一旦监听到有客户端请求,分析请求类型后,服务器向客户端返回状态信息和数据内容
## HTTP常见请求头/响应头
## 常见网络协议及端口

