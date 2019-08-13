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
  
  
# 实例

## 8g 内存服务器 mysql.cnf

```bash
[client]
#password	= your_password
port		= 3306
socket		= /tmp/mysql.sock

[mysqld]
port		= 3306
socket		= /tmp/mysql.sock
datadir = /www/server/data
default_storage_engine = InnoDB
performance_schema_max_table_instances = 400
table_definition_cache = 400
skip-external-locking
key_buffer_size = 128M
max_allowed_packet = 100G
table_open_cache = 512
sort_buffer_size = 2M
net_buffer_length = 4K
read_buffer_size = 2M
read_rnd_buffer_size = 256K
myisam_sort_buffer_size = 32M
thread_cache_size = 64
query_cache_size = 64M
tmp_table_size = 64M
sql-mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

explicit_defaults_for_timestamp = true
#skip-name-resolve
max_connections = 500
max_connect_errors = 100
open_files_limit = 65535

log-bin=mysql-bin
binlog_format=mixed
server-id = 1
expire_logs_days = 10
slow_query_log=1
slow-query-log-file=/www/server/data/mysql-slow.log
long_query_time=3
#log_queries_not_using_indexes=on
early-plugin-load = ""


innodb_data_home_dir = /www/server/data
innodb_data_file_path = ibdata1:10M:autoextend
innodb_log_group_home_dir = /www/server/data
innodb_buffer_pool_size = 512M
innodb_log_file_size = 256M
innodb_log_buffer_size = 64M
innodb_flush_log_at_trx_commit = 1
innodb_lock_wait_timeout = 50
innodb_max_dirty_pages_pct = 90
innodb_read_io_threads = 2
innodb_write_io_threads = 2

[mysqldump]
quick
max_allowed_packet = 500M

[mysql]
no-auto-rehash

[myisamchk]
key_buffer_size = 128M
sort_buffer_size = 2M
read_buffer = 2M
write_buffer = 2M

[mysqlhotcopy]
interactive-timeout
```

## 1~2g 服务器推荐配置
![2g](/uploads/20190813_01.png)
## 2~4g  配置
![4g](/uploads/20190813_02.png)
## 4~8g  配置
![4g](/uploads/20190813_03.png)
## 8~16g  配置
![4g](/uploads/20190813_04.png)
## 16~32g  配置
![4g](/uploads/20190813_05.png)





