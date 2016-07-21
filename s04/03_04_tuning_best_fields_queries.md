# 4.3.4 Tuning Best Fields Queries 对最优字段查询进行微调

假如用户输入的是“quick pets”那么结果为如何呢？第一篇博文的标题和第二篇博文的正文都有单词 quick，另外第二篇博文的标题含有单词“pet”，而这两篇博文都没有哪个字段是同时含有 quick 和 pet 这两个词的。

一个简单的 dis_max 查询，就像下面展示的那样，将会从多个查询子句中选择一个最佳的字段，然后忽略掉其他字段的匹配结果：

```bash
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Quick pets" }},
                { "match": { "body":  "Quick pets" }}
            ]
        }
    }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/15_Best_fields.json) 

结果：

```bash
{
  "hits": [
     {
        "_id": "1",
        "_score": 0.12713557, ①
        "_source": {
           "title": "Quick brown rabbits",
           "body": "Brown rabbits are commonly seen."
        }
     },
     {
        "_id": "2",
        "_score": 0.12713557, ②
        "_source": {
           "title": "Keeping pets healthy",
           "body": "My quick brown fox eats rabbits on a regular basis."
        }
     }
   ]
}
```

①,② 注意，两个文档的相关度评分是一样的

有时候我们会想要让既命中标题又命中正文的文档的优先级高于那些只在一个字段中有匹配的文档。这时候上面那个 dis_max 查询就不符合我们的需求了，因为它只是将最优的那个字段的相关度评分作为整个查询最终的评分。

## tie_breaker

在进行 dis_max 查询的时候，默认情况下最终的相关度评分只会以最优字段的得分为准，但如果你还想要把其他次优字段的得分也考虑进来，那你就可以使用 tie_breaker 参数：

```bash
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Quick pets" }},
                { "match": { "body":  "Quick pets" }}
            ],
            "tie_breaker": 0.3
        }
    }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/15_Best_fields.json) 

上面这个查询的结果：

```bash
{
  "hits": [
     {
        "_id": "2",
        "_score": 0.14757764, ①
        "_source": {
           "title": "Keeping pets healthy",
           "body": "My quick brown fox eats rabbits on a regular basis."
        }
     },
     {
        "_id": "1",
        "_score": 0.124275915, ②
        "_source": {
           "title": "Quick brown rabbits",
           "body": "Brown rabbits are commonly seen."
        }
     }
   ]
}
``` 

①,② 第二篇博文的得分比第一篇高出了一丢丢

tie_breaker 参数会改变 dis_max 查询的算分逻辑，使其往 bool 查询的算分逻辑靠拢。它会把算分逻辑变成这样：

  1. 拿到最优字段的得分
  2. 把其他子查询的得分乘以 tie_breaker 的值
  3. 求和然后进行 normalize 操作

使用 tie_breaker 的话，所有的子查询语句的匹配程度都会对最终结果的匹配度造成影响，而最优字段的权重更高。

> **注意**
> 
> tie_breaker参数可以是0~1之间的浮点数（包括0和1）。具体值应该设置成多少，就根据你的数据和你的查询条件自己来设定好了。推荐的做法是尽可能地取小一点的数，比如（0.1-0.4），因为值越大就越违背之所以要使用 dis_max 查询的初衷。

***

What would happen if the user had searched instead for “quick pets”? Both documents contain the word quick, but only document 2 contains the word pets. Neither document contains both words in the same field.

A simple dis_max query like the following would choose the single best matching field, and ignore the other:

```bash
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Quick pets" }},
                { "match": { "body":  "Quick pets" }}
            ]
        }
    }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/15_Best_fields.json) 

```bash
{
  "hits": [
     {
        "_id": "1",
        "_score": 0.12713557, ①
        "_source": {
           "title": "Quick brown rabbits",
           "body": "Brown rabbits are commonly seen."
        }
     },
     {
        "_id": "2",
        "_score": 0.12713557, ②
        "_source": {
           "title": "Keeping pets healthy",
           "body": "My quick brown fox eats rabbits on a regular basis."
        }
     }
   ]
}
```

①,② Note that the scores are exactly the same.

We would probably expect documents that match on both the title field and the body field to rank higher than documents that match on just one field, but this isn’t the case. Remember: the dis_max query simply uses the _score from the single best-matching clause.

## tie_breaker

It is possible, however, to also take the _score from the other matching clauses into account, by specifying the tie_breaker parameter:

```bash
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Quick pets" }},
                { "match": { "body":  "Quick pets" }}
            ],
            "tie_breaker": 0.3
        }
    }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/15_Best_fields.json) 

This gives us the following results:

```bash
{
  "hits": [
     {
        "_id": "2",
        "_score": 0.14757764, ①
        "_source": {
           "title": "Keeping pets healthy",
           "body": "My quick brown fox eats rabbits on a regular basis."
        }
     },
     {
        "_id": "1",
        "_score": 0.124275915, ②
        "_source": {
           "title": "Quick brown rabbits",
           "body": "Brown rabbits are commonly seen."
        }
     }
   ]
}
``` 

①,② Document 2 now has a small lead over document 1.

The tie_breaker parameter makes the dis_max query behave more like a halfway house between dis_max and bool. It changes the score calculation as follows:

  1. Take the _score of the best-matching clause.
  2. Multiply the score of each of the other matching clauses by the tie_breaker.
  3. Add them all together and normalize.

With the tie_breaker, all matching clauses count, but the best-matching clause counts most.

> **Note**
> 
> The tie_breaker can be a floating-point value between 0 and 1, where 0 uses just the best-matching clause and 1 counts all matching clauses equally. The exact value can be tuned based on your data and queries, but a reasonable value should be close to zero, (for example, 0.1 - 0.4), in order not to overwhelm the best-matching nature of dis_max.