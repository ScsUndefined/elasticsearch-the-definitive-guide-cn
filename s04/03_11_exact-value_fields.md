# 当涉及到 Exact-Value 型字段时

在最后我们将会谈及 exact-value not_analyzed 字段。遗憾的是，之前介绍的跨字段查询工具在面对，需要对 not_analyed 字段和 analyzed 字段进行混合查询，的需求时都将纷纷折戟。

要解释为什么的话，用 validate-query 来解释最容易了。假如现在我们在搜索 title ，first_name 和 last_name 者三个字段，并且只有 title 字段是 not_analyzed 的：

```bash
GET /_validate/query?explain
{
    "query": {
        "multi_match": {
            "query":       "peter smith",
            "type":        "cross_fields",
            "fields":      [ "title", "first_name", "last_name" ]
        }
    }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/55_Not_analyzed.json)

由于 title 字段是 not_analyzed 的，所以它会拿整个查询关键词去进行精确匹配！

```
title:peter smith
(
    blended("peter", fields: [first_name, last_name])
    blended("smith", fields: [first_name, last_name])
)
```

明显 title 字段中不可能存有整个搜索关键词，所以返回的搜索结果集永远都是空的。因此绝对要避免对 not_analyzed 字段进行 multi_match 查询。

~~这看似不合理，有点像 multi_math 的一个不完美之处，但其实也挺合理，not_analyzed 型字段本来就是被设计成用来查询精确的数据的，如果要对这些数据进行全文检索的话本来在入索引的时候就不该把它索引成 not_analyzed 型，能想到的解决思路只有用 fields 方式在映射的时候映射成主从字段，从字段的解析器可以使用 `keyword` 解析器，不过这样也会造成 multi_match 查询时涉及的多个字段的解析器不一致。~~

***

# Exact-Value Fields
The final topic that we should touch on before leaving multifield queries is that of exact-value not_analyzed fields. It is not useful to mix not_analyzed fields with analyzed fields in multi_match queries.

The reason for this can be demonstrated easily by looking at a query explanation. Imagine that we have set the title field to be not_analyzed:

```bash
GET /_validate/query?explain
{
    "query": {
        "multi_match": {
            "query":       "peter smith",
            "type":        "cross_fields",
            "fields":      [ "title", "first_name", "last_name" ]
        }
    }
}
```

VIEW IN SENSE 

Because the title field is not analyzed, it searches that field for a single term consisting of the whole query string!

```
title:peter smith
(
    blended("peter", fields: [first_name, last_name])
    blended("smith", fields: [first_name, last_name])
)
```

That term clearly does not exist in the inverted index of the title field, and can never be found. Avoid using not_analyzed fields in multi_match queries.