---
title: mysql优化
date: 2017-09-19 14:30:52
tags: mysql
---

# mysql 优化方向

## 1.SQL优化
     a.  sql 优化分析
     b.  索引优化
     c.  常用sql优化
     d.  常用优化技巧
## 2.优化数据库对象
     a.  优化表的数据类型
     b.  表拆分
     c.  逆规范式
     d.  使用中间表
## 3.优化Mysql Server
     a.  Mysql内存管理优化
     b.  log机制优化
     c.  调整Mysql并发相关参数
          
## 4.应用优化
     a.   数据库连接池
     b.   使用缓存减少压力
     c.   负载均衡建立集群
     d.   主主同步  主从复制 
     
# Mysql索引
     
## 索引的存储分类

*    B-TREE索引: 最常见的索引类型,大部分都支持
*    HASH索引: 只有Memory引擎支持,使用场景简单
*    R-Tree索引: 空间索引,地址空间类型数据
*    Full-text索引:全文索引,myisam的一个特殊索引,innodb 从5.6开始支持

## 查看索引

```SQL
show index from table_name
```

## 删除索引

```SQL
drop index 索引名称 on tbl_name
```

## 创建索引

```SQL
create table tbl_name(
字段名称 字段类型 [完整性约束条件],
…,
[unique|fulltext|spatial] index|key [索引名称](字段名称[(长度)] [asc|desc])
);
```

```SQL
create table test(
    -> id tinyint unsigned,
    -> username varchar(20),
    -> index in_id(id),
    -> key in_username(username)
    -> );
```
