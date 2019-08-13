---
title: u盘便携式Linux系统
date: 2018-02-26 16:41:33
tags: 娱乐
category: 娱乐
---

> u盘装Linux live cd ,便携式启动,记录一下.几个迷你linux系统

# kail  Linux(nice)

官网下载地址: `https://www.kali.org/downloads/`

 烧录工具(etcher):  `https://www.balena.io/etcher/`
 
 ## u 盘持久化

参考文章: `https://docs.kali.org/downloading/kali-linux-live-usb-persistence`
加密 u 盘形式: `https://www.cnblogs.com/pingzhe/p/8425209.html`

1.  装好系统后,任意磁盘分区工具划分一个分区,`/dev/sdb3`(自己第三块分区名字)
2.  接下来，在分区中创建一个ext3文件系统，并将其标记为“persistence”。
```
mkfs.ext3 -L persistence /dev/sdb3
e2label /dev/sdb3 persistence
```
3.  创建一个挂载点，在那里挂载新分区，然后创建配置文件以启用持久性。最后，卸载分区。
```
mkdir -p /mnt/my_usb
mount /dev/sdb3 /mnt/my_usb
echo "/ union" > /mnt/my_usb/persistence.conf
umount /dev/sdb3
```

# 常用的Linux CD

## Slitaz(个人常用)

官网网址: http://www.slitaz.org/en

## CDlinux(中文友好)

官网网址:  http://cdlinux.net

## Ubuntu

官网网址: http://cdimage.ubuntu.com/releases

## Puppy

官网网址: http://www.minilinux.net

# kail

# 路由器系统

ros os 路由器系統

u盘 持久化存储 

[](https://docs.kali.org/downloading/kali-linux-live-usb-persistence)
[](https://chenjiehua.me/linux/make-a-persistent-encryped-usb-kali-linux.html)
[](http://www.secist.com/archives/1265.html)

# WinPE推荐

## 天意(个人最爱)

## 老毛桃,大白菜

# 用法

UltraISO 刻录WinPe到U盘,设置bios U盘为第一启动项,加载linux live cd iso镜像