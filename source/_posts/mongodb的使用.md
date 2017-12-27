---
title: mongodb的使用
date: 2017-12-27 18:38:48
tags: mongodb
category: mongodb
---

# 安装mongodb服务端与客户端

```bash
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel62-3.6.1.tgz

mv mongodb-linux-x86_64-rhel62-3.6.1 /usr/local/mongodb

MongoDB 的可执行文件位于 bin 目录下，所以可以将其添加到 PATH 路径中：

export PATH=<mongodb-install-directory>/bin:$PATH

<mongodb-install-directory> 为你 MongoDB 的安装路径 我的 /usr/local/mongodb 。

export PATH=/usr/local/mongodb/bin:$PATH
```

## 启动服务端(mongod)

创建数据库目录
MongoDB的数据存储在data目录的db目录下，但是这个目录在安装过程不会自动创建，所以你需要手动创建data目录，并在data目录中创建db目录。

```bash
mkdir -p /data/db     #  /data/db 是 MongoDB 默认的启动的数据库路径(--dbpath)。
```

```bash
./mongod --dbpath=/data/db   # /data/db 可以自定义目录 ,如 /sui/test
```

## 启动MongoDB后台管理 Shell

```bash
$ cd /usr/local/mongodb/bin
$ ./mongo
```

# 安装php扩展

```
pecl install mongodb
```

添加扩展至php.ini ,pecl不行使用源码编译 `phpize`,我图个简单,就用pecl;


# php对mongodb基本操作

## 插入数据

```php
        $bulk = new \MongoDB\Driver\BulkWrite;
        $document = ['_id' => new \MongoDB\BSON\ObjectID, 'name' => '随某人'];

        $_id= $bulk->insert($document);

        var_dump($_id);

        $manager = new \MongoDB\Driver\Manager("mongodb://localhost:27017");
        $writeConcern = new \MongoDB\Driver\WriteConcern(\MongoDB\Driver\WriteConcern::MAJORITY, 1000);
        $result = $manager->executeBulkWrite('test.lucky', $bulk, $writeConcern);
```

## 读取数据 

```php
$manager = new \MongoDB\Driver\Manager("mongodb://localhost:27017");  

// 插入数据
$bulk = new \MongoDB\Driver\BulkWrite;
$bulk->insert(['x' => 1, 'name'=>'sui666', 'url' => 'http://www.sui666.com']);
$bulk->insert(['x' => 2, 'name'=>'sui777', 'url' => 'http://www.sui777.com']);
$bulk->insert(['x' => 3, 'name'=>'sui888', 'url' => 'http://www.sui888.com']);
$manager->executeBulkWrite('test.sites', $bulk);

$filter = ['x' => ['$gt' => 1]];  # 查询大于1的文档
$options = [
    'projection' => ['_id' => 0],
    'sort' => ['x' => -1],
];

// 查询数据
$query = new \MongoDB\Driver\Query($filter, $options);
$cursor = $manager->executeQuery('test.sites', $query);

foreach ($cursor as $document) {
    print_r($document);
}
```

## 更新数据

```php
        $bulk = new \MongoDB\Driver\BulkWrite;

        $bulk->update(
            ['x' => 2],
            ['$set' => ['name' => '会当凌绝顶,一览众山小', 'url' => 'www.google.com']],
            ['multi' => false, 'upsert' => false]
        );

        $manager = new \MongoDB\Driver\Manager("mongodb://localhost:27017");
        $writeConcern = new \MongoDB\Driver\WriteConcern(\MongoDB\Driver\WriteConcern::MAJORITY, 1000);
        $result = $manager->executeBulkWrite('test.sites', $bulk, $writeConcern);
```

## 删除数据

```php
$bulk = new \MongoDB\Driver\BulkWrite;
$bulk->delete(['x' => 1], ['limit' => 1]);   // limit 为 1 时，删除第一条匹配数据
$bulk->delete(['x' => 2], ['limit' => 0]);   // limit 为 0 时，删除所有匹配数据

$manager = new \MongoDB\Driver\Manager("mongodb://localhost:27017");  
$writeConcern = new \MongoDB\Driver\WriteConcern(\MongoDB\Driver\WriteConcern::MAJORITY, 1000);
$result = $manager->executeBulkWrite('test.sites', $bulk, $writeConcern);
```


