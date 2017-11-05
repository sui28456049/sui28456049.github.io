---
title: GatewayWorker研究
date: 2017-08-19 09:27:35
tags: php
category: php
---

> GatewayWorker基于Workerman开发的一个项目框架，用于快速开发TCP长连接应用，例如app推送服务端、即时IM服务端、游戏服务端、物联网、智能家居等等
GatewayWorker使用经典的Gateway和Worker进程模型。Gateway进程负责维持客户端连接，并转发客户端的数据给BusinessWorker进程处理，BusinessWorker进程负责处理实际的业务逻辑（默认调用Events.php处理业务），并将结果推送给对应的客户端。Gateway服务和BusinessWorker服务可以分开部署在不同的服务器上，实现分布式集群。
GatewayWorker提供非常方便的API，可以全局广播数据、可以向某个群体广播数据、也可以向某个特定客户端推送数据。配合Workerman的定时器，也可以定时推送数据。

## GatewayWorker 与 Workerman的关系
Workerman可以看做是一个纯粹的socket类库，可以开发几乎所有的网络应用，不管是TCP的还是UDP的，长连接的还是短连接的。Workerman代码精简，功能强大，使用灵活，能够快速开发出各种网络应用。同时Workerman相比GatewayWorker也更底层，需要开发者有一定的多进程编程经验。

因为绝大多数开发者的目标是基于Workerman开发TCP长连接应用，而长连接应用服务端有很多共同之处，例如它们有相同的进程模型以及单发、群发、广播等接口需求。所以才有了GatewayWorker框架，GatewayWorker是基于Workerman开发的一个TCP长连接框架，实现了单发、群送、广播等长连接必用的接口，并且内置了MySql类库。GatewayWorker框架实现了Gateway Worker进程模型，天然支持分布式多服务器部署，扩容缩容非常方便，能够应对海量并发连接。可以说GatewayWorker是基于Workerman实现的一个更完善的专门用于实现TCP长连接的项目框架。

## 用GatewayWorker还是Workerman？
如果你的项目是长连接并且需要客户端与客户端之间通讯，建议使用GatewayWorker。

短连接或者不需要客户端与客户端之间通讯的项目建议使用Workerman。

GatewayWorker不支持UDP监听，所以UDP服务请选择Workerman。

如果你是一个有多进程socket编程经验的人，喜欢定制自己的进程模型，可以选择Workerman。

## 进程模型[基于Gateway、Worker进程模型]
GatewayWorker分为Gateway服务BusinessWorker服务和Register服务.

Register服务类似一个全局的地址簿，Gateway进程启动后会到Register注册自己的内部通讯地址，BusinessWorker进程启动后去Register服务注册自己并查询到所有Gateway的内部通讯地址，然后与每个Gateway进程建立长连接用于后续通讯。注意Register服务本身通讯量很低，一般只有在Gateway、BusinessWorker进程启动时才会通讯，所以Register服务本身不会成为系统瓶颈。

Gateway进程负责接受客户端连接并维持这些连接，当有连接事件或者连接断开事件或者连接上有数据发来时，Gateway进程将这些事件或数据通过Gateway与BusinessWorker之前建立的连接转发给BusinessWorker，BusinessWorker进程内部根据事件及数据会默认调用Events.php的onConnect onClose onMessage 回调处理（开发者需实现onConnect onClose onMessage 里面的业务逻辑），如果有需要可以在这些回调中调用接口通过Gateway进程推送数据给任意客户端。具体接口参见《Lib\Gateway类提供的接口》一章。

Gateway只负责网络IO（非阻塞），BusinessWorker负责处理业务。由于Gateway和BusinessWorker之间是tcp长连接通讯，所以Gateway和BusinessWorker可以多机部署(分布式部署)，多机部署时只需要向一个统一的Register服务注册即可，也就是一个GatewayWorker集群只对应一个register服务，

![gateway](/uploads/gateway.png)
## 工作原理
1、Register、Gateway、BusinessWorker进程启动

2、Gateway、BusinessWorker进程启动后向Register服务进程发起长连接注册自己

3、Register服务收到Gateway的注册后，把所有Gateway的通讯地址保存在内存中

4、Register服务收到BusinessWorker的注册后，把内存中所有的Gateway的通讯地址发给BusinessWorker

5、BusinessWorker进程得到所有的Gateway内部通讯地址后尝试连接Gateway

6、如果运行过程中有新的Gateway服务注册到Register（一般是分布式部署加机器），则将新的Gateway内部通讯地址列表将广播给所有BusinessWorker，BusinessWorker收到后建立连接

7、如果有Gateway下线，则Register服务会收到通知，会将对应的内部通讯地址删除，然后广播新的内部通讯地址列表给所有BusinessWorker，BusinessWorker不再连接下线的Gateway

8、至此Gateway与BusinessWorker通过Register已经建立起长连接

9、客户端的事件及数据全部由Gateway转发给BusinessWorker处理，BusinessWorker默认调用Events.php中的onConnect onMessage onClose处理业务逻辑。

10、BusinessWorker的业务逻辑入口全部在Events.php中，包括onWorkerStart进程启动事件(进程事件)、onConnect连接事件(客户端事件)、onMessage消息事件（客户端事件）、onClose连接关闭事件（客户端事件）、onWorkerStop进程退出事件（进程事件）

## 目录结构
```
├── Applications // 这里是所有开发者应用项目
│   └── YourApp  // 其中一个项目目录，目录名可以自定义
│       ├── Events.php // 开发者只需要关注这个文件
│       ├── start_gateway.php // gateway进程启动脚本，包括端口号等设置
│       ├── start_businessworker.php // businessWorker进程启动脚本
│       └── start_register.php // 注册服务启动脚本
│
├── start.php // 全局启动脚本，此脚本会依次加载Applications/项目/start_*.php启动脚本
│
└── vendor    // GatewayWorker框架和Workerman框架源码目录，此目录开发者不用关心
```
### start_gateway.php
start_gateway.php为gateway进程启动脚本，主要定义了客户端连接的端口号、协议等信息，具体参见 Gateway类的使用一节。

客户端连接的就是start_gateway.php中初始化的Gateway端口。

### start_businessworker.php
start_businessworker.php为businessWorker进程启动脚本，也即是调用Events.php的业务处理进程，
### start_register.php
start_register.php为注册服务启动脚本，用于协调GatewayWorker集群内部Gateway与Worker的通信.

注意：客户端不要连接Register服务端口，客户端应该连接Gateway端口
## 简单聊天室
写在Events.php文件中
```php
use \GatewayWorker\Lib\Gateway;

/**
 * 主逻辑
 * 主要是处理 onConnect onMessage onClose 三个方法
 * onConnect 和 onClose 如果不需要可以不用实现并删除
 */
class Events
{
    /**
     * 当客户端连接时触发
     * 如果业务不需此回调可以删除onConnect
     * 
     * @param int $client_id 连接id
     */
    public static function onConnect($client_id) {
        // 向当前client_id发送数据 
        Gateway::sendToClient($client_id, "Hello $client_id\n");
        // 向所有人发送
        Gateway::sendToAll("$client_id login\n");
    }
    
   /**
    * 当客户端发来消息时触发
    * @param int $client_id 连接id
    * @param mixed $message 具体消息
    */
   public static function onMessage($client_id, $message) {
        // 向所有人发送 
        Gateway::sendToAll("$client_id said $message");
   }
   
   /**
    * 当用户断开连接时触发
    * @param int $client_id 连接id
    */
   public static function onClose($client_id) {
       // 向所有人发送 
       GateWay::sendToAll("$client_id logout");
   }
}
```
## Gateway类简介
文件位置：GatewayWorker/Gateway.php

Gateway类用于初始化Gateway进程。Gateway进程是暴露给客户端的让其连接的进程。所有客户端的请求都是由Gateway接收然后分发给BusinessWorker处理，同样BusinessWorker也会将要发给客户端的响应通过Gateway转发出去。

Gateway类是基于基础的Worker开发的。可以给Gateway对象的onWorkerStart、onWorkerStop、onConnect、onClose设置回调函数，但是无法给设置onMessage回调。Gateway的onMessage行为固定为将客户端数据转发给BusinessWorker。

## *BusinessWorker类简介
BusinessWorker类其实也是基于基础的Worker开发的。BusinessWorker是运行业务逻辑的进程，BusinessWorker收到Gateway转发来的事件及请求时会默认调用Events.php中的onConnect onMessage onClose方法处理事件及数据，开发者正是通过实现这些回调控制业务及流程。

### *业务处理类 Events
Events类为业务处理的入口文件，当有客户端事件发生时会触发相应的回调如下： 

1、每个BusinessWorker进程启动时，都会触发Events::onWorkerStart($businessworker)回调
2、当客户端连接到Gateway时，会触发Events::onConnect($client_id)回调。

3、当客户端发来数据时，会触发Events::onMessage($client_id, $data)回调。

4、当客户端关闭时，会触发Events::onClose($client_id)回调。

5、每个BusinessWorker进程退出时，都会触发Events::onWorkerStop($businessworker)回调
注意如果进程是非正常退出，例如被kill可能无法触发onWorkerStop。

## Register类[桥梁]
Register类其实也是基于基础的Worker开发的。Gateway进程和BusinessWorker进程启动后分别向Register进程注册自己的通讯地址，Gateway进程和BusinessWorker通过Register进程得到通讯地址后，就可以建立起连接并通讯了。

注意，客户端不要连接Register服务的端口，Register服务是GatewayWorker内部通讯用的。

Register类只能定制监听的ip和端口，并且目前只能使用text协议。

## Lib\Gateway类提供的接口
文件位置：GatewayWorker/Lib/Gateway.php

Lib\Gateway类是Gateway/BusinessWorker模型中给客户端发送数据的类。

提供了单发、群发以及关闭客户端连接的接口。