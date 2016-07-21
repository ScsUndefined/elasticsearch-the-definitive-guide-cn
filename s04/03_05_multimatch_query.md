# 4.3.5 multi_match Query

要进行跨字段查询时除了使用 dis_max 查询，还可以使用更方便，更简洁的 multi_match 查询

> **注意**
> 
> multi_match查询有很多类型可选，其中三种就对应了之前我们介绍过的三种应用场景：best_fields 以最优字段为准；most_fields 匹配的字段越多越优，cross_fields 跨字段查询

默认的类型是 best_fields，也即意味着，在默认情况下 multi_match 查询会为每个字段创建一个 match 查询，然后把这些 match 查询内嵌到一个 dis_max 查询中。类似这样一个 dis_max 查询：

```bash
{
  "dis_max": {
    "queries":  [
      {
        "match": {
          "title": {
            "query": "Quick brown fox",
            "minimum_should_match": "30%"
          }
        }
      },
      {
        "match": {
          "body": {
            "query": "Quick brown fox",
            "minimum_should_match": "30%"
          }
        }
      },
    ],
    "tie_breaker": 0.3
  }
}
```

就可以重写成下面这样的一个 multi_match 查询，是不是感觉写起来更简洁，更方便？

```bash
{
    "multi_match": {
        "query":                "Quick brown fox",
        "type":                 "best_fields", ①
        "fields":               [ "title", "body" ],
        "tie_breaker":          0.3,
        "minimum_should_match": "30%" ②
    }
}
```
[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/25_Best_fields.json) 


① best_fields 类型是默认的类型，所以这行其实可以省略

② 类似 minimum_should_match 或者 operator 这样的参数会被传递给每一个被生成的 match 子查询中

## Using Wildcards in Field Names 使用通配符来指定要查询的字段

Field names can be specified with wildcards: any field that matches the wildcard pattern will be included in the search. You could match on the book_title, chapter_title, and section_title fields, with the following:

字段名也可以使用通配符来设置，任何符合通配符规则的字段，都会被包含进整个搜索逻辑中。比如，如果你要搜索三个字段，名字分别为“book_title”，“chapter_title”以及“section_titile”。那你就可以使用下面这样的通配符：
```bash
{
    "multi_match": {
        "query":  "Quick brown fox",
        "fields": "*_title"
    }
}
```

## Boosting Individual Fields 有针对性地加权

你可以在字段名后使用`^`符号来有针对性地进行加权，`^`之后跟一个浮点数，就像下面这样：

```bash
{
    "multi_match": {
        "query":  "Quick brown fox",
        "fields": [ "*_title", "chapter_title^2" ] ① 
    }
}
```

① chapter_title 字段有一个值为 2 的 boost 值，而book_title 和 section_title 字段则还是默认的 1

***

The multi_match query provides a convenient shorthand way of running the same query against multiple fields.

> **Note**
> 
> There are several types of multi_match query, three of which just happen to coincide with the three scenarios that we listed in Know Your Data: best_fields, most_fields, and cross_fields.

By default, this query runs as type best_fields, which means that it generates a match query for each field and wraps them in a dis_max query. This dis_max query

```bash
{
  "dis_max": {
    "queries":  [
      {
        "match": {
          "title": {
            "query": "Quick brown fox",
            "minimum_should_match": "30%"
          }
        }
      },
      {
        "match": {
          "body": {
            "query": "Quick brown fox",
            "minimum_should_match": "30%"
          }
        }
      },
    ],
    "tie_breaker": 0.3
  }
}
```

could be rewritten more concisely with multi_match as follows:

```bash
{
    "multi_match": {
        "query":                "Quick brown fox",
        "type":                 "best_fields", ①
        "fields":               [ "title", "body" ],
        "tie_breaker":          0.3,
        "minimum_should_match": "30%" ②
    }
}
```
[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/25_Best_fields.json) 


① The best_fields type is the default and can be left out.

② Parameters like minimum_should_match or operator are passed through to the generated match queries.

## Using Wildcards in Field Names

Field names can be specified with wildcards: any field that matches the wildcard pattern will be included in the search. You could match on the book_title, chapter_title, and section_title fields, with the following:

```bash
{
    "multi_match": {
        "query":  "Quick brown fox",
        "fields": "*_title"
    }
}
```

## Boosting Individual Fields

Individual fields can be boosted by using the caret (^) syntax: just add ^boost after the field name, where boost is a floating-point number:

```bash
{
    "multi_match": {
        "query":  "Quick brown fox",
        "fields": [ "*_title", "chapter_title^2" ] ① 
    }
}
```

① The chapter_title field has a boost of 2, while the book_title and section_title fields have a default boost of 1.
