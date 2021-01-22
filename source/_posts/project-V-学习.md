---
title: project V 学习
date: 2019-12-04 10:00:58
tags: Linux
category: Linux
---

# 安装

一键安装: `bash <(curl -L -s https://install.direct/go.sh)`

* /usr/bin/v2ray/v2ray：V2Ray 程序；
* /usr/bin/v2ray/v2ctl：V2Ray 工具；
* /etc/v2ray/config.json：配置文件；
* /usr/bin/v2ray/geoip.dat：IP 数据文件
* /usr/bin/v2ray/geosite.dat：域名数据文件

#  快速入门

*   注意 json 里面去掉注释(v2ray 其实已经支持写注释了)
##  服务端 
```json
{
  "inbounds": [{
    "port": 10086, // 服务器监听端口，必须和上面的一样
    "protocol": "vmess",
    "settings": {
      "clients": [{ "id": "b831381d-6324-4d53-ad4f-8cda48b30811" }]
    }
  }],
  "outbounds": [{
    "protocol": "freedom",
    "settings": {}
  }]
}
```
##  客户端
``` json
{
	"inbounds": [{
		"port": 1080,
		"listen": "127.0.0.1",
		"protocol": "socks",
		"settings": {
			"udp": true
		}
	}],
	"outbounds": [{
		"protocol": "vmess",
		"settings": {
			"vnext": [{
				"address": "120.27.3.135", // 改为自己服务器地址
				"port": 10086,
				"users": [{
					"id": "b831381d-6324-4d53-ad4f-8cda48b30811"
				}]
			}]
		}
	}],
	"routing": {
		"domainStrategy": "IPOnDemand",
		"rules": [{
			"type": "field",
			"ip": ["geoip:private"],
			"outboundTag": "direct"
		}]
	}
}
```

 pac完整版
 ```
 // 20191204105605
 // http://127.0.0.1:8070/config.json
 
 {
   "dns": {
     "servers": [
       "8.8.8.8"
     ]
   },
   "inbounds": [
     {
       "listen": "127.0.0.1",
       "port": 1080,
       "protocol": "socks",
       "tag": "socksinbound",
       "settings": {
         "auth": "noauth",
         "udp": false,
         "ip": "127.0.0.1"
       }
     },
     {
       "listen": "127.0.0.1",
       "port": 8001,
       "protocol": "http",
       "tag": "httpinbound",
       "settings": {
         "timeout": 0
       }
     }
   ],
   "outbounds": [
     {
       "sendThrough": "0.0.0.0",
       "mux": {
         "enabled": false,
         "concurrency": 8
       },
       "protocol": "vmess",
       "settings": {
         "vnext": [
           {
             "address": "120.27.3.135",
             "users": [
               {
                 "id": "b831381d-6324-4d53-ad4f-8cda48b30811",
                 "alterId": 0,
                 "security": "auto",
                 "level": 0
               }
             ],
             "port": 10086
           }
         ]
       },
       "tag": "test",
       "streamSettings": {
         "wsSettings": {
           "path": "",
           "headers": {
             
           }
         },
         "quicSettings": {
           "key": "",
           "security": "none",
           "header": {
             "type": "none"
           }
         },
         "tlsSettings": {
           "allowInsecure": false,
           "alpn": [
             "http/1.1"
           ],
           "serverName": "server.cc",
           "allowInsecureCiphers": false
         },
         "sockopt": {
           
         },
         "httpSettings": {
           "path": "",
           "host": [
             ""
           ]
         },
         "tcpSettings": {
           "header": {
             "type": "none"
           }
         },
         "kcpSettings": {
           "header": {
             "type": "none"
           },
           "mtu": 1350,
           "congestion": false,
           "tti": 20,
           "uplinkCapacity": 5,
           "writeBufferSize": 1,
           "readBufferSize": 1,
           "downlinkCapacity": 20
         },
         "security": "none",
         "network": "tcp"
       }
     }
   ],
   "routing": {
     "name": "all_to_main",
     "domainStrategy": "AsIs",
     "rules": [
       {
         "type": "field",
         "outboundTag": "test",
         "port": "0-65535"
       }
     ]
   },
   "log": {
     "error": "/var/folders/rj/892wklf97db2hbzj9c74ydfc0000gn/T/cenmrev.v2rayx.log/error.log",
     "loglevel": "none",
     "access": "/var/folders/rj/892wklf97db2hbzj9c74ydfc0000gn/T/cenmrev.v2rayx.log/access.log"
   }
 }
 ```
 
# WebSocket+TLS+Web
TLS 的配置将写入 Nginx / Caddy / Apache 配置中，由这些软件来监听 443 端口（443 比较常用，并非 443 不可）
然后将流量转发到 V2Ray 的 WebSocket 所监听的内网端口（本例是 10086），V2Ray 服务器端不需要配置 TLS。
## 服务端v2ray 配置

```json
{
  "inbounds": [
    {
      "port": 10086,
      "listen":"127.0.0.1",//只监听 127.0.0.1，避免除本机外的机器探测到开放了 10086 端口
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "b831381d-6324-4d53-ad4f-8cda48b30811",
            "alterId": 64
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "wsSettings": {
        "path": "/ray"
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    }
  ]
}
```
## Nginx配置

```bash
server {
    listen       443 ssl;
    server_name  www.test.com;
    access_log  /webSite/access_log/test.log;
    ssl_certificate /www/server/ssl/3060613_.pem;
    ssl_certificate_key /www/server/ssl/3060613.key;

    ssl_protocols TLSv1.2 TLSv1 TLSv1.1 ;
    ssl_ciphers HIGH:!aNULL:!MD5;
    
    location /ray { # 与 V2Ray 配置中的 path 保持一致
      if ($http_upgrade != "websocket") { # WebSocket协商失败时返回404
          return 404;
      }
      proxy_redirect off;
      proxy_pass http://127.0.0.1:10086; # 假设WebSocket监听在环回地址的10000端口上
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_set_header Host $host;
      # Show real IP in v2ray access.log
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
## Caddy 配置
Caddy 会自动申请证书并自动更新，所以使用 Caddy 不用指定证书、密钥
```
domain.com
{
  log ./caddy.log
  proxy /ray localhost:10086 {
    websocket
    header_upstream -Origin
  }
}
```
## Apache 配置
```
<VirtualHost *:443>
  ServerName domain.com
  SSLCertificateFile /etc/v2ray/v2ray.crt
  SSLCertificateKeyFile /etc/v2ray/v2ray.key
  
  SSLProtocol -All +TLSv1 +TLSv1.1 +TLSv1.2
  SSLCipherSuite HIGH:!aNULL
  
  <Location "/ray/">
    ProxyPass ws://127.0.0.1:10086/ray/ upgrade=WebSocket
    ProxyAddHeaders Off
    ProxyPreserveHost On
    RequestHeader append X-Forwarded-For %{REMOTE_ADDR}s
  </Location>
</VirtualHost>
```
## v2ray客户端配置

```json
{
  "inbounds": [
    {
      "port": 1080,
      "listen": "127.0.0.1",
      "protocol": "socks",
      "sniffing": {
        "enabled": true,
        "destOverride": ["http", "tls"]
      },
      "settings": {
        "auth": "noauth",
        "udp": false
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "www.test.com",
            "port": 443,
            "users": [
              {
                "id": "b831381d-6324-4d53-ad4f-8cda48b30811",
                "alterId": 64
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "security": "tls",
        "wsSettings": {
          "path": "/ray"
        }
      }
    }
  ]
}
```
# dd

# 反向代理
A: 家里内网环境
B：拥有公网IP的环境
C: 公司网络环境


A 会主动向 B 发起请求，建立起一个连接；
用户在 C 上向 B 发起请求，欲访问 A 上的私有网盘；
B 接受 C 的请求，通过 A 向 B 建立的连接转发给 A(即 B 反向连接了 A)；



## A配置
```
{  
  "reverse":{ 
    // 这是 A 的反向代理设置，必须有下面的 bridges 对象
    "bridges":[  
      {  
        "tag":"bridge", // 关于 A 的反向代理标签，在路由中会用到
        "domain":"private.cloud.com" // A 和 B 反向代理通信的域名，可以自己取一个，可以不是自己购买的域名，但必须跟下面 B 中的 reverse 配置的域名一致
      }
    ]
  },
  "outbounds": [
    {  
      //A连接B的outbound  
      "tag":"tunnel", // A 连接 B 的 outbound 的标签，在路由中会用到
      "protocol":"vmess",
      "settings":{  
        "vnext":[  
          {  
            "address":"serveraddr.com", // B 地址，IP 或 实际的域名
            "port":16823,
            "users":[  
              {  
                "id":"b831381d-6324-4d53-ad4f-8cda48b30811",
                "alterId":64
              }
            ]
          }
        ]
      }
    },
    // 另一个 outbound，最终连接私有网盘    
    {  
      "protocol":"freedom",
      "settings":{  
      },
      "tag":"out"
    }    
  ],
  "routing":{   
    "rules":[  
      {  
        // 配置 A 主动连接 B 的路由规则
        "type":"field",
        "inboundTag":[  
          "bridge"
        ],
        "domain":[  
          "full:private.cloud.com"
        ],
        "outboundTag":"tunnel"
      },
      {  
        // 反向连接访问私有网盘的规则
        "type":"field",
        "inboundTag":[  
          "bridge"
        ],
        "outboundTag":"out"
      }
    ]
  }
}
```
## B公网环境
```
{  
  "reverse":{  //这是 B 的反向代理设置，必须有下面的 portals 对象
    "portals":[  
      {  
        "tag":"portal",
        "domain":"private.cloud.com"        // 必须和上面 A 设定的域名一样
      }
    ]
  },
  "inbounds": [
    {  
      // 接受 C 的inbound
      "tag":"external", // 标签，路由中用到
      "port":80,
      // 开放 80 端口，用于接收外部的 HTTP 访问 
      "protocol":"dokodemo-door",
        "settings":{  
          "address":"127.0.0.1",
          "port":80, //假设 NAS 监听的端口为 80
          "network":"tcp"
        }
    },
    // 另一个 inbound，接受 A 主动发起的请求  
    {  
      "tag": "tunnel",// 标签，路由中用到
      "port":16823,
      "protocol":"vmess",
      "settings":{  
        "clients":[  
          {  
            "id":"b831381d-6324-4d53-ad4f-8cda48b30811",
            "alterId":64
          }
        ]
      }
    }
  ],
  "routing":{  
    "rules":[  
      {  //路由规则，接收 C 请求后发给 A
        "type":"field",
        "inboundTag":[  
          "external"
        ],
        "outboundTag":"portal"
      },
      {  //路由规则，让 B 能够识别这是 A 主动发起的反向代理连接
        "type":"field",
        "inboundTag":[  
          "tunnel"
        ],
        "domain":[  
          "full:private.cloud.com"
        ],
        "outboundTag":"portal"
      }
    ]
  }
}
```
## C其他环境
C正常配置连接B服务器配置即可。此时C的ip就是内网A的ip。

# 反向代理2 
* 上面是局限于单端口,现在不限端口,和上面差不多,这时c连b

## A 配置(内网环境)

```
{  
  "reverse":{ 
    // 这是 A 的反向代理设置，必须有下面的 bridges 对象
    "bridges":[  
      {  
        "tag":"bridge", // 关于 A 的反向代理标签，在路由中会用到
        "domain":"pc1.localhost" // 一个域名，用于标识反向代理的流量，不必真实存在，但必须跟下面 B 中的 reverse 配置的域名一致
      }
    ]
  },
  "outbounds":[
    {  
      //A连接B的outbound  
      "tag":"tunnel", // A 连接 B的 outbound 的标签，在路由中会用到
      "protocol":"vmess",
      "settings":{  
        "vnext":[  
          {  
            "address":"serveraddr.com", //只需要变动这个, B 地址，IP 或 实际的域名
            "port":16823,
            "users":[  
              {  
                "id":"100405c9-8a69-4403-9acc-f47c10db96e9",
                "alterId":64
              }
            ]
          }
        ]
      }
    },
    // 另一个 outbound，最终连接私有网盘    
    {  
      "protocol":"freedom",
      "settings":{  
      },
      "tag":"out"
    }
  ],
  "routing":{  
    "rules":[  
      {  
      // 配置 A 主动连接 B 的路由规则
        "type":"field",
        "inboundTag":[  
          "bridge"
        ],
        "domain":[  
          "full:pc1.localhost"
        ],
        "outboundTag":"tunnel"
      },
      {  
      // 反向连接访问私有网盘的规则
        "type":"field",
        "inboundTag":[  
          "bridge"
        ],
        "outboundTag":"out"
      }
    ]    
  }
}
```
## B配置(公网服务器)
```
{  
  "reverse":{  //这是 B 的反向代理设置，必须有下面的 portals 对象
    "portals":[  
      {  
        "tag":"portal",
        "domain":"pc1.localhost"        // 必须和上面 A 设定的域名一样
      }
    ]
  },
  "inbounds":[
    {  
      // 接受 C 的inbound
      "tag":"tunnel", // 标签，路由中用到
      "port":11872,
      "protocol":"vmess",
      "settings":{  
        "clients":[  
          {  
            "id":"dd8fcd7e-45e5-4bc5-9707-9725e856c87e",
            "alterId":64
          }
        ]
      }
    },
    // 另一个 inbound，接受 A 主动发起的请求  
    {  
      "tag": "interconn",// 标签，路由中用到
      "port":16823,
      "protocol":"vmess",
      "settings":{  
        "clients":[  
          {  
            "id":"100405c9-8a69-4403-9acc-f47c10db96e9",
            "alterId":64
          }
        ]
      }
    }
  ],
  "routing":{   
    "rules":[  
      {  //路由规则，接收 C 的请求后发给 A
        "type":"field",
        "inboundTag":[  
          "interconn"
        ],
        "outboundTag":"portal"
      },
      {  //路由规则，让 B 能够识别这是 A 主动发起的反向代理连接
        "type":"field",
        "inboundTag":[  
          "tunnel"
        ],
        "domain":[  
          "full:private.cloud.com" // 将指定域名的请求发给 A，如果希望将全部流量发给 A，这里可以不设置域名规则。
        ],
        "outboundTag":"portal"
      }
    ]
  }
}
```
## C配置(用户端环境)

C连B  11872 配置即可

# 33
# 其他参考资料
*  官方文档: https://www.v2ray.com/
*  UUID 生成: https://www.uuidgenerator.net/
*  白话文: https://guide.v2fly.org/[](https://kht.zoosnet.net/LR/Chatpre.aspx?id=KHT56305690&lng=cn)
