# ElasticSearch

分布式搜索引擎

doc：<https://www.elastic.co/guide/cn/elasticsearch/guide/current/foreword_id.html>

# 基础入门

## 名词定义

以mysql为例子形象说明。

- 集群 ( **Cluster**)：数据库服务器地址，比如mysql数据库
- 索引 ( **index** )：服务器中的数据库，一个mysql中可以有很多数据库，比如图书馆数据库
- 类型( **type** )：数据库中的表，一个数据库多个表
- 文档( **document**)：表中的数据，一个表有很多条数据。每一条数据是一个文档
- 属性( **Field** )：表中的字段，一个表有很多字段。每一个字段就是一个属性

## 交互

RESTful API with JSON over HTTP

一个 Elasticsearch 请求和任何 HTTP 请求一样由若干相同的部件组成：

```js
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
```



被 `< >` 标记的部件：

| `VERB`         | 适当的 HTTP *方法* 或 *谓词* : `GET`、 `POST`、 `PUT`、 `HEAD` 或者 `DELETE`。 |
| -------------- | ------------------------------------------------------------ |
| `PROTOCOL`     | `http` 或者 `https`（如果你在 Elasticsearch 前面有一个 `https` 代理） |
| `HOST`         | Elasticsearch 集群中任意节点的主机名，或者用 `localhost` 代表本地机器上的节点。 |
| `PORT`         | 运行 Elasticsearch HTTP 服务的端口号，默认是 `9200` 。       |
| `PATH`         | API 的终端路径（例如 `_count` 将返回集群中文档数量）。Path 可能包含多个组件，例如：`_cluster/stats` 和 `_nodes/stats/jvm` 。 |
| `QUERY_STRING` | 任意可选的查询字符串参数 (例如 `?pretty` 将格式化地输出 JSON 返回值，使其更容易阅读) |
| `BODY`         | 一个 JSON 格式的请求体 (如果请求需要的话)                    |

例如，计算集群中文档的数量，我们可以用这个:

```json
curl -XGET 'http://localhost:9200/_count?pretty' -d '
{
    "query": {
        "match_all": {}
    }
}
'
```



Elasticsearch 返回一个 HTTP 状态码（例如：`200 OK`）和（除`HEAD`请求）一个 JSON 格式的返回值。前面的 `curl` 请求将返回一个像下面一样的 JSON 体：

```json
{
    "count" : 0,
    "_shards" : {
        "total" : 5,
        "successful" : 5,
        "failed" : 0
    }
}
```

## 基础语法

重点是查询，可以使用多种条件去查询

### 增

```json
//给服务器发送put请求
//url  依次是/索引/类型/文档id
PUT /megacorp/employee/1
//body
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
```



### 删

```json
DELETE /megacorp/employee/1
//结果如下：
{
    "found": true,
    "_index": "megacorp",
    "_type": "employee",
    "_id": "1",
    "_version": 2,
    "result": "deleted",	//结果是已删除
    "_shards": {
        "total": 2,	//目前该类型中还剩下2个文档
        "successful": 1,
        "failed": 0
    }
}
```

### 改

POST ..

### 查

精确查：

```json
//查询
GET /megacorp/employee/1
//结果如下：
{
  "_index" :   "megacorp",
  "_type" :    "employee",
  "_id" :      "1",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "first_name" :  "John",
      "last_name" :   "Smith",
      "age" :         25,
      "about" :       "I love to go rock climbing",
      "interests":  [ "sports", "music" ]
  }
}
```

查全部：

```json
GET /megacorp/employee/_search
```



模糊查：

普通get形式

```JSON
GET /megacorp/employee/_search?q=last_name:Smith
```

表达式形式：

*领域特定语言* （DSL）， 使用 JSON 构造了一个请求。

```json
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            "last_name" : "Smith"
        }
    }
}
```

更复杂的例子

```json
GET /megacorp/employee/_search
//这部分与我们之前使用的 match 查询 一样。
//这部分是一个 range 过滤器 ， 它能找到年龄大于 30 的文档，其中 gt 表示_大于_(great than)。
{
    "query" : {
        "bool": {
            "must": {
                "match" : {
                    "last_name" : "smith" 
                }
            },
            "filter": {
                "range" : {
                    "age" : { "gt" : 30 } 
                }
            }
        }
    }
}
```



## 和传统数据库明显差别

**Elasticsearch中的 *相关性* 概念非常重要，也是完全区别于传统关系型数据库的一个概念，数据库中的一条记录要么匹配要么不匹配。**

```json
GET /megacorp/employee/_search
{
    "query" : {
        "match" : {
            //查询文档中about属性的员工信息
            "about" : "rock climbing"
        }
    }
}
```

//正常数据库是返回like '%rock climbing%'  的数据

//但是match会进行关键字模糊匹配，并计算出相关性强弱。

//如果想要和传统数据一样的精确匹配则应该使用match_phrase而不是match



```json
{
   ...
   "hits": {
      "total":      2,
      "max_score":  0.16273327,
      "hits": [
         {
            ...
            "_score":         0.16273327, 
            "_source": {
               "first_name":  "John",
               "last_name":   "Smith",
               "age":         25,
               "about":       "I love to go rock climbing",
               "interests": [ "sports", "music" ]
            }
         },
         {
            ...
            "_score":         0.016878016, 
            "_source": {
               "first_name":  "Jane",
               "last_name":   "Smith",
               "age":         32,
               "about":       "I like to collect rock albums",
               "interests": [ "music" ]
            }
         }
      ]
   }
}
```



## 集群

集群是由一个或者多个拥有相同 `cluster.name` 配置的节点组成， 它们共同承担数据和负载的压力。当有节点加入集群中或者从集群中移除节点时，集群将会重新平均分布所有的数据。

一个运行中的 Elasticsearch 实例称为一个节点。作为用户，我们可以将请求发送到 *集群中的任何节点* ，包括主节点。 **每个节点都知道任意文档所处的位置，并且能够将我们的请求直接转发到存储我们所需文档的节点。** 无论我们将请求发送到哪个节点，它都能负责从各个包含我们所需文档的节点收集回数据，并将最终结果返回給客户端。 Elasticsearch 对这一切的管理都是透明的。

### 集群健康

GET /_cluster/health

```curl
GET /_cluster/health
//完整的请求是 curl -XGET 'localhost:9200/_cluster/health'
//如果是在windows环境，直接浏览器请求： “服务器ip地址:9200/_cluster/health ”
```

最关心的指标是status

```text
green
所有的主分片和副本分片都正常运行。
yellow
所有的主分片都正常运行，但不是所有的副本分片都正常运行。
red
有主分片没能正常运行。
```

