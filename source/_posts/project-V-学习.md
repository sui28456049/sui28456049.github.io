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

*   注意 json 里面去掉注释
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
 
 # 未完待续...用啥学啥
 
 # 其他参考资料
*  官方文档: https://www.v2ray.com/
*  UUID 生成: https://www.uuidgenerator.net/