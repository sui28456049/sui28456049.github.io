---
title: MySql高级总结
date: 2017-08-29 10:31:01
tags: Mysql
---

## 触发器

> 触发器是一个特殊的存储过程，不同的是存储过程要用CALL来调用，而触发器不需要使用CALL

> 也不需要手工启动，只要当一个预定义的事件发生的时候，就会被MYSQL自动调用。

注意:mysql语法以;为结束符,先修mysql定界符
DELIMITER $
记得最后修改回来 DELIMITER;

### 语法

#### 查看触发器

``
SHOW TRIGGERS
``
#### 删除触发器
```
DROP TRIGGER 
```
#### 创建触发器


```
CREATE TRIGGER <触发器名称>  --触发器必须有名字，最多64个字符，可能后面会附有分隔符.它和MySQL中其他对象的命名方式基本相象.
{ BEFORE | AFTER }  --触发器有执行的时间设置：可以设置为事件发生前或后。
{ INSERT | UPDATE | DELETE }  --同样也能设定触发的事件：它们可以在执行insert、update或delete的过程中触发。
ON <表名称>  --触发器是属于某一个表的:当在这个表上执行插入、 更新或删除操作的时候就导致触发器的激活. 我们不能给同一张表的同一个事件安排两个触发器。
FOR EACH ROW  --触发器的执行间隔：FOR EACH ROW子句通知触发器 每隔一行执行一次动作，而不是对整个表执行一次。
<触发器SQL语句>  --触发器包含所要触发的SQL语句：这里的语句可以是任何合法的语句， 包括复合语句，但是这里的语句受的限制和函数的一样。
```

```
DELIMITER $
CREATE TRIGGER first 
AFTER INSERT 
ON tab1
FOR EACH ROW
BEGIN
insert into tab2(tab2_id) value(new.tab1_id);
END$
DELIMITER;
```
### 示例

#### 新建表
```SQL
CREATE TABLE `t_a` (
  `id` smallint(1) unsigned NOT NULL AUTO_INCREMENT,
  `username` varchar(20) DEFAULT NULL,
  `groupid` mediumint(8) unsigned NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`)
) ENGINE=MyISAM AUTO_INCREMENT=16 DEFAULT CHARSET=latin1;
```

```SQL
CREATE TABLE `t_b` (
  `id` smallint(1) unsigned NOT NULL AUTO_INCREMENT,
  `username` varchar(20) DEFAULT NULL,
  `groupid` mediumint(8) unsigned NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`)
) ENGINE=MyISAM AUTO_INCREMENT=57 DEFAULT CHARSET=latin1;
```
#### insert 操作
```SQL
create trigger t1
after 
insert on t_a 
for each row
begin 
insert into t_b set username = new.username,groupid=new.groupid;
end$
```
#### update 操作
```SQL
# Trigger触发后，当t_a表groupid,username数据有更改时，对t_b表同步一条更新后的数据
   create trigger t2
   after
   update on t_a
   for each row 
   begin
    if new.groupid !=old.groupid or old.username != new.username then
         update `t_b` set groupid=new.groupid,username = new.username where username=old.username and groupid=old.groupid;
    end if;
   end$
```
#### delete 操作
```SQL
 create trigger t3
   after
   delete on t_a
   for each row 
   begin 
    delete from `t_b` where username=old.username and groupid=old.groupid; 
    insert into `t_b` (`username`,`groupid`) values (old.username,'5555');
   end$
```
## 存储过程
> 存储过程和函数是在数据库中定义一些SQL语句的集合，然后直接调用这些存储过程和函数来执行已经定义好的SQL语句。存储过程和函数可以避免开发人员重复的编写相同的SQL语句。而且，存储过程和函数是在MySQL服务器中存储和执行的，可以减少客户端和服务器端的数据传输。

### 基本语法

#### 查看

```
show procedure status; //存储过程 

show function status; //函数 

查看存储过程或函数的创建代码 
show create procedure proc_name; 
show create function func_name; 
```

#### 创建
```SQL
create procedure p1 () 
begin 
    select concat('查找东西.....'); 
end;   
``` 
存储过程用create procedure 创建， 业务逻辑和sql写在begin和end之间。mysql中可用call porcedureName ();来调用过程。
#### 执行

```SQL
-- 调用过程 
call p1;  
```
#### 删除
```
drop procedure if exists p1; 
```

### in/out/inout(不常用)

#### in

```SQL
create procedure p3(in n int)
begin
 declare total int default 0;
 declare num int default 0;
while(num < n) do
	set num=num+1;
	set total = num+total;
end while;
select total;
end;
call p3(100); #5050
```

##### out 

```SQL
 create procedure p5(in pid int, out name VARCHAR(20))
  begin
 select username into name from table where id = pid;
  end
 call p5(1,@sui)
 select @sui;
```

## 游标

### 概念

```SQL
      # declare 声明变量,set赋值变量
	   -- 一条sql,对应N条结果集的资源,取出资源的接口/句柄,就是游标.
	   -- 沿着游标,可以一次取出一行.

	     -- declare 声明;  declare 游标名 cursor for select select_statement
        -- open 打开;     open 游标名
        -- fetch 取值;    fetch 游标名;   into var1,var2[,.....]
        -- close 关闭;    close 游标名
```

### NOT FOUND 事件触发[exit]

```SQL
create procedure p9()
begin
	declare row_gid int;
	declare row_num int;
    declare row_name varchar(20);

	declare you int default 1;
	declare getgoods cursor for select gid,num,name from goods;
	declare exit handler for NOT FOUND set you = 0;

	open getgoods;
	epeat
	    fetch getgoods into row_gid,row_num,row_name;
	    select row_num,row_name;
	until you=0 end repeat;

	close getgoods;
end;$
```

# NOT FOUND 事件触发[continue]

```SQL
create procedure p10()
begin
     declare row_gid int;
     declare row_num int;
     declare row_name varchar(20);
     		
     declare you int default 1;
     declare getgoods cursor for select gid,num,name from goods;
     declare continue handler for NOT FOUND set you := 0;

     open getgoods;
       fetch getgoods into row_gid,row_num,row_name;
       	
     repeat
     	select row_num,row_name;
     	fetch getgoods into row_gid,row_num,row_name;
     until you=0 end repeat;

     close getgoods;

end;$
```

## mysql权限管理

### mysql.user表

```SQL
#创建test用户授权所有数据库和表
 grant all privileges on *.* to test@localhost identified by 'Myd*d123456';
 # 刷新权限表
 flush privileges;
 # 修改密码
 SET PASSWORD FOR 'username'@'host' = PASSWORD('newpassword');
 # 撤销授权
 REVOKE privilege ON databasename.tablename FROM 'username'@'host';
 #查看用户授权
  show grants for test@localhost;
 #删除用户
  DROP USER 'test'@'localhost'
```