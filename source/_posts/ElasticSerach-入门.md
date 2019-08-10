---
title: ElasticSerach 入门
date: 2019-07-25 17:19:01
tags: Linux
category: Linux
---
# intro
ElasticSerach（下面简称ES）是一个基于Lucene（一个开源全文搜索引擎）构建的开源、分布式可扩展的实时搜索和分析引擎。它还可以作为一个分布式实时文件存储系统。
ES是目前全文搜索引擎的首选，它可以快速地储存、搜索和分析海量数据。维基百科、Stack Overflow、Github 等系统都采用它。
ES采用java语言开发，对外提供了 RESTful API 的操作接口，开箱即用。

和关系型数据库的对比:

mysql 	    Elasticsearch	       释义
Databases	   Indices	     索引(名词)即数据库
Tables	        Types	         类型即表名
Rows	      Documents	       文档即每行数据
Columns	        Fields	           字段

# 安装
下载安装文档地地址:

https://www.elastic.co/cn/downloads/elasticsearch 

我通过 yum 安装  ps:/etc/elasticsearch/elasticsearch.yml编辑配置

也可以 docker安装

```
docker pull elasticsearch:7.2.0

docker run -d --name elasticsearch  -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.2.0
```

这里有个巨坑:
/etc/elasticsearch/elasticsearch.yml编辑配置

http.host: 0.0.0.0
原本以为只需要更改network.host，但是更改后无效。尝试使用默认的network.host，增加http.host: 0.0.0.0，可以成功。
`network.host//在集群中节点的网络地址，其它节点通过这个地址与其进行通信`,线上要设成具体的 IP

# 基本操作

使用RESTful API来举例使用ES。一个完整的REST请求包括如下三部分：

1、操作类型，如GET,PUT,POST,DELETE等
2、URL部分，如http://localhost:9200/blogs/blog/1
3、请求正文内容。对于GET请求，没有单独的请求正文内容，请求信息都包含在URL中。对于PUT,POST等操作，可以带正文内容，正文内容采用josn的格式。
ES的REST请求的返回内容的格式绝大部分都是JOSN格式的。也就是说请求和响应的内容都是JSON格式，这样有较好的一致性。

## 插入数据
```json
PUT http://120.27.3.185:9200/database/article/1
{
    "title": "first article",
    "content": "first article content",
    "author": "first"
}
```

返回的数据:

```json
{
    "_index": "database",
    "_type": "article",
    "_id": "1",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 0,
    "_primary_term": 1
}
```

## 查询数据
```json
GET http://120.27.3.185:9200/database/article/1
```

返回数据:

``` json
{
    "_index": "database",
    "_type": "article",
    "_id": "1",
    "_version": 1,
    "_seq_no": 2,
    "_primary_term": 1,
    "found": true,
    "_source": {
        "title": "first article",
        "content": "first article content",
        "author": "first"
    }
}
```
## 修改数据
```json
PUT http://120.27.3.185:9200/database/article/1
{
    "title": "22222",
    "content": "2",
    "author": "2222"
}
```

返回结果:

```json
{
    "_index": "database",
    "_type": "article",
    "_id": "1",
    "_version": 2,
    "result": "updated",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 3,
    "_primary_term": 1
}
```
可以看出，返回的josn内容中，相比插入操作，有这几个字段的内容发生了变化：

```json
"_version" : 2,
"result" : "updated",
```

可以看到，记录的 Id 没变，但是版本（version）从1变成2
操作类型（result）从created变成updated
created字段变成false( 我的 7.0 版本没有这个字段)，因为这次不是新建记录，而是修改操作。



ES文档的版本号是有用的，我们在修改数据时，可以传入一个版本号。这样当在一个客户端读取和更新文档的间隔中，有另外客户端更新了数据。
如果这个客户端的更新操作传入了版本号（前面读取的版本号），这时因为传入的版本号和当期实际的版本号不一致，就会操作失败。这样就防止可能的数据冲突。
这也是ES采用乐观并发控制（OCC）机制的体现。

加入版本号进行更新，只需在url后面加上version参数

```
PUT http://120.27.3.185:9200/database/article/1?version=2
```

## 删除操作

```
DELETE  http://120.27.3.185:9200/database/article/1
```

# 搜索操作(*)

先插入 2 条数据

```json
PUT  http://120.27.3.185:9200/database/article/1
{
	"title":"随某人",
	"content":"上海内容...."
}

PUT  http://120.27.3.185:9200/database/article/2
{
	"title":"随某人",
	"content":"杭州内容...."
}
```

下面我们来执行根据给定的字符串进行查找，只要数据中有包含给定字符串的内容就认为匹配上:

```json
POST  http://120.27.3.185:9200/_search
{
    "query": {
        "query_string": {
            "query": " 随"
        }
    }
}
```

返回结果:

```json
{
    "took": 170,
    "timed_out": false,
    "_shards": {
        "total": 2,
        "successful": 2,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 2,
            "relation": "eq"
        },
        "max_score": 0.816944,
        "hits": [
            {
                "_index": "database",
                "_type": "article",
                "_id": "1",
                "_score": 0.816944,
                "_source": {
                    "title": "随某人",
                    "content": "上海内容...."
                }
            },
            {
                "_index": "database",
                "_type": "article",
                "_id": "2",
                "_score": 0.816944,
                "_source": {
                    "title": "随某人",
                    "content": "杭州内容...."
                }
            }
        ]
    }
}
```

## 索指定json字段中的内容
```
POST  http://120.27.3.185:9200/_search
{
    "query": {
        "query_string": {
            "query": "随",
            "fields": ["title"]
        }
    }
}
```

 字段 title 中含有随,如果是`content` 则搜索不出.
字段通过查询条件中的field参数指定，可以看出，同时可指定多个字段。
 对比关系数据库，ES的搜索操作有点类似关系数据库中带like子句的select语句。
 
# 术语
##  索引（Index）
ES数据管理的顶层单位就叫做 Index（索引）。
它不是关系数据库中索引的概念，是单个数据库的同义词，即可以把它等价于关系数据库中的单个数据库。
Index的名称必须小写。

既然Index是ES管理数据的顶层单位，肯定先需要有索引，才能进行后续的数据存储等操作，类似关系数据库中得首先有数据库一样。
可前面的操作我们并没有看到创建Index的操作。是不是ES启动后有一个默认的Index呢？
其实情况是这样的，上面的REST请求 http://120.27.3.185:9200/database/article/1中的 database就是一个索引名，当我们执行数据插入操作时，如果url中指定的索引不存在，则ES会自动创建该索引。
就如我们前面的例子，我们插入数据时，索引database并不存在，所以ES按照默认的设置自动创建了database索引。

我们可以单独创建一个索引

```
PUT  http://120.27.3.185:9200/topics
```

上面操作通过http的PUT操作，创建了一个名称为topics的索引。
操作成功后的响应信息如下：

```
{
    "acknowledged": true,
    "shards_acknowledged": true,
    "index": "topics"
}
```

我们可以查看到ES中的所有索引

```
GET  http://120.27.3.185:9200/_cat/indices?v
```

响应 如下:

```
health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   blogs    cK1UOKHHQpyCstKMKqfeOQ   1   1          1            0        5kb            5kb
yellow open   topics   uSVSYQhjR6eCtLFnufMOjA   1   1          0            0       230b           230b
yellow open   database CqRkYbjBRyWKcbAOxt3SaA   1   1          2            0       15kb           15kb

```

通过DELETE操作，我们可以删除指定的索引，如下面操作：

```
DELETE  http://localhost:9200/topics
```

##  类型（Type）
在索引中，可以定义一个或多个类型（type），类型是索引的逻辑分区，是同一索引中有公共字段的文档的集合。
快被移除了,不用过多精力学习.

##  文档（Document）
##  文档 (Document)
Document是Index中的单条记录，它就像关系数据库中表中的记录（行）


##  字段（Field）
字段就是Docuemnt对应的json内容的字段。一个Docuemnt可以包括零个或多个字段，字段可以是一个简单的值（如字符串、整数、日期），也可以是一个数组或对象的嵌套结构。
每个字段都对应一个数据类型，如整数、字符串、数组、对象等。
##  映射（mapping）

mapping是类似于数据库中的表结构定义，主要作用如下：

* 定义index下的字段名
* 定义字段类型，比如数值型、浮点型、布尔型等
* 定义倒排索引相关的设置，比如是否索引、记录position等。
* 设置分词器等

与表结构不同的是，关系数据库的表的结构必须事先通过create table语句来明确。而映射既可以显示的通过命令来事先定义，也可以在存储文档（插入数据）时由ES来自动识别。如我们上面的例子，并没有显示的去定义Index的mapping信息。
可以通过 GET http://localhost:9200/_mapping 来查看Index的mapping信息。

##  主键（ID）
ES中的每个Document都有一个唯一的ID（在type下唯一），也就是说 index/type/id必须是唯一的。
我们可以插入数据时显示的指定ID，如前面的操作：
```
PUT  http://localhost:9200/database/article/1
```

也可以也ES系统自动生成，注意这时就不能用PUT操作，要用`POST`操作

```json
POST  http://localhost:9200/database/article
{
    "title": "elasticsearch title",
    "content": "elasticsearch content"
}
```
# 关系数据库对比
<table>
<thead>
<tr>
<th>ElasticSerach</th>
<th>Mysql</th>
</tr>
</thead>
<tbody>
<tr>
<td>Index</td>
<td>Database</td>
</tr>
<tr>
<td>Type</td>
<td>Table</td>
</tr>
<tr>
<td>Document</td>
<td>Row</td>
</tr>
<tr>
<td>Field</td>
<td>Column</td>
</tr>
<tr>
<td>Mapping</td>
<td>Schema</td>
</tr>
<tr>
<td>Everything is indexed</td>
<td>Index(表的索引)</td>
</tr>
<tr>
<td>ID</td>
<td>Primary Key</td>
</tr>
<tr>
<td>Query DSL</td>
<td>SQL</td>
</tr>
<tr>
<td>PUT/POST  http://....</td>
<td>insert into ....</td>
</tr>
<tr>
<td>GET http://....</td>
<td>select * from ...</td>
</tr>
<tr>
<td>POST http://... (搜索操作)</td>
<td>selcct * from... like ...</td>
</tr>
<tr>
<td>PUT  http://....</td>
<td>update .....</td>
</tr>
<tr>
<td>DELETE http://....</td>
<td>delete from...</td>
</tr>
</tbody>
</table>

#  插件安装

##  监控
https://github.com/mobz/elasticsearch-head#running-with-built-in-server

```
# for Elasticsearch 5.x, 6.x, and 7.x: site plugins are not supported. Run as a standalone server
git clone git://github.com/mobz/elasticsearch-head.git
cd elasticsearch-head
npm install
npm run start
open http://localhost:9100/
```

## 中文分词

 安装分词插件:
 
```
/usr/share/elasticsearch/bin/elasticsearch-plugin  install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.2.0/elasticsearch-analysis-ik-7.2.0.zip
```

```
POST http://120.27.3.185:9200/_analyze
{  
	"analyzer": "ik_max_word",
    "text": "春天是一个好季节"  
}
```

 返回结果:
 
 ```json
 {
    "tokens": [
        {
            "token": "春天",
            "start_offset": 0,
            "end_offset": 2,
            "type": "CN_WORD",
            "position": 0
        },
        {
            "token": "是",
            "start_offset": 2,
            "end_offset": 3,
            "type": "CN_CHAR",
            "position": 1
        },
        {
            "token": "一个",
            "start_offset": 3,
            "end_offset": 5,
            "type": "CN_WORD",
            "position": 2
        },
        {
            "token": "一",
            "start_offset": 3,
            "end_offset": 4,
            "type": "TYPE_CNUM",
            "position": 3
        },
        {
            "token": "个",
            "start_offset": 4,
            "end_offset": 5,
            "type": "COUNT",
            "position": 4
        },
        {
            "token": "好",
            "start_offset": 5,
            "end_offset": 6,
            "type": "CN_CHAR",
            "position": 5
        },
        {
            "token": "季节",
            "start_offset": 6,
            "end_offset": 8,
            "type": "CN_WORD",
            "position": 6
        }
    ]
}
 ```
 
  参考地址: [参考文章](https://www.jianshu.com/p/bb89ad7a7f7d)