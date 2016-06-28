# 5.6.6 Symbol Synonyms 符号同义词

最后一节内容我们来阐述下怎么对符号进行同义词处理，这和我们前面讲的同义词处理还真不太一样。**符号同义词** 是用别名来表示这个符号，以防止它在分词过程中被误认为是不重要的标点符号而被移除。

虽然绝大多数情况下，符号对于全文搜索而言都无关紧要，但有的时候字符组合，比如说字符组合而成的表情，或许又会是很有意义的东西，甚至有时候会感觉整个句子的含义，对比一下这两句话：

  * I am thrilled to be at work on Sunday. ~~大概可以理解为“虽然是礼拜天，但我依然在兴奋地工作”，工作狂~~
  * I am thrilled to be at work on Sunday :( ~~“周末了，我却还在紧张地加班”，加班狗~~

`标准` 分词器或许会简单地消除掉第二个句子里的字符表情，致使两个原本意思相去甚远的句子变得相同。

我们可以在文本被递交给分词器处理之前，先使用 [`映射` 字符过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-mapping-charfilter.html) 来把字符表情替换成 `emoticon_happy` 或者 `emoticon_sad` 。

```bash
PUT /my_index
{
  "settings": {
    "analysis": {
      "char_filter": {
        "emoticons": {
          "type": "mapping",
          "mappings": [ ①
            ":)=>emoticon_happy",
            ":(=>emoticon_sad"
          ]
        }
      },
      "analyzer": {
        "my_emoticons": {
          "char_filter": "emoticons",
          "tokenizer":   "standard",
          "filter":    [ "lowercase" ]
          ]
        }
      }
    }
  }
}

GET /my_index/_analyze?analyzer=my_emoticons
I am :) not :( ②
```

① `映射` 过滤器把字符从 `=>` 左边的格式转变成右边的样子

② 输出： `i`, `am`, `emoticon_happy`, `not`, `emoticon_sad`

很少有人会搜 `emoticon_happy` 这个词，但是确保类似字符表情这类重要的符号被存储到索引中是非常好的做法，在进行情感分析的时候会很有用。当然，我们也可以用真实的词汇来处理符号同义词，比如 `happy` ~~快乐~~ 或者 `sad` 伤感。

> **提示**
> 
> `映射` 字符过滤器是个非常有用的过滤器，它可以用来对一些已有的字词进行替换操作。你如果想要采用更灵活的正则表达式去替换字词的话，那你可以使用 `pattern_replace` 字符过滤器。

***

The final part of this chapter is devoted to symbol synonyms, which are unlike the synonyms we have discussed until now. *Symbol synonyms* are string aliases used to represent symbols that would otherwise be removed during tokenization.

While most punctuation is seldom important for full-text search, character combinations like emoticons may be very signficant, even changing the meaning of the text. Compare these:

  * I am thrilled to be at work on Sunday.
  * I am thrilled to be at work on Sunday :(
  
The `standard` tokenizer would simply strip out the emoticon in the second sentence, conflating two sentences that have quite different intent.

We can use the [`mapping` character filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-mapping-charfilter.html) to replace emoticons with symbol synonyms like `emoticon_happy` and `emoticon_sad` before the text is passed to the tokenizer:

```bash
PUT /my_index
{
  "settings": {
    "analysis": {
      "char_filter": {
        "emoticons": {
          "type": "mapping",
          "mappings": [ ①
            ":)=>emoticon_happy",
            ":(=>emoticon_sad"
          ]
        }
      },
      "analyzer": {
        "my_emoticons": {
          "char_filter": "emoticons",
          "tokenizer":   "standard",
          "filter":    [ "lowercase" ]
          ]
        }
      }
    }
  }
}

GET /my_index/_analyze?analyzer=my_emoticons
I am :) not :( ②
```

① The `mappings` filter replaces the characters to the left of `=>` with those to the right.

② Emits tokens `i`, `am`, `emoticon_happy`, `not`, `emoticon_sad`.

It is unlikely that anybody would ever search for `emoticon_happy`, but ensuring that important symbols like emoticons are included in the index can be helpful when doing sentiment analysis. Of course, we could equally have used real words, like `happy` and `sad`.

> **Tip**
> 
> The `mapping` character filter is useful for simple replacements of exact character sequences. For more-flexible pattern matching, you can use regular expressions with the `pattern_replace` character filter.