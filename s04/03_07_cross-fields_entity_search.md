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

这看上去有点像我们在 Multiple Query Strings 章节中讨论的，但其实两者之间区别很大。

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