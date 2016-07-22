# cross-fields 查询

定制自己的 \_all 字段的确是一种不错的解决方案，但其实还不够灵活，因为这要求你在索引你的数据的时候就事前预估到你需要一个什么样的 \_all 字段。~~但现实往往是编码速度跟不上需求的迭代速度，心中一万个草泥马奔腾而过~~也因此Elasticsearch 还提供一种可以搜索阶段采用的解决方案：cross_fields 型的 multi_match 查询。cross_fields 类型使得整个查询采取以词元为中心的查询策略，这有别于 best_fields 和 most_fields 类型所采取的以字段为中心的查询策略，它会将所有字段当作一个整个的大的字段来看待，然后再在每个字段中搜索每个词元。

如果非要举个栗子来解释以字段为中心的查询策略与以词元为中心的策略之间的差异的话，那就看一下这边这个以字段为中心的 most_fields 查询的 validate-explain 结果：

```bash
GET /_validate/query?explain
{
    "query": {
        "multi_match": {
            "query":       "peter smith",
            "type":        "most_fields",
            "operator":    "and", ①
            "fields":      [ "first_name", "last_name" ]
        }
    }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/50_Cross_field.json) 

① 必须要匹配所有的词元

然后运行一下上面这段代码，就会得到下面这个结果，这表示如果一个文档要匹配的话，那么 perter 和 Smith 这两个词都必须出现在同一个字段中，first_name 或者 last_name 字段都行：

```bash
(+first_name:peter +first_name:smith)
(+last_name:peter  +last_name:smith)
```

而一个以词元为中心的查询策略其实应该使用的是下面这种查询逻辑：

```bash
+(first_name:peter last_name:peter)
+(first_name:smith last_name:smith)
```

换句话说，Perter 必须存在于一个字段中，而 Smith 则必须存在于另一个字段中。

cross_fields 类型的搜索首先在将查询关键词进行解析，得到一系列的词元，然后它会在每个字段中搜索每一个词元。这一个不同点就能够解决我们在 [Field-Centric Queries](https://www.elastic.co/guide/en/elasticsearch/guide/current/field-centric.html) 章节中列举出来的三个问题中的两个，接下来我们就只剩一个问题需要解决了，那就是要消除 IDF 的差异对最终结果的排序造成的误导。

幸运的是，cross_fields 查询类型也已经帮我们解决了这个问题，不行的话我们 validate-query 一下：

```bash
GET /_validate/query?explain
{
    "query": {
        "multi_match": {
            "query":       "peter smith",
            "type":        "cross_fields", ①
            "operator":    "and",
            "fields":      [ "first_name", "last_name" ]
        }
    }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/50_Cross_field.json) 

① 使用以词元为中心的搜索方式 cross_fields

它解决了跨字段的 IDF 的差异性问题：

```bash
+blended("peter", fields: [first_name, last_name])
+blended("smith", fields: [first_name, last_name])
```

换句话说，它会在 first_name 和 last_name 字段中分别计算出 smith 这个词的 IDF，然后取最小值作为 smith 相对于这两个字段的 IDF。最终，因为 Smith 是一个常见的姓氏，所以它的 IDF 会很低，然后导致这个 IDF 同时被应用在 first_name 中，导致 smith 这个词在 first_name 字段中 IDF 也同样很低。

> **注意**
> 
> 如果要让 cross_fields 查询获得最佳的执行效果，那么最好所有被查询的字段都用相同解析器。因为使用了相同的解析器的字段会被分在同一组，作为一个混合的字段集。
>
> 如果你查询的部分字段使用了不同的解析器的话，这些字段就会被以 best_fields 中所采用到的相同的方式来添加到查询之中。举个栗子，如果我们往上面那个查询之中添加一个叫做 title 的字段（假设这个 title 采用了不同的解析器），那么最终 validate-query 的结果就会像下面这样：
> 
> ```bash
(+title:peter +title:smith)
(
  +blended("peter", fields: [first_name, last_name])
  +blended("smith", fields: [first_name, last_name])
)
```
>
> 当你使用 minimum_should_match 和 operator 时，理清这一点就更显得尤为重要了

# Per-Field Boosting
One of the advantages of using the cross_fields query over [custom _all fields](https://www.elastic.co/guide/en/elasticsearch/guide/current/custom-all.html) is that you can boost individual fields at query time.

For fields of equal value like first_name and last_name, this generally isn’t required, but if you were searching for books using the title and description fields, you might want to give more weight to the title field. This can be done as described before with the caret (^) syntax:

```bash
GET /books/_search
{
    "query": {
        "multi_match": {
            "query":       "peter smith",
            "type":        "cross_fields",
            "fields":      [ "title^2", "description" ] ①
        }
    }
}
```

① The title field has a boost of 2, while the description field has the default boost of 1.

The advantage of being able to boost individual fields should be weighed against the cost of querying multiple fields instead of querying a single custom _all field. Use whichever of the two solutions that delivers the most bang for your buck.

***

# cross-fields Queries

The custom _all approach is a good solution, as long as you thought about setting it up before you indexed your documents. However, Elasticsearch also provides a search-time solution to the problem: the multi_match query with type cross_fields. The cross_fields type takes a term-centric approach, quite different from the field-centric approach taken by best_fields and most_fields. It treats all of the fields as one big field, and looks for each term in any field.

To illustrate the difference between field-centric and term-centric queries, look at the explanation for this field-centric most_fields query:

```bash
GET /_validate/query?explain
{
    "query": {
        "multi_match": {
            "query":       "peter smith",
            "type":        "most_fields",
            "operator":    "and", ①
            "fields":      [ "first_name", "last_name" ]
        }
    }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/50_Cross_field.json) 

① All terms are required.

For a document to match, both peter and smith must appear in the same field, either the first_name field or the last_name field:

```bash
(+first_name:peter +first_name:smith)
(+last_name:peter  +last_name:smith)
```

A term-centric approach would use this logic instead:

```bash
+(first_name:peter last_name:peter)
+(first_name:smith last_name:smith)
```

In other words, the term peter must appear in either field, and the term smith must appear in either field.

The cross_fields type first analyzes the query string to produce a list of terms, and then it searches for each term in any field. That difference alone solves two of the three problems that we listed in [Field-Centric Queries](https://www.elastic.co/guide/en/elasticsearch/guide/current/field-centric.html), leaving us just with the issue of differing inverse document frequencies.

Fortunately, the cross_fields type solves this too, as can be seen from this validate-query request:

```bash
GET /_validate/query?explain
{
    "query": {
        "multi_match": {
            "query":       "peter smith",
            "type":        "cross_fields", ①
            "operator":    "and",
            "fields":      [ "first_name", "last_name" ]
        }
    }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/50_Cross_field.json) 

① Use cross_fields term-centric matching.

It solves the term-frequency problem by blending inverse document frequencies across fields:

```bash
+blended("peter", fields: [first_name, last_name])
+blended("smith", fields: [first_name, last_name])
```

In other words, it looks up the IDF of smith in both the first_name and the last_name fields and uses the minimum of the two as the IDF for both fields. The fact that smith is a common last name means that it will be treated as a common first name too.

> **Note**
> 
> For the cross_fields query type to work optimally, all fields should have the same analyzer. Fields that share an analyzer are grouped together as blended fields.
> 
> If you include fields with a different analysis chain, they will be added to the query in the same way as for best_fields. For instance, if we added the title field to the preceding query (assuming it uses a different analyzer), the explanation would be as follows:

> ```bash
(+title:peter +title:smith)
(
  +blended("peter", fields: [first_name, last_name])
  +blended("smith", fields: [first_name, last_name])
)
```

> This is particularly important when using the minimum_should_match and operator parameters.

# Per-Field Boosting
One of the advantages of using the cross_fields query over [custom _all fields](https://www.elastic.co/guide/en/elasticsearch/guide/current/custom-all.html) is that you can boost individual fields at query time.

For fields of equal value like first_name and last_name, this generally isn’t required, but if you were searching for books using the title and description fields, you might want to give more weight to the title field. This can be done as described before with the caret (^) syntax:

```bash
GET /books/_search
{
    "query": {
        "multi_match": {
            "query":       "peter smith",
            "type":        "cross_fields",
            "fields":      [ "title^2", "description" ] ①
        }
    }
}
```

① The title field has a boost of 2, while the description field has the default boost of 1.

The advantage of being able to boost individual fields should be weighed against the cost of querying multiple fields instead of querying a single custom _all field. Use whichever of the two solutions that delivers the most bang for your buck.