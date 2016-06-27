# 5.1.6 Mixed-Language Fields 单个字段对应多门语言

在单个字段中混合了多种语言的信息通常都来自于不受你控制的地方，比如从网上抓取的页面信息：

```bash
{ "body": "Page not found / Seite nicht gefunden / Page non trouvée" }
```

它们是最难处理的一类多语言文档。尽管你可以使用标准解析器来处理它，但这样做的话，你的文档就会丢失掉部分可被搜索性，而你如果指定一个恰当的词根提取策略，你文档的可被搜索性就会相对变大。当然，你不可能只选择单一的一种词根提取规则—词根提取规则是针对某个特定语言的，而你的文本却又由多种语言构成。更准确地说，词根提取规则是基于语言以及书写体的。就像在 in [Stemmer per Script](https://www.elastic.co/guide/en/elasticsearch/guide/current/language-pitfalls.html#different-scripts) 中提到的，如果每种语言都属于不同的书写体系，那么它们的词根提取规则就可以混合使用。

假使你正在使用多门基于同一种书写体系的语言，比如拉丁写法，那你有三种可用的方案：

  * 切分成多个独立的字段
  * 多次解析
  * 使用 n-grams
  * 
## 切分成多个独立的字段

在　[Identifying Language](https://www.elastic.co/guide/en/elasticsearch/guide/current/language-pitfalls.html#identifying-language) 一章中提及的语言探测器（CLD）可以告诉你文档的哪个部分是由哪门语言撰写的。然后你就可以根据语言来把文本切分，再用 [One Language per Field](https://www.elastic.co/guide/en/elasticsearch/guide/current/one-lang-fields.html) 中介绍的方法来处理。

## 多次解析

如果你只处理为数不多的几门语言，那你可以使用多个字段来解析你的文本，每个字段对应一种语言：

```bash
PUT /movies
{
  "mappings": {
    "title": {
      "properties": {
        "title": { ①
          "type": "string",
          "fields": {
            "de": { ②
              "type":     "string",
              "analyzer": "german"
            },
            "en": { ③
              "type":     "string",
              "analyzer": "english"
            },
            "fr": { ④
              "type":     "string",
              "analyzer": "french"
            },
            "es": { ⑤
              "type":     "string",
              "analyzer": "spanish"
            }
          }
        }
      }
    }
  }
}
```

① 主字段 title 使用标准解析器

②③④⑤ title 主字段下的每个子字段使用一种不同的语言解析器

## 使用 n-grams

~~还没看过 Ngrams 是什么，这部分暂时直译下，回头再校对下~~

我们可以用在 [Ngrams for Compound Words](https://www.elastic.co/guide/en/elasticsearch/guide/current/ngrams-compound-words.html) 章节中提到的方法来把任何文字索引成 n-grams。Most inflections involve adding a suffix (or in some languages, a prefix) to a word, so by breaking each word into n-grams, you have a good chance of matching words that are similar but not exactly the same.这可以和 多次解析 方法联合使用来为不支持的语言提供一个 catchall 字段：

```bash
PUT /movies
{
  "settings": {
    "analysis": {...} ①
  },
  "mappings": {
    "title": {
      "properties": {
        "title": {
          "type": "string",
          "fields": {
            "de": {
              "type":     "string",
              "analyzer": "german"
            },
            "en": {
              "type":     "string",
              "analyzer": "english"
            },
            "fr": {
              "type":     "string",
              "analyzer": "french"
            },
            "es": {
              "type":     "string",
              "analyzer": "spanish"
            },
            "general": { ②
              "type":     "string",
              "analyzer": "trigrams"
            }
          }
        }
      }
    }
  }
}
```

① 在 analysis 部分，我们定义了一个和 [Ngrams for Compound Words](https://www.elastic.co/guide/en/elasticsearch/guide/current/ngrams-compound-words.html) 中描述到的一模一样的 trigram 解析器。

② `title.general` 字段使用 trigrams 解析器来索引任何语言

当查询一个 catchall general 字段的时候，你可以使用 `minimum_should_match` 查询方法来减少低质量的匹配结果。有时候，给非`general`字段稍微提升下权重也是很有必要的:

```bash
GET /movies/movie/_search
{
    "query": {
        "multi_match": {
            "query":    "club de la lucha",
            "fields": [ "title*^1.5", "title.general" ], ①
            "type":     "most_fields",
            "minimum_should_match": "75%" ②
        }
    }
}
```

① `title` 或者 `title.*` 字段的权重都比 `title.general` 字段高出一丢丢

② `minimum_should_match` 降低了返回结果中的低质量的匹配结果，这个设置对 `title.general` 而言尤其重要

***

Usually, documents that mix multiple languages in a single field come from sources beyond your control, such as pages scraped from the Web:

```bash
{ "body": "Page not found / Seite nicht gefunden / Page non trouvée" }
```

They are the most difficult type of multilingual document to handle correctly. Although you can simply use the standard analyzer on all fields, your documents will be less searchable than if you had used an appropriate stemmer. But of course, you can’t choose just one stemmer—stemmers are language specific. Or rather, stemmers are language and script specific. As discussed in [Stemmer per Script](https://www.elastic.co/guide/en/elasticsearch/guide/current/language-pitfalls.html#different-scripts), if every language uses a different script, then stemmers can be combined.

Assuming that your mix of languages uses the same script such as Latin, you have three choices available to you:

  * Split into separate fields
  * Analyze multiple times
  * Use n-grams

## Split into Separate Fields

The Compact Language Detector mentioned in [Identifying Language](https://www.elastic.co/guide/en/elasticsearch/guide/current/language-pitfalls.html#identifying-language) can tell you which parts of the document are in which language. You can split up the text based on language and use the same approach as was used in [One Language per Field](https://www.elastic.co/guide/en/elasticsearch/guide/current/one-lang-fields.html).

## Analyze Multiple Times

If you primarily deal with a limited number of languages, you could use multi-fields to analyze the text once per language:

```bash
PUT /movies
{
  "mappings": {
    "title": {
      "properties": {
        "title": { ①
          "type": "string",
          "fields": {
            "de": { ②
              "type":     "string",
              "analyzer": "german"
            },
            "en": { ③
              "type":     "string",
              "analyzer": "english"
            },
            "fr": { ④
              "type":     "string",
              "analyzer": "french"
            },
            "es": { ⑤
              "type":     "string",
              "analyzer": "spanish"
            }
          }
        }
      }
    }
  }
}
```

① The main title field uses the standard analyzer.

②③④⑤ Each subfield applies a different language analyzer to the text in the `title` field.

## Use n-grams

You could index all words as n-grams, using the same approach as described in [Ngrams for Compound Words](https://www.elastic.co/guide/en/elasticsearch/guide/current/ngrams-compound-words.html). Most inflections involve adding a suffix (or in some languages, a prefix) to a word, so by breaking each word into n-grams, you have a good chance of matching words that are similar but not exactly the same. This can be combined with the analyze-multiple times approach to provide a catchall field for unsupported languages:

```bash
PUT /movies
{
  "settings": {
    "analysis": {...} ①
  },
  "mappings": {
    "title": {
      "properties": {
        "title": {
          "type": "string",
          "fields": {
            "de": {
              "type":     "string",
              "analyzer": "german"
            },
            "en": {
              "type":     "string",
              "analyzer": "english"
            },
            "fr": {
              "type":     "string",
              "analyzer": "french"
            },
            "es": {
              "type":     "string",
              "analyzer": "spanish"
            },
            "general": { ②
              "type":     "string",
              "analyzer": "trigrams"
            }
          }
        }
      }
    }
  }
}
```

① In the analysis section, we define the same trigrams analyzer as described in [Ngrams for Compound Words](https://www.elastic.co/guide/en/elasticsearch/guide/current/ngrams-compound-words.html).

② The `title.general` field uses the trigrams analyzer to index any language.

When querying the catchall general field, you can use `minimum_should_match` to reduce the number of low-quality matches. It may also be necessary to boost the other fields slightly more than the `general` field, so that matches on the main language fields are given more weight than those on the `general` field:

```bash
GET /movies/movie/_search
{
    "query": {
        "multi_match": {
            "query":    "club de la lucha",
            "fields": [ "title*^1.5", "title.general" ], ①
            "type":     "most_fields",
            "minimum_should_match": "75%" ②
        }
    }
}
```

① All `title` or `title.*` fields are given a slight boost over the `title.general` field.

② The `minimum_should_match` parameter reduces the number of low-quality matches returned, especially important for the `title.general` field.