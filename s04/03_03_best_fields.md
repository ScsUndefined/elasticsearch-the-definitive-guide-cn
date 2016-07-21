# 4.3.3 Best Fields 最优字段查询

现在我们假定我们有个博客网站，存了下面两篇博文，现在要提供一个搜索功能允许用户来检索博文：

```bash
PUT /my_index/my_type/1
{
    "title": "Quick brown rabbits",
    "body":  "Brown rabbits are commonly seen."
}

PUT /my_index/my_type/2
{
    "title": "Keeping pets healthy",
    "body":  "My quick brown fox eats rabbits on a regular basis."
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/15_Best_fields.json)

然后我们假设用户输入了“Brown fox”，这时候我们没法提前预知用户输入的文本会被分词成什么样，也不能预先知道分出来的每个词会命中标题还是命中正文，我们能知道的是用户是想搜出相关的文档。这时候我们目测一下，明显应该是第二个博文更符合用户的查询条件，因为第二篇博文的正文里同时含有 brown 和 fox 这两个单词。

现在运行下下面这个bool查询：

```bash
{
    "query": {
        "bool": {
            "should": [
                { "match": { "title": "Brown fox" }},
                { "match": { "body":  "Brown fox" }}
            ]
        }
    }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/15_Best_fields.json) 

然后你会发现第一篇文档的相关度评分居然高于第二篇，这明显有违用户需求：

```bash
{
  "hits": [
     {
        "_id":      "1",
        "_score":   0.14809652,
        "_source": {
           "title": "Quick brown rabbits",
           "body":  "Brown rabbits are commonly seen."
        }
     },
     {
        "_id":      "2",
        "_score":   0.09256032,
        "_source": {
           "title": "Keeping pets healthy",
           "body":  "My quick brown fox eats rabbits on a regular basis."
        }
     }
  ]
}
```

如果你想要知道为什么查询结果会不合理，那你就得明白这个 bool 查询计算相关度评分时采取的计算方式了：

  1. 执行 should 子句下的两个 match 查询
  2. 把两个 match 查询的得分相加
  3. 把总分乘以 match 子句的总数
  4. 把计算结果处理整个查询子句的个数（即2）

第一篇博文的标题和正文里都含有单词 brown，所以两个match语句就会与其匹配，并获得一个分数。而第二篇博文只在正文里有brown 和 fox 这两个词，而在标题中却都没有，它在步骤2中产出的评分就会由正文 match 查询产生的一个相对较高的得分加上标题 match 查询产生的一个零分相加而成。然后步骤三会将总分乘以 1 而步骤四则会把结果再除以 2。结果就导致了最终的分值反而比第一篇博文来得低。

在本例中，标题和正文字段其实存在着竞争关系。我们想要在二个字段之间选出一个最优的字段来进行相关度评分。

所以，不如我们不要去结合每个字段的匹配程度来计算出最终的相关度评分，而是直接采用最优的字段的相关度作为整个查询的匹配程度的衡量标准，而忽视掉次优字段的匹配结果。这中查询策略将会导致同时含有 brown 和 fox 这两个词的字段战胜仅包含 brown 或者 fox 的字段。

## dis_max Query

这个时候我们可以使用 dis_max 查询来替代 bool 查询，这种查询方式全称是 Disjunction Max 查询。Disjunction 表示或的关系（而 conjunction 表示与），所以 Disjunction Max 查询的作用就是，只要某个文档匹配 dix_max 查询中任意一条子查询，那么这个文档就会被认为是匹配整个 dix_max 查询的，就会被返回。而最终的相关度评分则会以匹配度最高的子查询的得分为准来返回：

```bash
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Brown fox" }},
                { "match": { "body":  "Brown fox" }}
            ]
        }
    }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/15_Best_fields.json) 

运行上面这段代码就会得到我们想要的结果了。

```bash
{
  "hits": [
     {
        "_id":      "2",
        "_score":   0.21509302,
        "_source": {
           "title": "Keeping pets healthy",
           "body":  "My quick brown fox eats rabbits on a regular basis."
        }
     },
     {
        "_id":      "1",
        "_score":   0.12713557,
        "_source": {
           "title": "Quick brown rabbits",
           "body":  "Brown rabbits are commonly seen."
        }
     }
  ]
}
```

***


Imagine that we have a website that allows users to search blog posts, such as these two documents:

```bash
PUT /my_index/my_type/1
{
    "title": "Quick brown rabbits",
    "body":  "Brown rabbits are commonly seen."
}

PUT /my_index/my_type/2
{
    "title": "Keeping pets healthy",
    "body":  "My quick brown fox eats rabbits on a regular basis."
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/15_Best_fields.json)

The user types in the words “Brown fox” and clicks Search. We don’t know ahead of time if the user’s search terms will be found in the title or the body field of the post, but it is likely that the user is searching for related words. To our eyes, document 2 appears to be the better match, as it contains both words that we are looking for.

Now we run the following bool query:

```bash
{
    "query": {
        "bool": {
            "should": [
                { "match": { "title": "Brown fox" }},
                { "match": { "body":  "Brown fox" }}
            ]
        }
    }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/15_Best_fields.json) 

And we find that this query gives document 1 the higher score:

```bash
{
  "hits": [
     {
        "_id":      "1",
        "_score":   0.14809652,
        "_source": {
           "title": "Quick brown rabbits",
           "body":  "Brown rabbits are commonly seen."
        }
     },
     {
        "_id":      "2",
        "_score":   0.09256032,
        "_source": {
           "title": "Keeping pets healthy",
           "body":  "My quick brown fox eats rabbits on a regular basis."
        }
     }
  ]
}
```

To understand why, think about how the bool query calculates its score:

  1. It runs both of the queries in the should clause.
  2. It adds their scores together.
  3. It multiplies the total by the number of matching clauses.
  4. It divides the result by the total number of clauses (two).

Document 1 contains the word brown in both fields, so both match clauses are successful and have a score. Document 2 contains both brown and fox in the body field but neither word in the title field. The high score from the body query is added to the zero score from the title query, and multiplied by one-half, resulting in a lower overall score than for document 1.

In this example, the title and body fields are competing with each other. We want to find the single best-matching field.

What if, instead of combining the scores from each field, we used the score from the best-matching field as the overall score for the query? This would give preference to a single field that contains both of the words we are looking for, rather than the same word repeated in different fields.

## dis_max Query
Instead of the bool query, we can use the dis_max or Disjunction Max Query. Disjunction means or (while conjunction means and) so the Disjunction Max Query simply means return documents that match any of these queries, and return the score of the best matching query:

```bash
{
    "query": {
        "dis_max": {
            "queries": [
                { "match": { "title": "Brown fox" }},
                { "match": { "body":  "Brown fox" }}
            ]
        }
    }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/15_Best_fields.json) 

This produces the results that we want:

```bash
{
  "hits": [
     {
        "_id":      "2",
        "_score":   0.21509302,
        "_source": {
           "title": "Keeping pets healthy",
           "body":  "My quick brown fox eats rabbits on a regular basis."
        }
     },
     {
        "_id":      "1",
        "_score":   0.12713557,
        "_source": {
           "title": "Quick brown rabbits",
           "body":  "Brown rabbits are commonly seen."
        }
     }
  ]
}
```