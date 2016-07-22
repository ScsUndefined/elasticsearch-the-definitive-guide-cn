# 4.3.6 Most Fields 越多字段匹配则越优

在进行全文检索的时候，你既要保证相关的数据都会被查询出来，又要保证不相关的内容不会被误查询出来，这两者其实存在竞争，你需要妥善地处理好什么才是相关的什么才是不相关的。而全文检索的目标则是把最相关的结果优先呈现给用户。

要保证所有相关的数据都被搜索出来，我们就得广撒网，即不仅是把包含了用户搜索关键词的数据查询出来，还要把和关键词相关的文档也返回给用户。比如，如果一个用户搜索了“敏捷的棕色狐狸”，那么含有“跑得快的狐狸”的文档也应该被认为是一个相关的文档，应该被返回给用户。

如果我们有 100 个文档里含有“敏捷的棕色狐狸”，而只有一个文档里含有 “跑得快的狐狸”，那么在最终用户得到的搜索结果中，那个含有“跑得快的狐狸”的文档会被垫底显示，因为相对于含有“敏捷的棕色狐狸”的文档，这个文档的相关度就没那么高了。我们会查询出非常多的相关数据，也因此我们要保证最相关的匹配结果永远被排在最上面。

一种常用的技术手段是。在入索引的时候把同一个文本以多种方式来索引一遍，每种方式都提供一种不同的相关度评估维度。而主字段则可以是包含了所有单词的词根形式的字段，以期能够尽可能多地查出相关的文档。比如我们可以这么来规划我们的主字段：
 
 * 用单词的词根形式来进行索引，比如如果原文是“jumps”,"jumping"或者"jumped"，在索引中则会存它们的词根“jump”。然后即使用户搜索的是“jumped”，我们仍然会把包含有“jumping”的结果给返回。
 * 包含了同义词。比如“jump”相关的文本在入索引的时候，也会被入成"leap"和“hop”。~~别问我为什么，磁盘空间大，任性。~~
 * 移除了音标。比如 ésta, está, 和 esta 都会被索引成 esta

这时候如果用户输入了“jumping”那么又有两个文档一个含有“jumped”一个则含有“jumping”，那么这两个文档都会被认为是匹配的结果，会返回给用户。这时候用户就会希望含有“jumping”的结果排在含有“jumped”的结果之前。因为它含有用户输入的原文“jumping”。

而此时主字段里只存了词的词根形式，即jump。就没法区分文档中的 “jumped” 和 “jumping” 究竟哪个词更匹配用户输入的“jumping”。要识别这个也不难，只要把同样的文本在入索引的时候，以其他形式入索引成一个新的副字段就行了。副字段就能用来进行更精确的匹配操作。所谓的其他形式，可以是保留词的未提取词根前的状态，或者保留词的音标，或者保留词的前后出现次序的信息。这些新的字段就为全文检索提供了新的评分维度。越是多的字段被匹配，那么结果的质量就越高。

如果一个文档的主字段中有用户输入的查询关键词，那它就会被认为是用户想要的结果。如果它的副字段，也含有用户输入的信息，那么该文档就会获得额外的加分，并被优先显示在搜索结果集中。

有关同义词，词序等内容会在本书的后续章节介绍，而本章我们仅使用词根来解释这一技术手段的细节。

## Multifield Mapping 多字段地映射

首先我们得把我们的数据索引两次，一次索引成词根形式，一种则索引成未提取词根的形式。要这么做的话我们需要使用 multifields。这我们在 String Sorting and Multifields 中已经接触过了：

```bash
DELETE /my_index

PUT /my_index
{
    "settings": { "number_of_shards": 1 }, ①
    "mappings": {
        "my_type": {
            "properties": {
                "title": { ②
                    "type":     "string",
                    "analyzer": "english",
                    "fields": {
                        "std":   { ③
                            "type":     "string",
                            "analyzer": "standard"
                        }
                    }
                }
            }
        }
    }
}
```
[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/30_Most_fields.json) 


① See [Relevance Is Broken!](https://www.elastic.co/guide/en/elasticsearch/guide/current/relevance-is-broken.html).

② title 字段被 english 解析器处理过了，所有单词都会被提纯成它的词根形式

③ title.std 字段使用弄个了 standard 解析，所以单词都会保留去词根前的原始状态

接下来我们索引下面这两个文档：

```bash
PUT /my_index/my_type/1
{ "title": "My rabbit jumps" }

PUT /my_index/my_type/2
{ "title": "Jumping jack rabbits" }
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/30_Most_fields.json) 

接下来我们对 titile 进行搜索，搜出所有和 “jumping rabbits” 相关的文档：

```bash
GET /my_index/_search
{
   "query": {
        "match": {
            "title": "jumping rabbits"
        }
    }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/30_Most_fields.json) 

由于采用了 english 解析器，所有这个查询在内部会被转化成对两个词根“jump”和“rabbit”的搜索。两个文档的 titile 字段都含有这两个词根，所以都会被搜索出来，并且相关度评分也是一致的：

```bash
{
  "hits": [
     {
        "_id": "1",
        "_score": 0.42039964,
        "_source": {
           "title": "My rabbit jumps"
        }
     },
     {
        "_id": "2",
        "_score": 0.42039964,
        "_source": {
           "title": "Jumping jack rabbits"
        }
     }
  ]
}
```

If we were to query just the title.std field, then only document 2 would match. However, if we were to query both fields and to combine their scores by using the bool query, then both documents would match (thanks to the title field) and document 2 would score higher (thanks to the title.std field):

如果我们对 title.std 字段进行搜索的话，那么只有第二个文档会被搜索出来。但如果你同时搜索两个字段，并使用bool查询来整合针对两个字段的查询子语句，那么两个文档就都会被搜索出来（因为 title 字段都命中了），并且第二个文档的得分还高一点（因为第二个文档的 title.std 额外命中了搜索关键词）：
```bash
GET /my_index/_search
{
  "query":{
    "bool":{
      "should":[
        {"match":{"title":"jumping rabbits"}},
        {"match":{"title.std":"jumping rabbits"}}
      ]
    }
  }
}
```

改写成 multi_match 就是：

```bash
GET /my_index/_search
{
   "query": {
        "multi_match": {
            "query":  "jumping rabbits",
            "type":   "most_fields", ①
            "fields": [ "title", "title.std" ]
        }
    }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/30_Most_fields.json) 

① 我们想要根据一个文档的各个字段对关键词的匹配程度来综合评估这个文档的相关度，因此我们选择了 most_fields 类型。这会导致 multi_match 在内部会被当作整合了“title”和“title.std”的这两个字段级查询子句的 bool 查询，而非一个 dis_max 查询

结果如下：

```bash
{
  "hits": [
     {
        "_id": "2",
        "_score": 0.8226396, ①
        "_source": {
           "title": "Jumping jack rabbits"
        }
     },
     {
        "_id": "1",
        "_score": 0.10741998, ②
        "_source": {
           "title": "My rabbit jumps"
        }
     }
  ]
}
``` 

①,② 第二个文档的相关度比第一个高（好多\~\~\~）

我们使用匹配范围更广泛的 title 字段来尽可能地增加匹配的结果——以增加查全率——然后我们使用 title.std 字段来作为一个新的维度以让更相关的结果优先显示出来。

每个字段的得分对最终的相关度评分造成的影响可以通过 boost 参数来控制，举个栗子，我们可以认为 titile 字段是最重要的字段，并给其增加权重，相对得这也就造成了其他字段的权重下降：

```bash
GET /my_index/_search
{
   "query": {
        "multi_match": {
            "query":       "jumping rabbits",
            "type":        "most_fields",
            "fields":      [ "title^10", "title.std" ] ①
        }
    }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/30_Most_fields.json) 

① title 字段的 boost 值为 10。这使得该字段的相关性比 title.std 字段要高得多得多。

***

# Most Fields

Full-text search is a battle between recall—returning all the documents that are relevant—and precision—not returning irrelevant documents. The goal is to present the user with the most relevant documents on the first page of results.

To improve recall, we cast the net wide—we include not only documents that match the user’s search terms exactly, but also documents that we believe to be pertinent to the query. If a user searches for “quick brown fox,” a document that contains fast foxes may well be a reasonable result to return.

If the only pertinent document that we have is the one containing fast foxes, it will appear at the top of the results list. But of course, if we have 100 documents that contain the words quick brown fox, then the fast foxes document may be considered less relevant, and we would want to push it further down the list. After including many potential matches, we need to ensure that the best ones rise to the top.

A common technique for fine-tuning full-text relevance is to index the same text in multiple ways, each of which provides a different relevance signal. The main field would contain terms in their broadest-matching form to match as many documents as possible. For instance, we could do the following:

  * Use a stemmer to index jumps, jumping, and jumped as their root form: jump. Then it doesn’t matter if the user searches for jumped; we could still match documents containing jumping.
  * Include synonyms like jump, leap, and hop.
  * Remove diacritics, or accents: for example, ésta, está, and esta would all be indexed without accents as esta.

However, if we have two documents, one of which contains jumped and the other jumping, the user would probably expect the first document to rank higher, as it contains exactly what was typed in.

We can achieve this by indexing the same text in other fields to provide more-precise matching. One field may contain the unstemmed version, another the original word with diacritics, and a third might use shingles to provide information about word proximity. These other fields act as signals that increase the relevance score of each matching document. The more fields that match, the better.

A document is included in the results list if it matches the broad-matching main field. If it also matches the signal fields, it gets extra points and is pushed up the results list.

We discuss synonyms, word proximity, partial-matching and other potential signals later in the book, but we will use the simple example of stemmed and unstemmed fields to illustrate this technique.

# Multifield Mapping

The first thing to do is to set up our field to be indexed twice: once in a stemmed form and once in an unstemmed form. To do this, we will use multifields, which we introduced in String Sorting and Multifields:

```bash
DELETE /my_index

PUT /my_index
{
    "settings": { "number_of_shards": 1 }, ①
    "mappings": {
        "my_type": {
            "properties": {
                "title": { ②
                    "type":     "string",
                    "analyzer": "english",
                    "fields": {
                        "std":   { ③
                            "type":     "string",
                            "analyzer": "standard"
                        }
                    }
                }
            }
        }
    }
}
```
[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/30_Most_fields.json) 

① See Relevance Is Broken!.

② The title field is stemmed by the english analyzer.

③ The title.std field uses the standard analyzer and so is not stemmed.

Next we index some documents:

```bash
PUT /my_index/my_type/1
{ "title": "My rabbit jumps" }

PUT /my_index/my_type/2
{ "title": "Jumping jack rabbits" }
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/30_Most_fields.json)

Here is a simple match query on the title field for jumping rabbits:

```bash
GET /my_index/_search
{
   "query": {
        "match": {
            "title": "jumping rabbits"
        }
    }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/30_Most_fields.json)

This becomes a query for the two stemmed terms jump and rabbit, thanks to the english analyzer. The title field of both documents contains both of those terms, so both documents receive the same score:

```bash
{
  "hits": [
     {
        "_id": "1",
        "_score": 0.42039964,
        "_source": {
           "title": "My rabbit jumps"
        }
     },
     {
        "_id": "2",
        "_score": 0.42039964,
        "_source": {
           "title": "Jumping jack rabbits"
        }
     }
  ]
}
```

If we were to query just the title.std field, then only document 2 would match. However, if we were to query both fields and to combine their scores by using the bool query, then both documents would match (thanks to the title field) and document 2 would score higher (thanks to the title.std field):

```bash
GET /my_index/_search
{
   "query": {
        "multi_match": {
            "query":  "jumping rabbits",
            "type":   "most_fields", ①
            "fields": [ "title", "title.std" ]
        }
    }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/30_Most_fields.json) 

① We want to combine the scores from all matching fields, so we use the most_fields type. This causes the multi_match query to wrap the two field-clauses in a bool query instead of a dis_max query.

```bash
{
  "hits": [
     {
        "_id": "2",
        "_score": 0.8226396, ①
        "_source": {
           "title": "Jumping jack rabbits"
        }
     },
     {
        "_id": "1",
        "_score": 0.10741998, ②
        "_source": {
           "title": "My rabbit jumps"
        }
     }
  ]
}
```

①,② Document 2 now scores much higher than document 1.

We are using the broad-matching title field to include as many documents as possible—to increase recall—but we use the title.std field as a signal to push the most relevant results to the top.

The contribution of each field to the final score can be controlled by specifying custom boost values. For instance, we could boost the title field to make it the most important field, thus reducing the effect of any other signal fields:

```bash
GET /my_index/_search
{
   "query": {
        "multi_match": {
            "query":       "jumping rabbits",
            "type":        "most_fields",
            "fields":      [ "title^10", "title.std" ] ①
        }
    }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/30_Most_fields.json) 

① The boost value of 10 on the title field makes that field relatively much more important than the title.std field.