---
title: 我不知道的 php
date: 2019-08-10 16:10:12
tags: php
category: php
---

# pack && unpack

二进制的编码包pack与解包unpack,网络编程中特别常用

## 为什么使用pack

我们知道，在网络传输过程中，我们的数据都是以二级制的网络报文在进行传输，很多时候，我们多语言之间交互的时候，想在传输的过程中约定传输的头信息，比如常见的http协议 是展示的明文传输的 

但很多时候，比如我们要做的是游戏的排行榜，而游戏的分数来自C++，每次游戏结束我们要刷新排行榜，可能我们之间并没有完整的 session会话来确保我们的通信是可靠的，这时，如果我们互相之间约定一个传输协议，比如请求头中那几位是验证信息，哪几位使我们要传输的数据，如果验证不通过，我们就认为是非法来源，直接返回403等状态码，场景大概是这样。进入pack的主题，怎么用。


```php
$username = 'sui@qq.com';
$password = '123456';
$packdata   = pack('SCSa32a32',0x0040,0x00,0x0006, $username, $password );
$unpackdata = unpack('Shead/Cflag/Sflag2/a32word/a32passd/',$packdata);
 
print_r($unpackdata);
```

手册地址:`https://www.php.net/manual/zh/function.pack.php`
参考地址:`https://blog.csdn.net/qq_34908844/article/details/79696135`
参考地址:`https://blog.csdn.net/binger819623/article/details/4314581`

