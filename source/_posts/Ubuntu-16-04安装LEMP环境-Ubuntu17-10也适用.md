---
title: Ubuntu 16.04安装LEMP环境(Ubuntu17.10也适用)
date: 2018-01-26 14:55:36
tags: Linux
category: Linux
---

> 最近喜欢用`Ubuntu`办公,安装一下环境吧~

直接用官方仓库的包,不想源码编译了~

# 更新软件包

```bash
sudo apt update

sudo apt upgrade
```

# 安装Nginx Web服务器

```bash
sudo apt install nginx
```

安装之后，我们可以通过运行以下命令来启动Ubuntu时自动启动Nginx。...

```bash
sudo systemctl enable nginx
```

然后用这个命令启动Nginx：

```bash
sudo systemctl start nginx
```

现在检查它的状态

```bash
systemctl status nginx
```

将www-data（Nginx用户）作为Web根目录的所有者。 默认情况下，它由root用户拥有。

sudo chown www-data:www-data /usr/share/nginx/html -R


# 安装MariaDB数据库服务器

```bash
sudo apt install mariadb-server mariadb-client
```

安装之后，MariaDB服务器应该被自动识别。 使用systemctl来检查它的状态。

```bash
systemctl status mariadb
```

启动:

```bash
sudo systemctl start mariadb
```

要启用MariaDB在引导时自动启动:

```bash
sudo systemctl enable mariadb
```

运行安装安全脚本:

```bash
sudo mysql_secure_installation
```

进入mysql:

```bash
sudo mariadb -u root
或者
sudo mysql -u root
```

# 安装php

```bash
sudo apt install php7.0 php7.0-fpm php7.0-mysql php-common php7.0-cli php7.0-common php7.0-json php7.0-opcache php7.0-readline php7.0-mbstring php7.0-xml php7.0-gd php7.0-curl  php-pear php7.0-dev php7.0-pdo
```

可以把所有`php7.0` 换成`php7.1`


查看php扩展

```bash
sudo apt-cache search php7.0-* 
```

启动php-fpm

```
sudo systemctl start php7.0-fpm

# 开机自启
sudo systemctl enable php7.0-fpm

# 检查状态

systemctl status php7.0-fpm
```

# 创建一个Nginx虚拟主机

> Nginx服务器就像Apache中的一个虚拟主机。 我们不会使用默认的服务器块，因为它不足以运行PHP代码，如果我们修改它，它变得一团糟。 因此，通过运行以下命令来删除启用了站点的目录中的默认符号链接。 （它仍然是可用的/etc/nginx/sites-available/default）

```bash
sudo rm /etc/nginx/sites-enabled/default
```

然后在/etc/nginx/conf.d/目录下创建一个全新的服务器文件。

```
sudo vim /etc/nginx/conf.d/default.conf
```

default.conf

```
server {
  listen 80;
  listen [::]:80;
  server_name localhost;
  root /usr/share/nginx/html/;
  index index.php index.html index.htm index.nginx-debian.html;

  location / {
    try_files $uri $uri/ /index.php;
  }

  error_page 404 /404.html;
  error_page 500 502 503 504 /50x.html;

  location = /50x.html {
    root /usr/share/nginx/html;
  }

  location ~ \.php$ {
    fastcgi_pass unix:/run/php/php7.0-fpm.sock;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
    include snippets/fastcgi-php.conf;
  }

  location ~ /\.ht {
    deny all;
  }
}
```

重启:

```bash
# 测试 sudo nginx -t

sudo nginx -s reload
```

# 安装PHP7.2

PHP7.2是PHP的最新稳定版本，于2017年11月30日发布，与PHP7.1相比，性能有所提升。 我们可以从Ondrej Sury[https://launchpad.net/~ondrej/+archive/ubuntu/php]添加PPA来在Ubuntu 17.10上安装PHP7.2。 那个人也是Certbot PPA的维护者。

```bash
sudo apt install software-properties-common

sudo add-apt-repository ppa:ondrej/php

sudo apt update
```

安装php
```
sudo apt install php7.2 php7.2-fpm php7.2-mysql php-common php7.2-cli php7.2-common php7.2-json php7.2-opcache php7.2-readline php7.2-mbstring php7.2-xml php7.2-gd php7.2-curl
```

现在启动PHP7.2-FPM。
```bash
sudo systemctl start php7.2-fpm
```
在系统启动时启用自动启动。
```bash
sudo systemctl enable php7.2-fpm
```
原理类似.........

可以愉快的使用乌班图了,哈哈哈~~~





