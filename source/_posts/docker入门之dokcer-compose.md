---
title: docker入门之dokcer-compose
date: 2019-12-05 15:06:59
tags: docker
category: docker
---
>>> Docker Compose 是 Docker 官方编排（Orchestration）项目之一，负责快速的部署分布式应用。

Compose 中有两个重要的概念：

* 服务 (service)：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。

* 项目 (project)：由一组关联的应用容器组成的一个完整业务单元，在 docker-compose.yml 文件中定义。

Compose 的默认管理对象是项目，通过子命令对项目中的一组容器进行便捷地生命周期管理。

* 清空所有镜像和容器
  清空全部镜像: docker image prune
  停止所有正在运行的容器: docker stop $(docker container ls -q)
  批量清理已经停止容器: docker container prune
* 安装
```
二进制安装
$ sudo curl -L https://github.com/docker/compose/releases/download/1.24.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

$ sudo chmod +x /usr/local/bin/docker-compose

PIP 安装
$ sudo pip install -U docker-compose
```
# 命令说明

```
docker-compose [COMMAND] --help 

或者 

docker-compose help [COMMAND] 可以查看具体某个命令的使
```
https://yeasy.gitbooks.io/docker_practice/compose/commands.html

# Compose 模板文件
https://yeasy.gitbooks.io/docker_practice/compose/compose_file.html

# 创建WordPress
```yaml
version: "3"
services:

   db:
     image: mysql:8.0
     command:
      - --default_authentication_plugin=mysql_native_password
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci     
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress

   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "8000:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
volumes:
  db_data:
```

构建并运行项目: `docker-compose up -d`

# Django
## Dockerfile
因为应用将要运行在一个满足所有环境依赖的 Docker 容器里面
那么我们可以通过编辑 Dockerfile 文件来指定 Docker 容器要安装内容。
vim Dockerfile
```
FROM python:3
ENV PYTHONUNBUFFERED 1
RUN mkdir /code
WORKDIR /code
COPY requirements.txt /code/
RUN pip install -r requirements.txt
COPY . /code/
```

以上内容指定应用将使用安装了 Python 以及必要依赖包的镜像
## requirements写入依赖
vim requirements.txt
```
Django>=2.0,<3.0
psycopg2>=2.7,<3.0
```

## docker-compose.yml
vim docker-compose.yml
```
version: "3"
services:

  db:
    image: postgres

  web:
    build: .
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/code
    ports:
      - "8000:8000"
    links:
      - db
```

## 运行
结构大概是这种的
```
[root@iZj6c9rzmdqczj81nwgtqgZ docker]# tree django/
django/
├── docker-compose.yml
├── Dockerfile
└── requirements.txt
```

运行: `docker-compose run web django-admin startproject django_example .`
由于 web 服务所使用的镜像并不存在，所以 Compose 会首先使用 Dockerfile 为 web 服务构建一个镜像，
接着使用这个镜像在容器里运行 django-admin startproject django_example 指令。

这将在当前目录生成一个 Django 应用。
 ```
 ├── django_example
 │   ├── __init__.py
 │   ├── settings.py
 │   ├── urls.py
 │   └── wsgi.py
 ├── docker-compose.yml
 ├── Dockerfile
 ├── manage.py
 └── requirements.txt
 ```
 如果你的系统是 Linux,记得更改文件权限`chmod -R 777 django_example`
 
 ## 修改数据库配置
 ```
 ALLOWED_HOSTS = ['www.domain.com']  #  修改你的域名
 DATABASES = {
     'default': {
         'ENGINE': 'django.db.backends.postgresql',
         'NAME': 'postgres',
         'USER': 'postgres',
         'HOST': 'db',
         'PORT': 5432,
     }
 }
 ```
 
 运行: `docker-compose up`