---
title: docker 初探笔记
date: 2017-12-01 06:13:19
tags: 云计算
category: 云计算
---


# 配置加速器

使用阿里镜像服务器

您可以通过修改daemon配置文件/etc/docker/daemon.json来使用加速器：

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://jfhqyyka.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```


# 容器

## *运行容器

 运行容器比较重要


### 安装ubuntu

```bash
docker pull ubuntu

docker run -i -t docker.io/ubuntu /bin/bash
```

 -t:在新容器内指定一个伪终端或终端。

 -i:允许你对容器内的标准输入 (STDIN) 进行交互。


### 安装lamp
```bash

sudo docker run -p 80:80 -p 3306:3306 -v /your/path/www : /var/www 
-v /your/path/apache2.conf : /etc/apache2/apache2.conf 
-v /your/path/my.cnf : /etc/mysql/my.cnf 
-t 
-i 
linode/lamp /bin/bash
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
  --rm
  -d tomcat  
 ```

 -p 8888:8080：将容器的8080端口映射到主机的8888端口

 -v $PWD/test:/usr/local/tomcat/webapps/test：将主机中当前目录下的test挂载到容器的/test

 --rm：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动 docker rm。我们这里只是随便执行个命令，看看结果，不需要排障和保留结果，因此使用 --rm 可以避免浪费空间。

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

## 进入容器

```bash

docker exec -it 69d1 bash # 随某人建议使用这玩意(不会导致容器的停止)

docker attach 243c  # 不推荐
```

Docker 1.13+ 版本中推荐使用 docker container 来管理镜像。

$ docker container run -d
$ docker container ls
$ docker container logs

$ docker container ls

$ docker container start

$ docker container restart

$ docker container attach

$ docker container exec

$ docker container rm nginx

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

# 使用网络

## 端口映射

```bash
docker run -d \
    -p 5000:5000 \
    -p 3000:80 \
    training/webapp \
    python app.py
```

## 容器互联

### 创建一个新的 Docker 网络:

```bash
docker network create -d bridge my-net
```

-d 参数指定 Docker 网络类型，有 bridge overlay。

### 连接容器

busybox(Linux瑞士小军刀)

运行一个容器并连接到新建的 my-net 网络

```bash
$ docker run -it --rm --name box1 --net my-net busybox sh
```

打开新的终端，再运行一个容器并加入到 my-net 网络

```bash
$ docker run -it --rm --name box2 --net my-net busybox sh
```

ping 来证明 box1 容器和 box2 容器建立了互联关系

## 查看容器ip

```bash
docker inspect 容器ID或容器名 |grep '"IPAddress"'
```


# Dockerfile 定制镜像

抽空在学.......



