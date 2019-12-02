---
title: python 快速运行一个应用
date: 2019-12-02 10:51:51
tags: python
category: python
---
# 安装pip```pip --version  # 检查是否安装curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py   # 下载安装脚本sudo python get-pip.py    # python2.x安装运行pip安装脚本sudo python3 get-pip.py   # python3.x安装运行pip安装脚本python 一般使用 `pip3 install````# virtualenv虚拟环境* 安装 `pip install virtualenv`* 运行切换到虚拟环境 `cd my_project && virtualenv my_project_env`* 指定环境 `virtualenv -p /usr/bin/python2.7 my_project_env`* 使用需激活 `source my_project_env/bin/activate`* 停用 `deactivate`