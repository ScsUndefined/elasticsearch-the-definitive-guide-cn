# 5.1.1 Using Language Analyzers 使用语言解析器

内置的语言解析器是全局可用的，并且不需要在使用前进行配置。在映射字段的时候就可以指定它们：

```bash
PUT /my_index
{
  "mappings": {
    "blog": {
      "properties": {
        "title": {
          "type":     "string",
          "analyzer": "english" ①
        }
      }
    }
  }
}
```

① `title`字段会使用`英文`解析器而不会再采用默认的`标准`解析器。

当然，用`英文`解析器来处理信息，会使我们丢失一些信息:

```
GET /my_index/_analyze?field=title  ①
I'm not happy about the foxes
```

① 解析出的分词: i'm, happi, about, fox

我们无法得知这个文档中提到 fox（狐狸）的时候指的是一只还是一群；`not`是一个停词，所以被移除了，或许我们也无从得知这个文档是对狐狸表示 happy（欢迎）还是 not happy（不欢迎）。所以，使用`英文`解析器会导致我们的匹配原则变得更松散从何增加查全率，但这同时也使得我们不能像从前那样精准地基于内容对文档进行排序。

为了避免上述问题，我们可以使用 [multifields 多字段](https://www.elastic.co/guide/en/elasticsearch/guide/current/multi-fields.html)  特性来对 `title` 建 2 次索引：一次使用`英文`解析器，而另一次使用`标准`解析器。

```bash
PUT /my_index
{
  "mappings": {
    "blog": {
      "properties": {
        "title": { ① 
          "type": "string",
          "fields": {
            "english": { ②
              "type":     "string",
              "analyzer": "english"
            }
          }
        }
      }
    }
  }
}
```

① 主字段 `title` 使用 `标准` 解析器

② 子字段 `title.english` 使用 `英文` 解析器

当这个映射设置好之后，我们就可以索引一些测试文档来示范怎么在查询的时候同时使用这两个字段：

```bash
PUT /my_index/blog/1
{ "title": "I'm happy for this fox" }

PUT /my_index/blog/2
{ "title": "I'm not happy about my fox problem" }

GET /_search
{
  "query": {
    "multi_match": {
      "type":     "most_fields", ①
      "query":    "not happy foxes",
      "fields": [ "title", "title.english" ]
    }
  }
}
```

① 使用 [most_fields 多字段](https://www.elastic.co/guide/en/elasticsearch/guide/current/most-fields.html) 查询方式来在多个字段中匹配同一个文本

尽管 2 个文档都不包含 `foxes` 这个词，但由于 `title.englist` 字段里有词根，所以这 2 个文档都出现在了查询结果中。第二个文档比第一个文档更相关，因为单词 `not` 在 `title` 字段中被匹配成功了。

***

The built-in language analyzers are available globally and don’t need to be configured before being used. They can be specified directly in the field mapping:

```bash
PUT /my_index
{
  "mappings": {
    "blog": {
      "properties": {
        "title": {
          "type":     "string",
          "analyzer": "english" ①
        }
      }
    }
  }
}
```

① The `title` field will use the `english` analyzer instead of the default `standard` analyzer.

Of course, by passing text through the `english` analyzer, we lose information:

```
GET /my_index/_analyze?field=title  ①
I'm not happy about the foxes
```

① Emits token: i'm, happi, about, fox

We can’t tell if the document mentions one `fox` or many `foxes`; the word `not` is a stopword and is removed, so we can’t tell whether the document is happy about foxes or not. By using the `english` analyzer, we have increased recall as we can match more loosely, but we have reduced our ability to rank documents accurately.

To get the best of both worlds, we can use [multifields](https://www.elastic.co/guide/en/elasticsearch/guide/current/multi-fields.html) to index the `title` field twice: once with the `english` analyzer and once with the `standard` analyzer:

```bash
PUT /my_index
{
  "mappings": {
    "blog": {
      "properties": {
        "title": { ① 
          "type": "string",
          "fields": {
            "english": { ②
              "type":     "string",
              "analyzer": "english"
            }
          }
        }
      }
    }
  }
}
```

① The main `title` field uses the `standard` analyzer.

② The `title.english` subfield uses the `english` analyzer.

With this mapping in place, we can index some test documents to demonstrate how to use both fields at query time:

```bash
PUT /my_index/blog/1
{ "title": "I'm happy for this fox" }

PUT /my_index/blog/2
{ "title": "I'm not happy about my fox problem" }

GET /_search
{
  "query": {
    "multi_match": {
      "type":     "most_fields", ①
      "query":    "not happy foxes",
      "fields": [ "title", "title.english" ]
    }
  }
}
```

① Use the [most_fields](https://www.elastic.co/guide/en/elasticsearch/guide/current/most-fields.html) query type to match the same text in as many fields as possible.

Even though neither of our documents contain the word `foxes`, both documents are returned as results thanks to the word stemming on the `title.english` field. The second document is ranked as more relevant, because the word `not` matches on the `title` field.