title: Elasticsearch 学习笔记 1
date: 2015-09-23 16:24:24
tags: Elasticsearch
categories: Elasticsearch
---

Elasticsearch 是一个开源的搜索引擎，基于 Lucene（被认为是迄今为止最先进、性能最好的、功能最全的搜索引擎库）。

#### 安装

唯一依赖 Java，如果没有则先安装 Java。

Elasticsearch 安装包可以从[这里](http://www.elasticsearch.org/download/)下载。

#### 运行

将压缩包解压后并可以直接运行
```
./bin/elasticsearch -d
```
加上参数 `-d` 以守护进程的方式运行。

测试安装是否成功
```
curl 'http://localhost:9200/?pretty'
```
如果看到一下回复信息则表明安装成功。
```
{
  "status" : 200,
  "name" : "Paul Bailey",
  "cluster_name" : "elasticsearch",
  "version" : {
    "number" : "1.7.2",
    "build_hash" : "e43676b1385b8125d647f593f7202acbd816e8ec",
    "build_timestamp" : "2015-09-14T09:49:53Z",
    "build_snapshot" : false,
    "lucene_version" : "4.10.4"
  },
  "tagline" : "You Know, for Search"
}
```

#### RESTful API

Elasticsearch 为 Java 用户提供了两种内置客户端：节点客户端(node client)和传输客户端(Transport client)。而其他语言则可以通过 RESTful API 进行交互。

默认提供 RESTful API 的端口是 9200，以 JSON 数据进行交互。

可以直接用`curl`发送请求，例如：
```
curl -XPUT "http://120.25.84.82:9200/megacorp/employee/2" -d'
{
    "first_name" :  "Jane",
    "last_name" :   "Smith",
    "age" :         32,
    "about" :       "I like to collect rock albums",
    "interests":  [ "music" ]
}'
```

或者使用`marvel`这个插件，下面的示例都使用的这个插件。
> 安装插件 `./bin/plugin -i elasticsearch/marvel/latest`
> 访问 `http://localhost:9200/_plugin/marvel/sense/index.html`


##### 创建

```
PUT /megacorp/employee/2
{
    "first_name" :  "Jane",
    "last_name" :   "Smith",
    "age" :         32,
    "about" :       "I like to collect rock albums",
    "interests":  [ "music" ]
}
```

##### 查询单个

```
GET /megacorp/employee/2
```

返回的结果中，原始 JSON 文档在 `_source` 字段中

```
{
   "_index": "megacorp",
   "_type": "employee",
   "_id": "2",
   "_version": 1,
   "found": true,
   "_source": {
      "first_name": "Jane",
      "last_name": "Smith",
      "age": 32,
      "about": "I like to collect rock albums",
      "interests": [
         "music"
      ]
   }
}
```

##### 简单搜索

```
GET /megacorp/employee/_search
```

返回结果，默认返回前10个结果。

```
{
   "took": 2,
   "timed_out": false,
   "_shards": {
      "total": 5,
      "successful": 5,
      "failed": 0
   },
   "hits": {
      "total": 3,
      "max_score": 1,
      "hits": [
         {
            "_index": "megacorp",
            "_type": "employee",
            "_id": "1",
            "_score": 1,
            "_source": {
               "first_name": "John",
               "last_name": "Smith",
               "age": 25,
               "about": "I love to go rock climbing",
               "interests": [
                  "sports",
                  "music"
               ]
            }
         },
         {
            "_index": "megacorp",
            "_type": "employee",
            "_id": "2",
            "_score": 1,
            "_source": {
               "first_name": "Jane",
               "last_name": "Smith",
               "age": 32,
               "about": "I like to collect rock albums",
               "interests": [
                  "music"
               ]
            }
         },

         ...

      ]
   }
}
```

##### 简单搜索带条件

```
GET /megacorp/employee/_search?q=first_name:John
```

**DSL查询(Query DSL)**

DSL(Domain Specific Language 特定领域语言)以 JSON 请求体的形式出现。

```
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "first_name" : "John"
        }
    }
}
```

还支持很多复杂的查询，以后在具体介绍。

##### 全文搜索

这个功能一般的数据库是没有提供的。例如：搜索所有喜欢“rock climbing”的员工。

```
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "about" : "rock climbing"
        }
    }
}
```

返回的结果，默认是按`相似匹配度`来排序的，`_score`是这条记录的`相关性评分`。

```
{
   "took": 3,
   "timed_out": false,
   "_shards": {
      "total": 5,
      "successful": 5,
      "failed": 0
   },
   "hits": {
      "total": 2,
      "max_score": 0.16273327,
      "hits": [
         {
            "_index": "megacorp",
            "_type": "employee",
            "_id": "1",
            "_score": 0.16273327,
            "_source": {
               "first_name": "John",
               "last_name": "Smith",
               "age": 25,
               "about": "I love to go rock climbing",
               "interests": [
                  "sports",
                  "music"
               ]
            }
         },
         {
            "_index": "megacorp",
            "_type": "employee",
            "_id": "2",
            "_score": 0.016878016,
            "_source": {
               "first_name": "Jane",
               "last_name": "Smith",
               "age": 32,
               "about": "I like to collect rock albums",
               "interests": [
                  "music"
               ]
            }
         }
      ]
   }
}
```

##### 短语搜索

匹配若干个关键字，使用`match_phrase`。

```
GET /megacorp/employee/_search
{
    "query" : {
        "match_phrase" : {
            "about" : "rock climbing"
        }
    }
}
```


RESTful API 提供了很多很强大的接口，可以方便的进行搜索、聚合等操作，这里只是简单介绍了下，具体详细的请看相关文档。
