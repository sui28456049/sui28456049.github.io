---
title: 使用Aria2提升下载速度
date: 2019-11-23 09:44:19
tags: 娱乐
category: 娱乐
---

# 简介
aria2是用于下载文件的实用程序。支持的协议是HTTP（S），FTP，SFTP，BitTorrent和Metalink。
aria2可以从多个来源/协议下载文件，并尝试利用最大下载带宽。
它支持同时从HTTP（S）/ FTP / SFTP和BitTorrent下载文件，而从HTTP（S）/ FTP / SFTP下载的数据上传到BitTorrent群。
使用Metalink的块校验和，aria2在下载BitTorrent之类的文件时会自动验证数据块。

```
地址: https://github.com/aria2/aria2

```

# 安装 aria2
   mac os:
   使用 `brew install aria2` 安装  路径`/usr/local/Cellar/aria2/1.35.0/bin` 填到环境变量
   往~/.bash_profile中添加export PATH=$PATH:/usr/local/Cellar/aria2/1.35.0/bin；
   source ~/.bash_profile使环境变量生效
   `aria2c -v`
  
# 配置
aria2默认会读取~/.aria2/aria2.conf中的配置

vim ~/.aria2/aria2.conf

```bash
rpc-secret=123456_token 
enable-rpc=true
rpc-allow-origin-all=true
rpc-listen-all=true
#rpc-listen-port=6800

max-concurrent-downloads=5
continue=true
lowest-speed-limit=0
max-connection-per-server=5
min-split-size=10M
split=10

max-overall-download-limit=0
max-download-limit=0
max-overall-upload-limit=0
max-upload-limit=0

dir=/Users/Download
file-allocation=prealloc

input-file=/Users/apple/.aria2/aria2c/aria2.session
save-session=/Users/apple/Download/.aria2/aria2c/aria2.session
save-session-interval=60
```

# aria2 命令

* 在终端输入aria2c -D即可在后台运行aria2服务。

## 多线程下载
```
aria2c -x 16 https://mirrors.dtops.cc/iso/MacOS/daliansky_macos/macOS%20Catalina%2010.15.1%2819B88%29%20Installer%20for%20Clover%205098%20and%20WEPE.dmg
```

# 安装(AriaNg)图形界面
   
```
https://github.com/mayswind/AriaNg
```

[](https://www.jianshu.com/p/65240657bbb5)



