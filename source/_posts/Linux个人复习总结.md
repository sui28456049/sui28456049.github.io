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