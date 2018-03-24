---
title: mysql技巧(不定时更新)
date: 2018-03-24 09:06:44
tags: mysql
category: mysql
---

# 比较两个表的不同数据

## 第一步

首先，使用`UNION`语句来组合两个表中的行; 仅包含需要比较的列。返回的结果集用于比较。

```sql
SELECT t1.pk, t1.c1
FROM t1
UNION ALL
SELECT t2.pk, t2.c1
FROM t2
```
比较两个表，以查找MySQL中不匹配的记录。
## 第二步


根据需要比较的主键和列分组记录。如果需要比较的列中的值相同，则COUNT(*)返回2，否则COUNT(*)返回1。


```sql
SELECT pk, c1
FROM
 (
   SELECT t1.pk, t1.c1
   FROM t1
   UNION ALL
   SELECT t2.pk, t2.c1
   FROM t2
)  t
GROUP BY pk, c1
HAVING COUNT(*) = 1
ORDER BY pk
```

比较两个表，以查找MySQL中不匹配的记录。


# 处理重复数据

一般步骤:

	1. 确定哪一列包含的值可能会重复。
	2. 在列选择列表使用COUNT(*)列出的那些列。
	3. 在GROUP BY子句中列出的列。
	4. HAVING子句设置重复数大于1。

## 在一列中找到重复的数据

```sql
SELECT 
    col, 
    COUNT(col)
FROM
    table_name
GROUP BY col
HAVING COUNT(col) > 1;
```

如果表中出现多个值，则该值将被视为重复。在这个语句中，使用COUNT函数的GROUP BY子句来计算指定列(col)的值。HAVING子句中的条件仅包含值count大于1的行，这些行是重复的行。

## 在多个列中查找重复值

```sql
SELECT 
    col1, COUNT(col1),
    col2, COUNT(col2),
    ...

FROM
    table_name
GROUP BY 
    col1, 
    col2, ...
HAVING 
       (COUNT(col1) > 1) AND 
       (COUNT(col2) > 1) AND 
       ...
```

只有当列的组合重复时，行才被认为是重复的，所以在HAVING子句中使用了AND运算符。

# 删除MySQL中的重复行

## 使用`DELETE JOIN`语句删除重复的行

//删除重复的行并保持最高的ID

```sql
DELETE t1 FROM contacts t1
        INNER JOIN
    contacts t2 
WHERE
    t1.id < t2.id AND t1.email = t2.email;
```


//删除重复的行并保留最低的ID

```sql
DELETE t1 FROM contacts t1
        INNER JOIN
    contacts t2 
WHERE
    t1.id > t2.id AND t1.email = t2.email;
```

## 使用直接表删除重复的行

大致步骤一般为:

* 创建一个新表，其结构与要删除重复行的原始表相同。
* 将原始表中的不同行插入直接表。
* 删除原始表并将直接表重命名为原始表。

### 步骤1 -

```sql
CREATE TABLE source_copy FROM source;
```

### 步骤2 -

```sql
INSERT INTO source_copy
SELECT * FROM source
GROUP BY col; -- column that has duplicate values
```

### 步骤3 -

```sql
DROP TABLE source;
ALTER TABLE source_copy RENAME TO source;
```


# MySQL复制表

```sql
CREATE TABLE new_table 
SELECT col1, col2, col3 
FROM
    existing_table
WHERE
    conditions;
```

注意，上面的声明只是复制表及其数据。它不会复制与表关联的其他数据库对象，如索引，主键约束，外键约束，触发器等。



要从表中复制数据以及表的所有依赖对象

```sql
CREATE TABLE IF NOT EXISTS new_table LIKE existing_table;

INSERT new_table SELECT * FROM existing_table;
```





# mysql 常用函数(个人用到的)


## IF(expr1,expr2,expr3) 

if(sex=1,"男","女")  类似三元运算符

## CONCAT(拼接字符串)

## CONCAT_WS(将多个字符串连接成一个字符串，但是可以一次性指定分隔符)

select CONCAT_WS('-',firstname,lastname) as name;

## GROUP_CONCAT(将group by产生的同一个分组中的值连接起来，返回一个字符串结果。)

语法: group_concat( [distinct] 要连接的字段 [order by 排序字段 asc/desc  ] [separator '分隔符'] )

说明：通过使用distinct可以排除重复值；如果希望对结果中的值进行排序，可以使用order by子句；separator是一个字符串值，缺省为一个逗号。


