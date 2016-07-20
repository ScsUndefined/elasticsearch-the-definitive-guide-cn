# 4.3.1 Multiple Query Strings 多个查询关键词

The simplest multifield query to deal with is the one where we can *map search terms to specific fields*. If we know that *War and Peace* is the title, and *Leo Tolstoy* is the author, it is easy to write each of these conditions as a `match` clause and to combine them with a `bool` query:

最简单的跨字段查询的场景是，我们知道查询关键词中的哪个部分对应哪个字段。比如我们知道“战争与和平”是书名而“列夫 托尔斯泰”是作者名，这时候我们只要在一个`bool`查询中嵌两个`should`查询就可以了：

```bash
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title":  "War and Peace" }},
        { "match": { "author": "Leo Tolstoy"   }}
      ]
    }
  }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/05_Multiple_query_strings.json)

`bool`查询采取*匹配得越多则越好*的策略来计算相关度。所以整个查询的相关度评分是由各个子查询的评分累加起来的。即，如果一个文档同时满足多个查询子句，则它的相关度就比只满足单个查询子句的查询结果来得高。

当然，你在`bool`查询中除了使用`match`子查询外，也可以嵌其他类型的查询语句，包括`bool`类型。比如我们可以再嵌一个子查询来指定我们偏爱的译者翻译的版本：

```bash
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title":  "War and Peace" }},
        { "match": { "author": "Leo Tolstoy"   }},
        { "bool":  {
          "should": [
            { "match": { "translator": "Constance Garnett" }},
            { "match": { "translator": "Louise Maude"      }}
          ]
        }}
      ]
    }
  }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/05_Multiple_query_strings.json)

然后我们再仔细想想，为什么在这里会把译者相关的查询条件组合成一个`bool`查询？四个查询子句其实都是`match`类型，所以，为啥咱们就不能直接把这四个`match`写在同一层级呢？

要解释这么做的原因，就要回顾下刚才讲到的相关度评分的计算规则。整个`bool`查询会执行每个`match`查询，然后把各个`match`查询的相关度评分累加起来，并乘以命中的查询子句的数量，最后除以总的查询子句的个数。同一级的每个查询子句都拥有相同的权重。在上面那个查询中，表示译者查询条件的`bool`子查询占整个`bool`查询的三分之一的权重。而如果我们把所有的`match`下在同一层级，那么标题和作者的权重就会从三分之一被稀释成四分之一。

## Prioritizing Clauses 控制子查询的权重

仔细想想，在上面的场景中，其实译者信息和标题以及作者一样享有三分之一的权重也不太合理，因为我们更关注作者和书名，而非译者。所以我们需要给标题查询子句和作者查询子句增加权重。

最简单的办法就是用`boost`参数来增加权重。比如在这里我们如果要增加标题和作者的权重，就把`boost`的值设置为比`1`大的数：

```bash
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { ①
            "title":  {
              "query": "War and Peace",
              "boost": 2
        }}},
        { "match": { ②
            "author":  {
              "query": "Leo Tolstoy",
              "boost": 2
        }}},
        { "bool":  { ③
            "should": [
              { "match": { "translator": "Constance Garnett" }},
              { "match": { "translator": "Louise Maude"      }}
            ]
        }}
      ]
    }
  }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/05_Multiple_query_strings.json)
 

①,② 标题和作者的 `match` 查询子句的 `boost`值为2

③ 而译者的 `bool` 查询子句的 `boost` 值为 1

`boost`参数的最佳值很容易确定，只要多测试几遍就可以了。一个比较推荐的值范围是 1 到 10 或者 1 到 15 也行。再高的话可能就没那么有效果了。因为相关度评分在计算的时候被会 normalized。

***

The simplest multifield query to deal with is the one where we can *map search terms to specific fields*. If we know that *War and Peace* is the title, and *Leo Tolstoy* is the author, it is easy to write each of these conditions as a `match` clause and to combine them with a `bool` query:

```bash
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title":  "War and Peace" }},
        { "match": { "author": "Leo Tolstoy"   }}
      ]
    }
  }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/05_Multiple_query_strings.json)

The `bool` query takes a *more-matches-is-better* approach, so the score from `each` match clause will be added together to provide the final `_score` for each document. Documents that match both clauses will score higher than documents that match just one clause.

Of course, you’re not restricted to using just `match` clauses: the `bool` query can wrap any other query type, including other `bool` queries. We could add a clause to specify that we prefer to see versions of the book that have been translated by specific translators:

```bash
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { "title":  "War and Peace" }},
        { "match": { "author": "Leo Tolstoy"   }},
        { "bool":  {
          "should": [
            { "match": { "translator": "Constance Garnett" }},
            { "match": { "translator": "Louise Maude"      }}
          ]
        }}
      ]
    }
  }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/05_Multiple_query_strings.json)

Why did we put the translator clauses inside a separate `bool` query? All four `match` queries are `should` clauses, so why didn’t we just put the translator clauses at the same level as the title and author clauses?

The answer lies in how the score is calculated. The `bool` query runs each `match` query, adds their scores together, then multiplies by the number of matching clauses, and divides by the total number of clauses. Each clause at the same level has the same weight. In the preceding query, the `bool` query containing the translator clauses counts for one-third of the total score. If we had put the translator clauses at the same level as title and author, they would have reduced the contribution of the title and author clauses to one-quarter each.

## Prioritizing Clauses
It is likely that an even one-third split between clauses is not what we need for the preceding query. Probably we’re more interested in the title and author clauses than we are in the translator clauses. We need to tune the query to make the title and author clauses relatively more important.

The simplest weapon in our tuning arsenal is the `boost` parameter. To increase the weight of the `title `and `author` fields, give them a `boost` value higher than `1`:

```bash
GET /_search
{
  "query": {
    "bool": {
      "should": [
        { "match": { ①
            "title":  {
              "query": "War and Peace",
              "boost": 2
        }}},
        { "match": { ②
            "author":  {
              "query": "Leo Tolstoy",
              "boost": 2
        }}},
        { "bool":  { ③
            "should": [
              { "match": { "translator": "Constance Garnett" }},
              { "match": { "translator": "Louise Maude"      }}
            ]
        }}
      ]
    }
  }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/05_Multiple_query_strings.json)
 

①,② The `title` and `author` clauses have a `boost` value of `2`.

③ The nested `bool` clause has the default `boost` of `1`.

The “best” value for the `boost` parameter is most easily determined by trial and error: set a `boost` value, run test queries, repeat. A reasonable range for `boost` lies between 1 and 10, maybe 15. Boosts higher than that have little more impact because scores are [normalized](https://www.elastic.co/guide/en/elasticsearch/guide/current/_boosting_query_clauses.html#boost-normalization).