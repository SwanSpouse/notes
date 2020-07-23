# 简介

Elastcisearch 是分布式的 文档 存储。它能存储和检索复杂的数据结构–序列化成为JSON文档–以 实时 的方式。 换句话说，一旦一个文档被存储在 Elasticsearch 中，它就是可以被集群中的任意节点检索到。

在 Elasticsearch 中， 每个字段的所有数据 都是 默认被索引的 。 即每个字段都有为了快速检索设置的专用倒排索引。而且，不像其他多数的数据库，它能在 相同的查询中 使用所有这些倒排索引，并以惊人的速度返回结果。

```javascript
{
    "name":         "John Smith",
    "age":          42,
    "confirmed":    true,
    "join_date":    "2014-06-01",
    "home": {
        "lat":      51.5,
        "lon":      0.1
    },
    "accounts": [
        {
            "type": "facebook",
            "id":   "johnsmith"
        },
        {
            "type": "twitter",
            "id":   "johnsmith"
        }
    ]
}
```

## 文档元数据

* \_index : 文档存放位置 
* \_type: 文档表示的对象类型
* \_id：文档唯一标识

## 索引：

逻辑上的命名。实际文档存储在分片中。一个索引包含多个分片。

## 类型：

同一个索引下，不同种类的数据做逻辑区分。

## ID:

ID和索引、类型组合，唯一确定当前的产品。

Relational DB -&gt; Databases -&gt; Tables -&gt; Rows -&gt; Columns

Elasticsearch -&gt; Indices -&gt; Types -&gt; Documents -&gt; Fields

## 创建新文档：

PUT /website/blog/123?op\_type=create

PUT /website/blog/123/\_create

## **搜索文档：**

GET /{index}/{type}/{id}

## 更新：

POST /website/blog/1/\_update

## reference

* [https://www.elastic.co/guide/cn/elasticsearch/guide/current/\_Document\_Metadata.html\#\_id](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_Document_Metadata.html#_id)

