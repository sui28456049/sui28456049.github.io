---
title: Linux个人复习总结
date: 2020-04-04 20:39:50
tags: Linux
cateogry: Linux
---

# 文本处理

## grep

grep [-ivnc] '需要匹配的字符串'

* -i 不区分大小写
* -c 统计包含匹配的行数
* -n 输出行号
* -v 反向匹配

## sort

sort [-ntkr] 文件名

* -n 采取数字排序
* -t 指定分隔符
* -k 指定第几列
* -r 反向排序

## uniq

uniq [-ic]

* -i 忽略大小写
* -c 计算重复行数

# 进程管理

ps [ef axu]

* -A 列出所有的进程和 -e有同样的效果
* -a 列出不和本终端有关的所有进程
* -w 显示加宽可以显示更多的信息
* -u 显示有效使用者相关的进程
* aux 显示所有包含其他使用者的进程

常用
```
ps -ef|grep php
ps -axu|grep php
```
# sed

sed(stream editor)是一种非交互的流编辑器.默认情况下,sed不会改变原文件本身,而只是对流经sed命令的文本进行修改,并将修改后的结果打印到标准输出(也是就是屏幕)

sed 处理以行为单位

注意: 要想保存修改后的文件.必须使用重定向生成新文件,直接修改文件本身"-i"参数

使用场景:

* 常规编辑器编辑困难的文件
* 过于庞大的文本(vim 一个几百m的文件)
* 有规律的文本修改

https://www.runoob.com/linux/linux-comm-sed.html

# awk 

列处理工具

https://www.runoob.com/linux/linux-comm-awk.html