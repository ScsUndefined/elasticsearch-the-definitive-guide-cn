# 5.1.5 One Language per Field 单个字段对应一门主语言

许多文档都是用来表示某个特定的实体的，比如产品，电影或者法律通告，经常会被翻译成多个语言版本。除了上述介绍的做法，把每种语言的版本都可以存储在单个索引的单个文档中，这儿还有一种可行的方式，就是，把各个语言版本都存储在同一个文档中：

```bash
{
   "title":     "Fight club",
   "title_br":  "Clube de Luta",
   "title_cz":  "Klub rváčů",
   "title_en":  "Fight club",
   "title_es":  "El club de la lucha",
   ...
}
```

每种语言版本的信息都被存储在各个分开的字段中，并会被各自特定的语言解析器处理：

```bash
PUT /movies
{
  "mappings": {
    "movie": {
      "properties": {
        "title": { ①
          "type":       "string"
        },
        "title_br": { ②
            "type":     "string",
            "analyzer": "brazilian"
        },
        "title_cz": { ③
            "type":     "string",
            "analyzer": "czech"
        },
        "title_en": { ④
            "type":     "string",
            "analyzer": "english"
        },
        "title_es": { ⑤
            "type":     "string",
            "analyzer": "spanish"
        }
      }
    }
  }
}
```

① 这个 titile 字段保存初始内容，并使用标准解析器处理

②③④⑤ 多语言版本的 title 会使用各自特定的语言解析器处理   

和“每个语言对应一个索引”的解决方案一样，“每个字段对应一个索引”方案也保证了统计词频时不会因为多语言而导致结果不准确。但是这个方案没有前者灵活。尽管使用 [update-mapping API](https://www.elastic.co/guide/en/elasticsearch/guide/current/mapping-intro.html#updating-a-mapping) 新增字段的时候非常方便，但是新的字段可能需要一个对应的，新的语言解析器，而这个解析器的设置只能在索引创建的时候指定。当然有个变通的方法，那就是先 [关掉](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-open-close.html) 索引，然后使用 [update-settings API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-update-settings.html) 来新增解析器，之后再重启索引，但是关闭索引就意味着你需要有一段停工时间。~~老板：程序员不是都喜欢晚下班嘛，让他们熬到凌晨3点搞这东西不就行了？我先回去睡<strong style="background-color:black">秘书</strong>了~~

单个语言的文档可以被分开查询，或者查询操作也可以以多个语言为目标，去查询多个字段。我们甚至可以提升字段的权重来优先显示某个语言的结果：

```bash
GET /movies/movie/_search
{
    "query": {
        "multi_match": {
            "query":    "club de la lucha",
            "fields": [ "title*", "title_es^2" ], ①
            "type":     "most_fields"
        }
    }
}
```

① 这个查询操作查询了所有以“title”开头的字段，但是用数字 2 来提升了 title_es 的权重。而其他的字段相应得，只会有个中性值 1。

***

For documents that represent entities like products, movies, or legal notices, it is common for the same text to be translated into several languages. Although each translation could be represented in a single document in an index per language, another reasonable approach is to keep all translations in the same document:

```bash
{
   "title":     "Fight club",
   "title_br":  "Clube de Luta",
   "title_cz":  "Klub rváčů",
   "title_en":  "Fight club",
   "title_es":  "El club de la lucha",
   ...
}
```

Each translation is stored in a separate field, which is analyzed according to the language it contains:

```bash
PUT /movies
{
  "mappings": {
    "movie": {
      "properties": {
        "title": { ①
          "type":       "string"
        },
        "title_br": { ②
            "type":     "string",
            "analyzer": "brazilian"
        },
        "title_cz": { ③
            "type":     "string",
            "analyzer": "czech"
        },
        "title_en": { ④
            "type":     "string",
            "analyzer": "english"
        },
        "title_es": { ⑤
            "type":     "string",
            "analyzer": "spanish"
        }
      }
    }
  }
}
```

① The title field contains the original title and uses the standard analyzer.

②③④⑤ Each of the other fields uses the appropriate analyzer for that language.

Like the index-per-language approach, the field-per-language approach maintains clean term frequencies. It is not quite as flexible as having separate indices. Although it is easy to add a new field by using the [update-mapping API](https://www.elastic.co/guide/en/elasticsearch/guide/current/mapping-intro.html#updating-a-mapping), those new fields may require new custom analyzers, which can only be set up at index creation time. As a workaround, you can [close](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-open-close.html) the index, add the new analyzers with the [update-settings API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-update-settings.html), then reopen the index, but closing the index means that it will require some downtime.

The documents of a single language can be queried independently, or queries can target multiple languages by querying multiple fields. We can even specify a preference for particular languages by boosting that field:

```bash
GET /movies/movie/_search
{
    "query": {
        "multi_match": {
            "query":    "club de la lucha",
            "fields": [ "title*", "title_es^2" ], ①
            "type":     "most_fields"
        }
    }
}
```

  ① This search queries any field beginning with title but boosts the title_es field by 2. All other fields have a neutral boost of 1.