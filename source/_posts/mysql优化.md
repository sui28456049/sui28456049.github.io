---
title: mysql优化
date: 2017-09-19 14:30:52
tags: mysql
category: mysql
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
     a.   数据库连接池(php-cp(php-connect-pool)是用php扩展写的一个数据库连接池。)
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

# 数据对象优化

## 优化表数据类型

  使用procedure analyse()对当前应用的表进行分析,他会优化建议,用户根据实际情况优化.

 ```sql
 select * from test procedure analyse();
 ```

 ## 表拆分

  *   垂直拆分(针对某些列常用,某些列可能不常用)

  *   水平拆分(首先表很大.表中数据有独立性,能简单分类,需要把表存放在多种介质.)

 ## 使用中间表

  *   数据查询量大
  *   数据统计,数据分析

# 调整mysql参数优化

## Myisam 内存优化

  *    key_buffer_size
  决定MYISAM索引缓存区的大小,直接影响MyISAM表的存取效率,建议1/4内存

  ```SQL
  show variables like  "%key%"
  SHOW GLOBAL STATUS LIKE '%key_read%';
  ```
  *    read_buffer_size(读缓存)

## Innodb 内存优化

  *    innodb_buffer_pool_size
  存储引擎表数据和索引数据的最大缓存区大小

```SQL
SELECT @@innodb_buffer_pool_size;
```

  *     innodb_old_blocks_pct LRU   (决定old sublist 的比例) 
        innodb_old_blocks_time LRU  (数据转移间隔时间)


```SQL
SELECT variables like %old_blocks%;
```

## mysql并发参数

![mysql](/uploads/0924.png)
 
# Mysql应用优化

## 使用连接池

## 减少对Mysql的真实连接
  
  *   避免相同数据的重复执行
  *   使用Mysql缓存
  *   增加系统数据cache

## 负载均衡

  *   LVS分布式
  *  读写分离,主从复制
