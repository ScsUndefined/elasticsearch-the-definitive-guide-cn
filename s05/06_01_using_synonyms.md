# 5.6.1 Using Synonyms 使用同义词

同义词特性可以通过，使用 [同义词标记过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-synonym-tokenfilter.html) 来替换现有的标记，或者新的标记增加到标记流中，来使用：

```bash
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_synonym_filter": {
          "type": "synonym", ①
          "synonyms": [ ②
            "british,english",
            "queen,monarch"
          ]
        }
      },
      "analyzer": {
        "my_synonyms": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_synonym_filter" ③
          ]
        }
      }
    }
  }
}
```

① 首先我们定义了一个 `同义词` 类型的标记过滤器

② 我们会在 [Formatting Synonyms](https://www.elastic.co/guide/en/elasticsearch/guide/current/synonym-formats.html) 章节中深入讨论“同义词的格式”

③ 随后我们创建一个使用了 `my_synonym_filter` 的自定义解析器

> **提示**
> 
> 同义词可以使用 `synonyms` 参数来内嵌指定，或者通过集群中每个节点都持有着的一份同义词文件来指定。同义词文件的路径应该通过 `synonyms_path` 参数来指定，并且它的值应该是绝对路径或者相对于 `config` 文件夹的相对路径。你可以查阅 [Updating Stopwords](https://www.elastic.co/guide/en/elasticsearch/guide/current/using-stopwords.html#updating-stopwords) 文档来了解怎么刷新同义词列表。

通过 `analyze` API 来测试我们的解析器：

```bash
GET /my_index/_analyze?analyzer=my_synonyms
Elizabeth is the English queen

```bash
Pos 1: (elizabeth)
Pos 2: (is)
Pos 3: (the)
Pos 4: (british,english) ①
Pos 5: (queen,monarch) ②
``` 

① ② 所有的同义词都处于原始标记流的同一个位置

一个这样的文档就可以匹配任何这样的查询： `English queen`, `British queen`, `English monarch`, 或者 `British monarch`。甚至一个短语查询都能把该文档查出来，因为每种术语的位置都被记录了下来。

>  **提示**
>  
> 在入索引阶段和查询阶段使用同一个 `同义词` 标记过滤器是完全没有必要的。比如说，如果我们在入索引阶段，我们把 `English` 用 `englist` 和 `british` 这两个词来代替，然后在查询阶段我们只需要查询这些词中的某一个。我们换个角度再打个比方，如果在入索引的时候不使用同义词，之后我们可以在查询阶段，把一个针对 `English` 的查询转化成一个针对 `english` 或者 `british` 的查询。
>
> 至于到底是在入索引阶段进行同义词拓展操作还是在查询阶段，这其实是个比较难以决断的问题。我们会在 [Expand or contract](https://www.elastic.co/guide/en/elasticsearch/guide/current/synonyms-expand-or-contract.html) 一文中深入探究这个问题。

***

Synonyms can replace existing tokens or be added to the token stream by using the [synonym token filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-synonym-tokenfilter.html):

```bash
PUT /my_index
{
  "settings": {
    "analysis": {
      "filter": {
        "my_synonym_filter": {
          "type": "synonym", ①
          "synonyms": [ ②
            "british,english",
            "queen,monarch"
          ]
        }
      },
      "analyzer": {
        "my_synonyms": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_synonym_filter" ③
          ]
        }
      }
    }
  }
}
```

① First, we define a token filter of type `synonym`.

② We discuss synonym formats in [Formatting Synonyms](https://www.elastic.co/guide/en/elasticsearch/guide/current/synonym-formats.html).

③ Then we create a custom analyzer that uses the `my_synonym_filter`.

> **Tip**
> 
> Synonyms can be specified inline with the `synonyms` parameter, or in a synonyms file that must be present on every node in the cluster. The path to the synonyms file should be specified with the `synonyms_path` parameter, and should be either absolute or relative to the Elasticsearch `config` directory. See [Updating Stopwords](https://www.elastic.co/guide/en/elasticsearch/guide/current/using-stopwords.html#updating-stopwords) for techniques that can be used to refresh the synonyms list.

Testing our analyzer with the `analyze` API shows the following:

```bash
GET /my_index/_analyze?analyzer=my_synonyms
Elizabeth is the English queen

```bash
Pos 1: (elizabeth)
Pos 2: (is)
Pos 3: (the)
Pos 4: (british,english) ①
Pos 5: (queen,monarch) ②
``` 

① ② All synonyms occupy the same position as the original term.

A document like this will match queries for any of the following: `English queen`, `British queen`, `English monarch`, or `British monarch`. Even a phrase query will work, because the position of each term has been preserved.

>  **Tip**
>  
> Using the same `synonym` token filter at both index time and search time is redundant. If, at index time, we replace `English` with the two terms `english` and `british`, then at search time we need to search for only one of those terms. Alternatively, if we don’t use synonyms at index time, then at search time, we would need to convert a query for `English` into a query for `english` OR `british`.
>
> Whether to do synonym expansion at search or index time can be a difficult choice. We will explore the options more in [Expand or contract](https://www.elastic.co/guide/en/elasticsearch/guide/current/synonyms-expand-or-contract.html).