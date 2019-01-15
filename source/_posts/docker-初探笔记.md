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
  -d tomcat  
 ```

 -p 8888:8080：将容器的8080端口映射到主机的8888端口

 -v $PWD/test:/usr/local/tomcat/webapps/test：将主机中当前目录下的test挂载到容器的/test

 --rm：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立即删除，除非手动 docker rm。我们这里只是随便执行个命令，看看结果，不需要排障和保留结果，因此使用 --rm 可以避免浪费空间。


### 安装wordpress


```bash
# 安装mysql
docker run --name sui-mysql -p 3307:3306 -v /var/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql

# 安装wordpress
docker run --name sui-wordpress --link sui-mysql:mysql -p 8080:80 -d wordpress
```

### 安装gitlab

```bash
sudo docker run --detach \
    --hostname www.sui.com \
    --publish 443:443 --publish 8080:80 --publish 2222:22 \
    --restart always \
    --name gitlab \
    --volume /srv/gitlab/config:/etc/gitlab \
    --volume /srv/gitlab/logs:/var/log/gitlab \
    --volume /srv/gitlab/data:/var/opt/gitlab \
      gitlab/gitlab-ce:latest
```

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

docker container prune  # 清除所有的容器

docker stop $(docker ps -q) & docker rm $(docker ps -aq)   #停止删除容器
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
docker rmi $(docker images -q)  # 删除所有镜像
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


# Dockerfile 安装Node

## 创建Node.js程序

1. 创建 package.json，并写入相关信息和依赖(/usr/local/src/node下)

```js
{
  "name": "suiweb",
  "version": "0.0.1",
  "description": "Node.js on Docker",
  "author": "MRsui",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.13.3"
  }
}
```

2. 创建服务端程序server.js(/usr/local/src/node下)

```js
    var express = require('express');

    var PORT = 8888;

    var app = express();
    app.get('/', function (req, res) {
            res.send('Hello world\n');
          });

    app.listen(PORT);
    console.log('Running on http://localhost:' + PORT);
```

## 创建Dockerfile

```
#docker.io服务器pull镜像
FROM node

#创建app目录,保存我们的代码
RUN mkdir -p /usr/local/src/node
#设置工作目录
WORKDIR /usr/local/src/node

#复制所有文件到 工作目录。
COPY . /usr/local/src/node

#编译运行node项目，使用npm安装程序的所有依赖,利用taobao的npm安装

WORKDIR /usr/local/src/node

RUN npm install --registry=https://registry.npm.taobao.org

#暴露container的端口
EXPOSE 8888

#运行命令
CMD ["npm", "start"]
```

## 构建Image

```bash
sudo docker build -t sui_node .
```
## 运行镜像

```bash
sudo docker run -d --name sui_node -p 8888:8888 sui_node
```

## 测试

```bash
curl -i localhost:8888 
```

备注: Dockerfile创建镜像,不推荐build

# Docker Compose(/19/1/14 add)

使用一个 Dockerfile 模板文件，可以让用户很方便的定义一个单独的应用容器。然而，在日常工作中，经常会碰到需要多个容器相互配合来完成某项任务的情况。例如要实现一个 Web 项目，除了 Web 服务容器本身，往往还需要再加上后端的数据库服务容器，甚至还包括负载均衡容器等。

Compose 恰好满足了这样的需求。它允许用户通过一个单独的 docker-compose.yml 模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目（project）。

Compose 中有两个重要的概念：

服务 (service)：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。

项目 (project)：由一组关联的应用容器组成的一个完整业务单元，在 docker-compose.yml 文件中定义。

`yaml语法配置项为2个空格`

## 跑起来再说

### web 应用

app.py

```python
from flask import Flask
from redis import Redis

app = Flask(__name__)
redis = Redis(host='redis', port=6379)

@app.route('/')
def hello():
    count = redis.incr('hits')
    return 'Hello World! 该页面已被访问 {} 次。\n'.format(count)

if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)
```

### Dockerfile

```sh
FROM python:3.6-alpine
ADD . /code
WORKDIR /code
RUN pip install redis flask
CMD ["python", "app.py"]
```

### docker-compose.yml

```sh
version: '3'
services:

  web:
    build: .
    ports:
     - "5000:5000"

  redis:
    image: "redis:alpine"
```

## Compose 命令说明

常用选项:

docker-compose down    此命令将会停止 up 命令所启动的容器，并移除网络

docker-compose up      启动

docker-compose build    构建（重新构建）项目中的服务容器。

docker-compose exec  nginx   进入指定的容器。

docker-compose ps   列出容器

docker-compose restart

docker-compose rm 

选项：

  -d 后台运行容器。

  --name NAME 为容器指定一个名字。

  --entrypoint CMD 覆盖默认的容器启动指令。

  -e KEY=VAL 设置环境变量值，可多次使用选项来设置多个环境变量。

  -u, --user="" 指定运行容器的用户名或者 uid。

  --no-deps 不自动启动关联的服务容器。

  --rm 运行命令后自动删除容器，d 模式下将忽略。

  -p, --publish=[] 映射容器端口到本地主机。

  --service-ports 配置服务端口并映射到本地主机。

  -T 不分配伪 tty，意味着依赖 tty 的指令将无法运行。

更多参考: https://yeasy.gitbooks.io/docker_practice/compose/commands.html#stop

## Compose 模板文件常用配置(.yaml)

1. `build`  指定 Dockerfile 所在文件夹的路径（可以是绝对路径，或者相对 docker-compose.yml 文件的路径）。 Compose 将会利用它自动构建这个镜像，然后使用这个镜像。 

2. `container_name` 指定容器名称。默认将会使用 项目名称_服务名称_序号 这样的格式。
container_name: sui-web

3. `depends_on`  解决容器的依赖、启动先后的问题。

4. `environment`  设置环境变量。

5. `expose`  暴露端口，但不映射到宿主机，只被连接的服务访问。

6. `extra_hosts 类似 Docker 中的 --add-host 参数，指定额外的 host 名称映射信息。`

```sh
extra_hosts:
 - "googledns:8.8.8.8"
 - "dockerhub:52.1.157.61"
```

会在启动后的服务容器中 /etc/hosts 文件中添加如下两条条目。

```
8.8.8.8 googledns
52.1.157.61 dockerhub
```
7. `image`   指定为镜像名称或镜像 ID

8. `logging`  日志

9. `network_mode` 设置网络模式

```sh
network_mode: "bridge"
network_mode: "host"
network_mode: "none"
network_mode: "service:[service name]"
network_mode: "container:[container name/id]"
```
10. `ports`   暴露端口信息。使用宿主端口：容器端口 (HOST:CONTAINER) 格式，或者仅仅指定容器的端口（宿主将会随机选择端口）都可以。

11. `volumes`  数据卷所挂载路径设置。可以设置宿主机路径 （HOST:CONTAINER） 或加上访问模式 （HOST:CONTAINER:ro）。


## 实例

### python (Django/PostgreSQL应用)



### php(运行wordpress)

新建一个名为 wordpress 的文件夹，然后进入这个文件夹。
```sh
version: "3"
services:

   db:
     image: mysql:5.7
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     ports:
       - "4399:3306"
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress

   wordpress:
     container_name: sui-web
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "3000:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress

   adminer:
     image: adminer
     restart: always
     ports:
       - "8080:8080"

   # 定义phpmyadmin ,80 端口冲突
   # phpmyadmin:
   #   image: phpmyadmin/phpmyadmin
   #   restart: always
   #   ports:
   #     - "8888:80"
volumes:
  db_data:
```

运行 docker-compose up -d Compose 就会拉取镜像再创建我们所需要的镜像，然后启动 wordpress 和数据库容器。 接着浏览器访问 127.0.0.1:8080 端口就能看到 WordPress 安装界面了。


# 其他工具

在线生成php docker 工具 https://phpdocker.io/generator

不错的的东西: https://laradock.io/documentation/
