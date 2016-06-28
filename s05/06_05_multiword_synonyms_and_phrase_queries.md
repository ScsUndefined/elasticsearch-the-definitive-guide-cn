# 5.6.5 Multiword Synonyms and Phrase Queries  多词同义与短语查询

至此，同义词看上去还挺简单的有木有。然而不幸的是，复杂的部分才刚刚开始。为了能使  [短语查询](https://www.elastic.co/guide/en/elasticsearch/guide/current/phrase-matching.html) 正常工作，Elasticsearch 需要知道每个词在初始文本中的位置。多次同意会严重破坏词的位置信息，尤其当被新增的同义词标记的长度各不相同的时候。

我们创建一个同义词标记过滤器，然后使用下面这样的同义词规则：

```
"usa,united states,u s a,united states of america"
```

```bash
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_synonym_filter": {
          "type": "synonym",
          "synonyms": [
            "usa,united states,u s a,united states of america"
          ]
        }
      },
      "analyzer": {
        "my_synonyms": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_synonym_filter"
          ]
        }
      }
    }
  }
}

GET /my_index/_analyze?analyzer=my_synonyms&text=
The United States is wealthy
```

解析器会输出下面这样的结果：

```
Pos 1:  (the)
Pos 2:  (usa,united,u,united)
Pos 3:  (states,s,states)
Pos 4:  (is,a,of)
Pos 5:  (wealthy,america)
```

如果你用上面这个同义词标记过滤器索引一个文档，然后执行一个短语查询，那你就会得到骇人的结果，即下面这些短语都不会匹配成功：

  * The usa is wealthy
  * The united states of america is wealthy
  * The U.S.A. is wealthy
  
但是下面这些短语就会：

  * United states is wealthy
  * Usa states of wealthy
  * The U.S. of wealthy
  * U.S. is america

如果你是在查询阶段使同义词，那你就会看到更加诡异的匹配结果。看下这个 `validate-query` 查询:

```bash
GET /my_index/_validate/query?explain
{
  "query": {
    "match_phrase": {
      "text": {
        "query": "usa is wealthy",
        "analyzer": "my_synonyms"
      }
    }
  }
}
```

查询关键字会被同义词标记过滤器处理成类似这样的信息：

```
"(usa united u united) (is states s states) (wealthy a of) america"
```

这会匹配包含有 `u is of america` 的文档，但是匹配不出任何含有 `america` 的文档

> **提示**
> 
> 多词同义对高亮匹配结果也会造成影响。一个针对 `USA` 的查询，返回的结果可能却高亮了 *United States* ，比如可能返回结果会像这样： “The *United States* is wealthy”.

## 使用简单缩写式进行短语查询

避免这一乱象的一种方式就是使用 [简单缩写式](https://www.elastic.co/guide/en/elasticsearch/guide/current/synonyms-expand-or-contract.html#synonyms-contraction) 来用单个词表示所有的同义词，然后在查询阶段，就只需要针对这单个词进行查询了：

```bash
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_synonym_filter": {
          "type": "synonym",
          "synonyms": [
            "united states,u s a,united states of america=>usa"
          ]
        }
      },
      "analyzer": {
        "my_synonyms": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_synonym_filter"
          ]
        }
      }
    }
  }
}

GET /my_index/_analyze?analyzer=my_synonyms
The United States is wealthy
```

上面那个查询信息就会被处理成类似下面这样：

```
Pos 1:  (the)
Pos 2:  (usa)
Pos 3:  (is)
Pos 5:  (wealthy)
```

现在我们再次执行我们之前做过的那个  `validate-query` 查询，就会输出一个简单又合理的结果：

```
"usa is wealthy"
```

这个方法的缺点是，因为把 `united states of america` 转换成了同义词 `usa`，你就不能使用 `united states of america` 去搜索出 `united` 或者 `states` 等词。你需要使用一个额外的字段并用另一个解析器链来达到这个目的。

## 同义词与 query_string 查询

本书很少谈论到 `query_string` 查询，因为真心不推荐你用它。在“[More-Complicated Queries](https://www.elastic.co/guide/en/elasticsearch/guide/current/search-lite.html#query-string-query)”一节中有提到，由于 `query_string` 查询支持一个精简的查询语法，因此，可能这会导致它搜出一些出人意料的结果或者甚至是含有语法错误的结果。

这种查询方式存在不少问题，而其中之一便与多词同义有关。为了支持它的查询语法，你必须把查询文本用指定的，该语法所能识别的操作符号来标示，比如 `AND`, `OR`, `+`, `-`, `field:`等。（更多相关内容参阅  [`query_string` syntax](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html#query-string-syntax) ）

而在这种语法的解析过程中，解析动作会把查询文本在空格符处作切分，然后分别把每个切分出来的词传递给相关性解析器。这也即意味着你的同义词解析器永远都不可能收到类似“united states of america”这样的多个单词组成的同义词。由于不会把`United States`作为一个原子性的文本，所以同义词解析器的输入信息永远都是两个被切分开的词 `United` 和 `States`。

所幸，`match`查询相比而言就可靠得多了，因为它不支持上述语法，所以多个字组成的同义词不会被切分开，而是会完整地交给解析器处理。

***

So far, synonyms appear to be quite straightforward. Unfortunately, this is where things start to go wrong. For [phrase queries](https://www.elastic.co/guide/en/elasticsearch/guide/current/phrase-matching.html) to function correctly, Elasticsearch needs to know the position that each term occupies in the original text. Multiword synonyms can play havoc with term positions, especially when the injected synonyms are of differing lengths.

To demonstrate, we’ll create a synonym token filter that uses this rule:

```
"usa,united states,u s a,united states of america"
```

```bash
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_synonym_filter": {
          "type": "synonym",
          "synonyms": [
            "usa,united states,u s a,united states of america"
          ]
        }
      },
      "analyzer": {
        "my_synonyms": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_synonym_filter"
          ]
        }
      }
    }
  }
}

GET /my_index/_analyze?analyzer=my_synonyms&text=
The United States is wealthy
```

The tokens emitted by the analyze request look like this:

```
Pos 1:  (the)
Pos 2:  (usa,united,u,united)
Pos 3:  (states,s,states)
Pos 4:  (is,a,of)
Pos 5:  (wealthy,america)
```

If we were to index a document analyzed with synonyms as above, and then run a phrase query without synonyms, we’d have some surprising results. These phrases would not match:

  * The usa is wealthy
  * The united states of america is wealthy
  * The U.S.A. is wealthy
  
However, these phrases would:

  * United states is wealthy
  * Usa states of wealthy
  * The U.S. of wealthy
  * U.S. is america
  
If we were to use synonyms at query time instead, we would see even more-bizarre matches. Look at the output of this `validate-query` request:

```bash
GET /my_index/_validate/query?explain
{
  "query": {
    "match_phrase": {
      "text": {
        "query": "usa is wealthy",
        "analyzer": "my_synonyms"
      }
    }
  }
}
```

The explanation is as follows:

```
"(usa united u united) (is states s states) (wealthy a of) america"
```

This would match documents containg `u is of america` but wouldn’t match any document that didn’t contain the term `america`.

> **Tip**
> 
> Multiword synonyms affect highlighting in a similar way. A query for `USA` could end up returning a highlighted snippet such as: “The *United States is wealthy*”.

## Use Simple Contraction for Phrase Queries

The way to avoid this mess is to use [simple contraction](https://www.elastic.co/guide/en/elasticsearch/guide/current/synonyms-expand-or-contract.html#synonyms-contraction) to inject a single term that represents all synonyms, and to use the same synonym token filter at query time:

```bash
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_synonym_filter": {
          "type": "synonym",
          "synonyms": [
            "united states,u s a,united states of america=>usa"
          ]
        }
      },
      "analyzer": {
        "my_synonyms": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_synonym_filter"
          ]
        }
      }
    }
  }
}

GET /my_index/_analyze?analyzer=my_synonyms
The United States is wealthy
```

The result of the preceding `analyze` request looks much more sane:

```
Pos 1:  (the)
Pos 2:  (usa)
Pos 3:  (is)
Pos 5:  (wealthy)
```

And repeating the `validate-query` request that we made previously yields a simple, sane explanation:

```
"usa is wealthy"
```

The downside of this approach is that, by reducing `united states of america` down to the single term `usa`, you can’t use the same field to find just the word `united` or `states`. You would need to use a separate field with a different analysis chain for that purpose.

## Synonyms and the query_string Query

We have tried to avoid discussing the `query_string` query because we don’t recommend using it. In "[More-Complicated Queries](https://www.elastic.co/guide/en/elasticsearch/guide/current/search-lite.html#query-string-query)", we said that, because the `query_string` query supports a terse mini search-syntax, it could frequently lead to surprising results or even syntax errors.

One of the gotchas of this query involves multiword synonyms. To support its search-syntax, it has to parse the query string to recognize special operators like `AND`, `OR`, `+`, `-`, `field:`, and so forth. (See the full [`query_string` syntax](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-query-string-query.html#query-string-syntax) for more information.)

As part of this parsing process, it breaks up the query string on whitespace, and passes each word that it finds to the relevant analyzer separately. This means that your synonym analyzer will never receive a multiword synonym. Instead of seeing `United States` as a single string, the analyzer will receive `United` and `States` separately.

Fortunately, the trustworthy `match` query supports no such syntax, and multiword synonyms will be passed to the analyzer in their entirety.