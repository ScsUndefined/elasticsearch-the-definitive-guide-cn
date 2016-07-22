# 跨字段型实体的搜索

现在我们来讨论一个更常见的情形：对跨字段型的实体进行搜索。像人，产品或者地址等实体，它们的标识信息会被分开存储到多个字段中。比如我们有一个表示人的数据，会被这样切分成两个字段来入索引：

```bash
{
    "firstname":  "Peter",
    "lastname":   "Smith"
}
```

或者一个这样子的地址：

```bash
{
    "street":   "5 Poland Street",
    "city":     "London",
    "country":  "United Kingdom",
    "postcode": "W1V 3DG"
}
```

这看上去有点像我们在 Multiple Query Strings 章节中讨论的，但其实两者之间区别很大。在 Multiple Query Strings 中，我们对每个查询的字段都指定了各自的搜索关键词，而在此我们需要使用相同的关键字来搜索每个字段。

我们的用户可能搜索一个叫 “Peter Smith” 的人或者一个叫 “Poland Street W1V.” 的地址。这些关键字对应的数据都被拆分存储在不同的字段里，所以采用 dis_max 或者 best_fields 查询来应对这一场景明显都不合适。~~我怎么觉得 best_fields 其实貌似也行。。。~~

# 一个拍着屁股想出来的办法

Really, we want to query each field in turn and add up the scores of every field that matches, which sounds like a job for the bool query:

既然我们想要在多个字段里搜索同一个关键词，我们直接想到的做法或许是像下面这样的，在一个bool查询里嵌套多个 should 子查询，每个should子查询都这个关键词对不同字段的 match 查询：

```bash
{
  "query": {
    "bool": {
      "should": [
        { "match": { "street":    "Poland Street W1V" }},
        { "match": { "city":      "Poland Street W1V" }},
        { "match": { "country":   "Poland Street W1V" }},
        { "match": { "postcode":  "Poland Street W1V" }}
      ]
    }
  }
}
```

当然这么编码看上去有点蠢，你可以用 most_fields 类型的 multi_match 来简写上面这个 bool 查询：~~所以我说 most_fields 好像也行嘛~~

```bash
{
  "query": {
    "multi_match": {
      "query":       "Poland Street W1V",
      "type":        "most_fields",
      "fields":      [ "street", "city", "country", "postcode" ]
    }
  }
}
```

# 使用 most_fields 的弊端

~~瞬间被打脸~~

使用 most_fields 方式来应对这一场景有着一些难以察觉的潜在风险点：

  * most_fields 查询方式是将查询关键词分词，然后把从关键词中细分出来的各个词元去对每个字段进行查询，然后如果某个文档匹配中的字段越多则该文档相关度越高，而不会去判断关键词的词元命中率
   
  * It is designed to find the most fields matching any words, rather than to find the most matching words across all fields.
  
  * It can’t use the operator or minimum_should_match parameters to reduce the long tail of less-relevant results.
 
  * Term frequencies are different in each field and could interfere with each other to produce badly ordered results.

***

# Cross-fields Entity Search

Now we come to a common pattern: cross-fields entity search. With entities like person, product, or address, the identifying information is spread across several fields. We may have a person indexed as follows:

```bash
{
    "firstname":  "Peter",
    "lastname":   "Smith"
}
```

Or an address like this:

```bash
{
    "street":   "5 Poland Street",
    "city":     "London",
    "country":  "United Kingdom",
    "postcode": "W1V 3DG"
}
```

This sounds a lot like the example we described in Multiple Query Strings, but there is a big difference between these two scenarios. In Multiple Query Strings, we used a separate query string for each field. In this scenario, we want to search across multiple fields with a single query string.

Our user might search for the person “Peter Smith” or for the address “Poland Street W1V.” Each of those words appears in a different field, so using a dis_max / best_fields query to find the single best-matching field is clearly the wrong approach.

# A Naive Approach

Really, we want to query each field in turn and add up the scores of every field that matches, which sounds like a job for the bool query:

```bash
{
  "query": {
    "bool": {
      "should": [
        { "match": { "street":    "Poland Street W1V" }},
        { "match": { "city":      "Poland Street W1V" }},
        { "match": { "country":   "Poland Street W1V" }},
        { "match": { "postcode":  "Poland Street W1V" }}
      ]
    }
  }
}
```

Repeating the query string for every field soon becomes tedious. We can use the multi_match query instead, and set the type to most_fields to tell it to combine the scores of all matching fields:

```bash
{
  "query": {
    "multi_match": {
      "query":       "Poland Street W1V",
      "type":        "most_fields",
      "fields":      [ "street", "city", "country", "postcode" ]
    }
  }
}
```

# Problems with the most_fields Approach

The most_fields approach to entity search has some problems that are not immediately obvious:

  * It is designed to find the most fields matching any words, rather than to find the most matching words across all fields.
  
  * It can’t use the operator or minimum_should_match parameters to reduce the long tail of less-relevant results.
 
  * Term frequencies are different in each field and could interfere with each other to produce badly ordered results.