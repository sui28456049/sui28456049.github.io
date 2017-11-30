---
title: docker 初探笔记
date: 2017-12-01 06:13:19
tags: 云计算
category: 云计算
---

# 容器

## *运行容器

 运行容器比较重要

### 安装lamp
```bash

sudo docker run -p 80:80 -p 3306:3306 -v /your/path/www : /var/www 
-v /your/path/apache2.conf : /etc/apache2/apache2.conf 
-v /your/path/my.cnf : /etc/mysql/my.cnf 
-t 
-i linode/lamp /bin/bash
```

docker run：运行一个container，如果后面要绑定宿主主机的0-1024端口需要使用sudo
-p port1:port2: 将宿主机的端口port1映射到容器中的port2
-v file1:file2: 将宿主机的文件\路径挂载到容器中的文件\路径
-i --interactive               Keep STDIN open even if not attached ————交互 
-t --tty                        Allocate a pseudo-TTY————分配伪终端

### 安装tomcat镜像

 拉取镜像
 docker pull tomcat

 ```bash
  docker run --name tomcat 
  -p 8888:8080 
  -v $PWD/test:/usr/local/tomcat/webapps/test 
  -d tomcat  
 ```

 -p 8888:8080：将容器的8080端口映射到主机的8888端口
 -v $PWD/test:/usr/local/tomcat/webapps/test：将主机中当前目录下的test挂载到容器的/test


## 查看正在运行的容器

```bash
docker ps

docker ps -a  # 显示所有容器
```

## 停止容器

```bash
docker stop 容器名(或者容器id)  
```

## 移除容器

```bash
docker rm  容器名(或者容器id)  

docker restart 容器名(或者容器id)  # 重启容器

docker start  容器名(或者容器id) # 启动容器
```


# 镜像

## 查看镜像

```bash
docker images;
```


## 搜索镜像

```bash
docker search nginx(镜像名)
```
如果你不知道怎么使用这个镜像，或者这个镜像里面的初始配置，那么你可以在

`https://hub.docker.com/` 查看帮助

## 获取镜像

```bash
docker pull 
```

## 删除镜像

```bash
docker rmi (名或id)
```

# 容器间网络互通

## 查看容器ip

```bash
docker inspect 容器ID或容器名 |grep '"IPAddress"'
```



