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

##  重要(本人觉得特别重要)
<b>
Connection类提供的接口
WorkerMan中有两个重要的类Worker与Connection。

每个客户端连接对应一个Connection对象，可以设置对象的onMessage、onClose等回调，同时提供了向客户端发送数据send接口与关闭连接close接口，以及其它一些必要的接口。

可以说Worker是一个监听容器，负责接受客户端连接，并把连接包装成connection对象式提供给开发者操作。
</b>

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