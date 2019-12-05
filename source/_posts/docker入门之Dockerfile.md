---
title: docker入门之Dockerfile
date: 2019-12-05 11:57:06
tags: docker
category: docker
---

# RUN(执行命令)
RUN 指令是用来执行命令行命令的。由于命令行的强大能力，RUN 指令在定制镜像时是最常用的指令之一
* shell 格式：RUN <命令>，就像直接在命令行中输入的命令一样。刚才写的 Dockerfile 中的 RUN 指令就是这种格式。
```
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```
* exec 格式：RUN ["可执行文件", "参数1", "参数2"]，这更像是函数调用中的格式。
```
FROM debian:stretch

RUN buildDeps='gcc libc6-dev make wget' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
```
# COPY(复制文件)
COPY 指令将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径> 位置。
```
COPY package.json /usr/src/app/
```
# ADD(高级复制文件)
ADD 指令和 COPY 的格式和性质基本一致。但是在 COPY 基础上增加了一些功能。
不推荐使用
# CMD( 容器启动)
* shell 格式：CMD <命令>
* exec 格式：CMD ["可执行文件", "参数1", "参数2"...]
Docker 不是虚拟机，容器就是进程.
既然是进程，那么在启动容器的时候，需要指定所运行的程序及参数。
`CMD 指令就是用于指定默认的容器主进程的启动命令的。`
对于容器而言，其启动程序就是容器应用进程，容器就是为了主进程而存在的.
主进程退出，容器就失去了存在的意义，从而退出，其它辅助进程不是它需要关心的东西。
```
而使用 service nginx start 命令，则是希望 upstart 来以后台守护进程形式启动 nginx 服务。
而刚才说了 CMD service nginx start 会被理解为 CMD [ "sh", "-c", "service nginx start"]，因此主进程实际上是 sh。那么当 service nginx start 命令结束后，sh 也就结束了，sh 作为主进程退出了，自然就会令容器退出。

正确的做法是直接执行 nginx 可执行文件，并且要求以前台形式运行。比如：

CMD ["nginx", "-g", "daemon off;"]
```
# ENTRYPOINT(入口点)
`ENTRYPOINT 的目的和 CMD 一样，都是在指定容器启动程序及参数。ENTRYPOINT 在运行时也可以替代.`
## 让镜像变成像命令一样使用
```
FROM ubuntu:18.04
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
ENTRYPOINT [ "curl", "-s", "https://ip.cn" ]
```

 运行
 ```
 docker build -t myip . #构建镜像
 docker run myip -i
 docker run myip
 ```
## 应用运行前的准备工作
# ENV(设置环境变量)
 两种格式
* ENV <key> <value>
* ENV <key1>=<value1> <key2>=<value2>...
# ARG(构建参数)
格式：ARG <参数名>[=<默认值>]
构建参数和 ENV 的效果一样，都是设置环境变量。所不同的是，ARG 所设置的构建环境的环境变量，在将来容器运行时是不会存在这些环境变量的。
但是不要因此就使用 ARG 保存密码之类的信息，因为 docker history 还是可以看到所有值的。
# VOLUME(定义匿名卷)
容器运行时应该尽量保持容器存储层不发生写操作,一开始就应该挂载好.
VOLUME /data
这里的 /data 目录就会在运行时自动挂载为匿名卷，任何向 /data 中写入的信息都不会记录进容器存储层，从而保证了容器存储层的无状态化。
当然，运行时可以覆盖这个挂载设置。比如：
```
docker run -d -v mydata:/data xxxx
```
# EXPOSE(暴露端口)
EXPOSE 指令是声明运行时容器提供服务端口，`这只是一个声明`，在运行时并不会因为这个声明应用就会开启这个端口的服务。
在 Dockerfile 中写入这样的声明有两个好处:
	一个是帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射；
	另一个用处则是在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口。
-p，是映射宿主端口和容器端口,就是将容器的对应端口服务公开给外界访问,而EXPOSE 仅仅是声明容器打算使用什么端口而已，并不会自动在宿主进行端口映射。
# USER(指定当前用户)
# HEALTHCHECK(健康检查)
```
FROM nginx
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
HEALTHCHECK --interval=5s --timeout=3s \
  CMD curl -fs http://localhost/ || exit 1
```
这里我们设置了每 5 秒检查一次（这里为了试验所以间隔非常短，实际应该相对较长），如果健康检查命令超过 3 秒没响应就视为失败，
并且使用 curl -fs http://localhost/ || exit 1 作为健康检查命令。
```
使用 docker build 来构建这个镜像：

$ docker build -t myweb:v1 .
$ docker run -d --name web -p 80:80 myweb:v1
```
# ONBUILD(为他人做嫁衣裳)
