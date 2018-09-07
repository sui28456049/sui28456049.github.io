---
title: MySQL大数据量分页查询方法及其优化
date: 2018-09-07 14:19:09
tags: mysql
category: mysql
---

# 直接使用数据库提供的SQL语句

SELECT * FROM 表名称 LIMIT M,N


适应场景: 适用于数据量较少的情况(元组百/千级)

原因/缺点: 全表扫描,速度会很慢 且 有的数据库结果集返回不稳定 limit限制的是从结果集的M位置处取出N条输出,其余抛弃.

# 建立主键或唯一索引, 利用索引

SELECT * FROM your_table WHERE pk>=1000 ORDER BY pk_id ASC LIMIT 0,20

适应场景: 适用于数据量多的情况(元组数上万). 最好ORDER BY后的列对象是主键或唯一所以,使得ORDERBY操作能利用索引被消除但结果集是稳定的

原因: 索引扫描,速度会很快..

# 利用"子查询/连接+索引"快速定位元组的位置,然后再读取元组

SELECT * FROM product WHERE id >= (select id from product limit 866613, 1) limit 20


// 连接查询


SELECT * FROM product a JOIN (select id from product limit 866613, 20) b ON a.ID = b.id


# 添加复合索引

添加一个复合索引 search(vtype,id)

select id from collect where vtype=1 limit 90000,10; 非常快！0.04秒完成！

再测试: 

select id ,title from collect where vtype=1 limit 90000,10; 非常遗憾，8-9秒，没走search索引！


再测试：修改索引为search(id,vtype)，还是select id 这个语句，也非常遗憾，0.5秒。


# 番外(为什么复合左右顺序有影响????)

`复合索引有左前缀匹配原则`

最左前缀匹配原则，非常重要的原则，

mysql会一直向右匹配直到遇到范围查询(>、<、between、like)就停止匹配，

比如a = 1 and b = 2 and c > 3 and d = 4 如果建立(a,b,c,d)顺序的索引，d是用不到索引的，如果建立(a,b,d,c)的索引则都可以用到，a,b,d的顺序可以任意调整。
=和in可以乱序，比如a = 1 and b = 2 and c = 3 建立(a,b,c)索引可以任意顺序，mysql的查询优化器会帮你优化成索引可以识别的形式 




主题思路就是先按索引排序查询，外面再查具体业务字段

