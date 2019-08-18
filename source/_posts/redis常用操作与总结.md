---
title: redis常用操作与总结
date: 2018-08-18 11:50:24
tags: php
category: php
---
>Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。 它支持多种类型的数据结构，如 字符串（strings）， 散列（hashes）， 列表（lists）， 集合（sets）， 有序集合（sorted sets） 与范围查询， bitmaps， hyperloglogs 和 地理空间（geospatial） 索引半径查询。 Redis 内置了 复制（replication），LUA脚本（Lua scripting）， LRU驱动事件（LRU eviction），事务（transactions） 和不同级别的 磁盘持久化（persistence）， 并通过 Redis哨兵（Sentinel）和自动 分区（Cluster）提供高可用性（high availability）。
这里是我常用redis一些操作与总结,我都用的是`predis`这个类库

`composer require predis/predis` 安装,mac图像界面`another redis desktop manager`

# redis常见数据结构
1. 字符串（strings）
2. 散列（hashes）
3. 列表（lists）
4. 集合（sets）
5. 有序集合（sorted sets）

# 基础操作
## 字符串操作
```php
<?php
require __DIR__.'/vendor/autoload.php';

$client = new Predis\Client();
// 获取数据
$value = $client->get('key');
echo $value . PHP_EOL;

// 修改数据，与创建数据一致，即覆盖数据
$client->set('key', 'value2');
echo $client->get('key') . PHP_EOL;

// 追加数据
$client->append('key', '_value2');
echo $client->get('key') . PHP_EOL;

// 删除数据
$client->del('key');
// $redis->delete('key');
var_dump($client->get('key'));

// 创建数据，带有效期
$client->setex('timeout_key', 5, 'timeout_value');
// 获取数据的有效期
echo $client->ttl('timeout_key') . PHP_EOL;

// 判断是否已经写入，未写入则写入
$client->set('unique_key', 'unique_value');
if (!$client->setnx('unique_key', 'unique_value')) {
    echo $client->get('unique_key') . PHP_EOL;
}

// 批量创建
$multi = ['key1' => 'value1', 'key2' => 'value2', 'key3' => 'value3'];
$client->mset($multi);

// 批量获取
$result = $client->mget(array_keys($multi));
dd($result);

```
## 列表
```php
require __DIR__.'/vendor/autoload.php';


$redis = new \Redis();
$redis->connect('127.0.0.1', 6379);
// Redis 没设置密码则不需要这行代码
// $redis->auth('password');

// 向队列左侧加入元素
$redis->lPush('lists', 'X');
$redis->lPush('lists', 'X');
// 向队列右侧加入元素
$redis->rPush('lists', 'Z');

// 将索引为1的数据修改为 Y
$redis->lSet('lists', 1, 'Y');

// 获取 list 长度
$length = $redis->lLen('lists');
echo $length;

// 遍历 list
$lists = $redis->lRange('lists', 0, $length - 1);
dump($lists);

// 从左侧出队一个元素（获取并删除）
$x = $redis->lPop('lists');
echo $x . PHP_EOL;
// 从右侧出队一个元素（获取并删除）
$z = $redis->rPop('lists');
echo $z . PHP_EOL;

// 获取左侧第一个元素
$y = $redis->lGet('lists', 0);
echo $y . PHP_EOL;

// 删除队列
$redis->del('lists');

```
## 集合
```php
<?php
require __DIR__.'/vendor/autoload.php';


$redis = new \Redis();
$redis->connect('127.0.0.1', 6379);
// Redis 没设置密码则不需要这行代码
// $redis->auth('password');

// 创建集合
$redis->sAdd('sets', 'value1', 'value2');
// 以数组形式创建集合
$redis->sAddArray('sets2',['value3','value4','value1']);

// 取两个集合的并集
$union = $redis->sUnion('sets', 'sets2');
// 取两个集合的差集
$diff = $redis->sDiff('sets', 'sets2');
// 取两个集合的交集
$inter = $redis->sInter('sets', 'sets2');

$length = $redis->sCard('sets');
echo $length.PHP_EOL;

// 获取集合中全部元素
// 不推荐使用这种方法获取全部数据，会导致服务器执行超时
$sets = $redis->sMembers('sets');
var_dump($sets);

// 判断元素是否是集合中的成员
$isMember = $redis->sIsMember('sets', 'value2');
var_dump($isMember);

// 删除集合中的元素
$redis->sRem('sets', 'value2');
var_dump($redis->sMembers('sets'));

// 随机获取一个元素
echo $redis->sRandMember('sets');

// 随机获取一个元素并从集合中删除
echo $redis->sPop('sets');

// 删除集合
$redis->del('sets', 'sets2');


```
## 哈希
```php
$redis = new \Redis();
$redis->connect('127.0.0.1', 6379);
// Redis 没设置密码则不需要这行代码
// $redis->auth('opG5dGo9feYarUifaLb8AdjKcAAXArgZ');

// 创建 hash 表
// 向名字叫 'hash' 的 hash表 中添加元素 ['key1' => 'val1']
$redis->hSet('hash', 'key1', 'val1');

// 获取 hash表 中键名是 key1 的值
echo $redis->hGet('hash', 'key1') . PHP_EOL;

// 获取 hash表的元素个数
echo $redis->hLen('hash') . PHP_EOL;

// 获取 hash表 中所有的键
$keys = $redis->hKeys('hash');
var_dump($keys);

// 获取 hash表 中所有的值
$vals = $redis->hVals('hash');
var_dump($vals);

// 获取 hash表 中所有的键值对
// 不推荐使用这种方法获取全部数据，会导致服务器执行超时
// $all = $redis->hGetAll('hash');
// var_dump($all);

// 判断 hash 表中是否存在键名是 key2 的元素
$bool = $redis->hExists('hash', 'key2');
echo $bool ? '存在' : '不存在' . PHP_EOL;

// 批量添加元素
$redis->hMset('hash', ['key2' => 'val2', 'key3' => 'val3']);

// 批量获取元素
$hashes = $redis->hMGet('hash', ['key1', 'key2', 'key3']);
var_dump($hashes);

// 删除 hash表
$redis->delete('hash');
```
## 有序集合
```php
<?php
require __DIR__.'/vendor/autoload.php';

$redis = new \Redis();
$redis->connect('127.0.0.1', 6379);
// Redis 没设置密码则不需要这行代码
// $redis->auth('opG5dGo9feYarUifaLb8AdjKcAAXArgZ');

// 添加成员
$redis->zAdd('zset', 95, '小明');
$redis->zAdd('zset', 99, '小刚');
$redis->zAdd('zset', 100, '小红');

// 统计成员个数
echo $redis->zCard('zset') . PHP_EOL;

// 获取某个成员的分数
$score = $redis->zScore('zset', '小明');
echo $score . PHP_EOL;

// 获取某个成员的排名
$rank = $redis->zRank('zset', '小明'); // 从低到高排序的名次
$revRank = $redis->zRevRank('zset', '小明'); // 从高到低排序的名次
echo $rank . PHP_EOL;
echo $revRank . PHP_EOL;

// 给指定成员增加分数
$redis->zIncrBy('zset', 1, '小明'); // 给小明加一分

// 返回指定排名范围的成员
$range = $redis->zRange('zset', 0, 9, true); // 返回分数从低到高排序的前10名及分数
$revRange = $redis-> zRevRange('zset', 0, 9, true); // 返回分数从高到低排序的前10名及分数
var_dump($range);
var_dump($revRange);

// 删除成员
$redis->zRem('zet', '小明');

// 返回指定分数范围的成员
$rangeByScore = $redis->zRangeByScore('zet', 98, 100); // 返回指定分数范围内从低到高排序的成员
$revRangeByScore = $redis->zRevRangeByScore('zet', 98, 100); // 返回指定分数范围内从高到低排序的成员
var_dump($rangeByScore);
var_dump($revRangeByScore);

```



# redis 存储session
# 计数器
# 限时访问
# 排行榜
# tag标签
# 分布式锁
# 抽奖
