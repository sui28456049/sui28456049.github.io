---
title: Workerman 初探
date: 2017-08-15 11:17:28
tags: php
---
> Workerman是一个高性能的PHP socket 服务器框架，workerman基于PHP多进程以及libevent事件轮询库，PHP开发者只要实现一两个接口，便可以开发出自己的网络应用，例如Rpc服务、聊天室服务器、手机游戏服务器等。

题外话:还有一款更加强大的,名为swoole,底层为c,上层php,性能很强大.这里先学习这个,workerman也很优秀,百万级并发,完全够用.

## 简单的开发实例
###  1.使用HTTP协议对外提供Web服务
创建http_test.php文件（位置任意，能引用到Workerman/Autoloader.php即可，下同）
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/Workerman/Autoloader.php';

// 创建一个Worker监听2345端口，使用http协议通讯
$http_worker = new Worker("http://0.0.0.0:2345");

// 启动4个进程对外提供服务
$http_worker->count = 4;

// 接收到浏览器发送的数据时回复hello world给浏览器
$http_worker->onMessage = function($connection, $data)
{
    // 向浏览器发送hello world
    $connection->send('hello world');
};

// 运行worker
Worker::runAll();
```
#### 命令行运行（windows用户用 cmd命令行，下同）
```
php http_test.php start
```
#### 测试

假设服务端ip为127.0.0.1

在浏览器中访问url http://127.0.0.1:2345

注意：如果出现无法访问的情况，请检查服务器防火墙。

###  2.使用WebSocket协议对外提供服务

创建ws_test.php文件
```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/Workerman/Autoloader.php';

// 创建一个Worker监听2346端口，使用websocket协议通讯
$ws_worker = new Worker("websocket://0.0.0.0:2346");

// 启动4个进程对外提供服务
$ws_worker->count = 4;

// 当收到客户端发来的数据后返回hello $data给客户端
$ws_worker->onMessage = function($connection, $data)
{
    // 向客户端发送hello $data
    $connection->send('hello ' . $data);
};

// 运行worker
Worker::runAll();
```
#### 命令行运行
```
php ws_test.php start
```
#### 测试

打开chrome浏览器，按F12打开调试控制台，在Console一栏输入(或者把下面代码放入到html页面用js运行)
```js
// 假设服务端ip为127.0.0.1
ws = new WebSocket("ws://127.0.0.1:2346");
ws.onopen = function() {
    alert("连接成功");
    ws.send('sui');
    alert("给服务端发送一个字符串：sui");
};
ws.onmessage = function(e) {
    alert("收到服务端的消息：" + e.data);
};
```
注意：如果出现无法访问的情况，请检查服务器防火墙。

### 直接使用TCP传输数据
创建tcp_test.php

```php
<?php
use Workerman\Worker;
require_once __DIR__ . '/Workerman/Autoloader.php';

// 创建一个Worker监听2347端口，不使用任何应用层协议
$tcp_worker = new Worker("tcp://0.0.0.0:2347");

// 启动4个进程对外提供服务
$tcp_worker->count = 4;

// 当客户端发来数据时
$tcp_worker->onMessage = function($connection, $data)
{
    // 向客户端发送hello $data
    $connection->send('hello ' . $data);
};

// 运行worker
Worker::runAll();
```
#### 命令行运行
```
php tcp_test.php start
```
#### 测试：命令行运行
```
telnet 127.0.0.1 2347
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
sui
hello sui
```
注意：如果出现无法访问的情况，请检查服务器防火墙。

##  Worker类(主容器)
<b>
WorkerMan中有两个重要的类Worker与Connection。

Worker类用于实现端口的监听，并可以设置客户端连接事件、连接上消息事件、连接断开事件的回调函数，从而实现业务处理。

可以设置Worker实例的进程数（count属性），Worker主进程会fork出count个子进程同时监听相同的端口，并行的接收客户端连接，处理连接上的事件。
</b>

###  worker例子

```php
use Workerman\Worker;
require_once __DIR__ . '/Workerman/Autoloader.php';

$worker = new Worker('http://0.0.0.0:8484');


$worker->onWorkerStart = function($worker)
{
    echo "Worker starting...\n";
};

$worker->onWorkerStop = function($worker)
{
    echo "Worker stopping...\n";
};

$worker->onConnect = function($connection)
{
    echo "new connection from ip " . $connection->getRemoteIp() . "\n";
};

$worker->onClose = function($connection)
{
    echo "connection closed\n";
};
// 运行worker
Worker::runAll();
```

### 小测试
```php
use Workerman\Worker;
require_once __DIR__ . '/Workerman/Autoloader.php';

$worker = new Worker('http://0.0.0.0:8484');
$worker->onConnect = function($connection)
{
    echo "new connection from ip " . $connection->getRemoteIp() . "\n";
    print_r($connection);
};
// 运行worker
Worker::runAll();
```

运行 curl http://0.0.0.0:8484

返回的数据
```
Workerman\Connection\TcpConnection Object
(
    [onMessage] => Closure Object
        (
            [this] => Workerman\Worker Object
                (
                    [id] => 0
                    [name] => none
                    [count] => 1
                    [user] => parallels
                    [group] => 
                    [reloadable] => 1
                    [reusePort] => 
                    [onWorkerStart] => 
                    [onConnect] => Closure Object
                        (
                            [parameter] => Array
                                (
                                    [$connection] => <required>
                                )

                        )

                    [onMessage] => Closure Object
 *RECURSION*
                    [onClose] => 
                    [onError] => 
                    [onBufferFull] => 
                    [onBufferDrain] => 
                    [onWorkerStop] => 
                    [onWorkerReload] => 
                    [transport] => tcp
                    [connections] => Array
                        (
                            [1] => Workerman\Connection\TcpConnection Object
 *RECURSION*
                        )

                    [protocol] => \Workerman\Protocols\Http
                    [_autoloadRootPath:protected] => /home/sui
                    [_mainSocket:protected] => Resource id #12
                    [_socketName:protected] => http://0.0.0.0:8484
                    [_context:protected] => Resource id #7
                    [workerId] => 000000006b500573000000001a96f053
                )

        )

    [onClose] => 
    [onError] => 
    [onBufferFull] => 
    [onBufferDrain] => 
    [protocol] => \Workerman\Protocols\Http
    [transport] => tcp
    [worker] => Workerman\Worker Object
        (
            [id] => 0
            [name] => none
            [count] => 1
            [user] => parallels
            [group] => 
            [reloadable] => 1
            [reusePort] => 
            [onWorkerStart] => 
            [onConnect] => Closure Object
                (
                    [parameter] => Array
                        (
                            [$connection] => <required>
                        )

                )

            [onMessage] => Closure Object
                (
                    [this] => Workerman\Worker Object
 *RECURSION*
                )

            [onClose] => 
            [onError] => 
            [onBufferFull] => 
            [onBufferDrain] => 
            [onWorkerStop] => 
            [onWorkerReload] => 
            [transport] => tcp
            [connections] => Array
                (
                    [1] => Workerman\Connection\TcpConnection Object
 *RECURSION*
                )

            [protocol] => \Workerman\Protocols\Http
            [_autoloadRootPath:protected] => /home/sui
            [_mainSocket:protected] => Resource id #12
            [_socketName:protected] => http://0.0.0.0:8484
            [_context:protected] => Resource id #7
            [workerId] => 000000006b500573000000001a96f053
        )

    [bytesRead] => 0
    [bytesWritten] => 0
    [id] => 1
    [_id:protected] => 1
    [maxSendBufferSize] => 1048576
    [_socket:protected] => Resource id #19
    [_sendBuffer:protected] => 
    [_recvBuffer:protected] => 
    [_currentPackageLength:protected] => 0
    [_status:protected] => 2
    [_remoteAddress:protected] => 127.0.0.1:47950
    [_isPaused:protected] => 
    [_sslHandshakeCompleted:protected] => 
)

```

返回的结果是一个TcpConnection Object对象,里面有许多连接的信息.

##  TcpConnection类(连接对象)

> WorkerMan中有两个重要的类Worker与Connection。
每个客户端连接对应一个Connection对象，可以设置对象的onMessage、onClose等回调，同时提供了向客户端发送数据send接口与关闭连接close接口，以及其它一些必要的接口。
可以说Worker是一个监听容器，负责接受客户端连接，并把连接包装成connection对象式提供给开发者操作。

```php
use Workerman\Worker;
require_once __DIR__ . '/Workerman/Autoloader.php';

$worker = new Worker('websocket://0.0.0.0:8484');
// 当有客户端连接事件时
$worker->onConnect = function($connection)
{
    // 设置连接的onMessage回调
    $connection->onMessage = function($connection, $data)
    {
        var_dump($data);
        $connection->send('receive success\n');
    };
    
     // 设置连接的onClose回调
    $connection->onClose = function($connection)
    {
        echo "connection closed\n";
    };
};
// 运行worker
Worker::runAll();

```

## AsyncTcpConnection(TcpConnection的子类)
> AsyncTcpConnection是TcpConnection的子类，拥有与TcpConnection一样的属性与接口。AsyncTcpConnection用于异步创建一个TcpConnection连接。
### 异步访问外部http服务
```php
use \Workerman\Worker;
use \Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/Workerman/Autoloader.php';

$task = new Worker();
// 进程启动时异步建立一个到www.baidu.com连接对象，并发送数据获取数据
$task->onWorkerStart = function($task)
{
    $connection_to_baidu = new AsyncTcpConnection('tcp://www.baidu.com:80');
    // 当连接建立成功时，发送http请求数据
    $connection_to_baidu->onConnect = function($connection_to_baidu)
    {
        echo "connect success\n";
        $connection_to_baidu->send("GET / HTTP/1.1\r\nHost: www.baidu.com\r\nConnection: keep-alive\r\n\r\n");
    };
    $connection_to_baidu->onMessage = function($connection_to_baidu, $http_buffer)
    {
        echo $http_buffer;
    };
    $connection_to_baidu->onClose = function($connection_to_baidu)
    {
        echo "connection closed\n";
    };
    $connection_to_baidu->onError = function($connection_to_baidu, $code, $msg)
    {
        echo "Error code:$code msg:$msg\n";
    };
    $connection_to_baidu->connect();
};

// 运行worker
Worker::runAll();
```
### Mysql代理
```php
use \Workerman\Worker;
use \Workerman\Connection\AsyncTcpConnection;
require_once __DIR__ . '/Workerman/Autoloader.php';

// 真实的mysql地址，假设这里是本机3306端口
$REAL_MYSQL_ADDRESS = 'tcp://127.0.0.1:3306';

// 代理监听本地4406端口
$proxy = new Worker('tcp://0.0.0.0:4406');

$proxy->onConnect = function($connection)
{
    global $REAL_MYSQL_ADDRESS;
    // 异步建立一个到实际mysql服务器的连接
    $connection_to_mysql = new AsyncTcpConnection($REAL_MYSQL_ADDRESS);
    // mysql连接发来数据时，转发给对应客户端的连接
    $connection_to_mysql->onMessage = function($connection_to_mysql, $buffer)use($connection)
    {
        $connection->send($buffer);
    };
    // mysql连接关闭时，关闭对应的代理到客户端的连接
    $connection_to_mysql->onClose = function($connection_to_mysql)use($connection)
    {
        $connection->close();
    };
    // mysql连接上发生错误时，关闭对应的代理到客户端的连接
    $connection_to_mysql->onError = function($connection_to_mysql)use($connection)
    {
        $connection->close();
    };
    // 执行异步连接
    $connection_to_mysql->connect();

    // 客户端发来数据时，转发给对应的mysql连接
    $connection->onMessage = function($connection, $buffer)use($connection_to_mysql)
    {
        $connection_to_mysql->send($buffer);
    };
    // 客户端连接断开时，断开对应的mysql连接
    $connection->onClose = function($connection)use($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };
    // 客户端连接发生错误时，断开对应的mysql连接
    $connection->onError = function($connection)use($connection_to_mysql)
    {
        $connection_to_mysql->close();
    };

};
// 运行worker
Worker::runAll();
```

### 测试

```
mysql -uroot -P4406 -h127.0.0.1 -p

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 25004
Server version: 5.5.31-1~dotdeb.0 (Debian)

Copyright (c) 2000, 2013, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

## 定时器Timer类

### 定时函数为匿名函数，利用闭包传递参数```php
use \Workerman\Worker;
use \Workerman\Lib\Timer;
require_once __DIR__ . '/Workerman/Autoloader.php';

$ws_worker = new Worker('websocket://0.0.0.0:8080');
$ws_worker->count = 8;
// 连接建立时给对应连接设置定时器
$ws_worker->onConnect = function($connection)
{
    // 每10秒执行一次
    $time_interval = 10;
    $connect_time = time();
    // 给connection对象临时添加一个timer_id属性保存定时器id
    $connection->timer_id = Timer::add($time_interval, function()use($connection, $connect_time)
    {
         $connection->send($connect_time);
    });
};
// 连接关闭时，删除对应连接的定时器
$ws_worker->onClose = function($connection)
{
    // 删除定时器
    Timer::del($connection->timer_id);
};

// 运行worker
Worker::runAll();
```

测试
打开chrome浏览器，按F12打开调试控制台，在Console一栏输入(或者把下面代码放入到html页面用js运行)
```php
// 假设服务端ip为127.0.0.1
ws = new WebSocket("ws://127.0.0.1:8080");
ws.onopen = function() {
    alert("连接成功");
    ws.send('我要当前时间');
    //alert("给服务端发送一个字符串：sui");
};
ws.onmessage = function(e) {
    alert("收到服务端的消息：" + e.data);
};
```
