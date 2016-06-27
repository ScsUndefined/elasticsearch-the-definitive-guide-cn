# 5.2.5 Tidying Up Input Text 优化输入的文本

当输入的文本很规范的时候分词器就可以工作得最完美，所谓 *规范* 即要求输入的文本符合 Unicode 算法期望的格式。但通常情况下，我们需要处理的文本五花八门，一点都规范。所以在分词之前先把它提纯一下，就可以提高分词结果的质量。

## 对 HTML 进行分词

用 `标准` 分词器或者 `icu` 分词器处理 HTML 的时候，分词结果基本无能看。这些分词器根本就不知道怎么处理 HTML 标签，举个例子：

```bash
GET /_analyzer?tokenizer=standard
<p>Some d&eacute;j&agrave; vu <a href="http://somedomain.com>">website</a>
```

`标准` 分词器会混淆 HTML 标签和标签内的正文，并产生这样的结果： `p`, `Some`, `d`, `eacute`, `j`, `agrave`, `vu`, `a`, `href`, `http`, `somedomain.com`, `website`, `a`。明显就是不是我们想要的！

可以在解析器内部增加一个叫*字符过滤器*的东西，它可以在文本被送去给分词器处理之前先对文本进行一次预处理。比如在刚才那种情况下，我们就可以使用 `html_strip` 字符过滤器来移除 HTML 标签并将类似 `&eacute;` 的 HTML 转义字符还原成相应的 Unicode 字符。

字符过滤器可以通过 `analyze` API 来测试，只要在查询字符串中指明它们就行了：

```bash
GET /_analyzer?tokenizer=standard&char_filters=html_strip
<p>Some d&eacute;j&agrave; vu <a href="http://somedomain.com>">website</a>
```

如果要把它们加入到一个解析器中的话，那它们就应该像下面这样被增加到 `自定义的` 解析器定义信息中：

```bash
PUT /my_index
{
    "settings": {
        "analysis": {
            "analyzer": {
                "my_html_analyzer": {
                    "tokenizer":     "standard",
                    "char_filter": [ "html_strip" ]
                }
            }
        }
    }
}
```

一旦创建完毕后，我们新的 `my_html_analyzer` 就可以通过 `analyze` API 来测试了：

```bash
GET /my_index/_analyzer?analyzer=my_html_analyzer
<p>Some d&eacute;j&agrave; vu <a href="http://somedomain.com>">website</a>
```

现在分词器就可以输出我们期望的结果了： `Some`, `déjà`, `vu`, `website`

## 优化标点符号

`标准` 分词器和 `icu` 分词器都能把词 *中* 的单引号正确地视为该词的一部分，而词 *两边* 的单引号却不是词的一部分。在切分 `You're my 'favorite'` 这个文本的时候，输出结果就会是 `You're, my, favorite`。

但不幸的是，Unicode 列出了一些字符，它们有时候都会当做单引号来使用：

`U+0027`
    单引号 (`'`)—原始的 ASCII 字符
    
`U+2018`
    英式单开引号 (`‘`)—用来标记一个引用内容的起始位置
    
`U+2019`
    英式单闭引号 (`’`)—用来标记一个引用内容的结束位置，但也时候在文本中它也会被当做原始版的单引号来使用

当上述字符出现在词中的时候，上述两种分词器都会把它们当做一个单引号来对待（也即认为它们是词的一部分）。然后，接下来是另外三个类似单引号的字符：

`U+201B`
   高位单引号 (`‛`)—作用和 `U+2018` 一样，但是外观不同
    
`U+0091`
    ISO-8859-1 中的英式单开引号—不应该在 Unicode 中使用
    
`U+0092`
    ISO-8859-1 中的英式单闭引号—同样不应该在 Unicode 中使用

上述两种分词器都是视这 3 中字符为一个词的边界，即会在这个地方做切分动作，从而把文本切分为多个部分。问题就出在这里了，一些出版方会使用 `U+201B` 作为一个规范化途径来书写人名，比如`M‛coy`，而后两种字符则也可能会被你的文字处理者使用，至于到底会不会就看它的年代了。

甚至，即使使用了“可被处理的”引号，一个用单引号写的词—`You’re`—也和用单引号写的词—`You're`—不一样，这意味着针对前者的查询操作查不到后者，反之也一样。

索引，我们可以使用 `映射` 字符过滤器来解决这个问题，它允许我们用一种字符来替换另一种字符。比如现在，我们就可以把各式引号都替换成 `U+0027` 单引号：

```bash
PUT /my_index
{
  "settings": {
    "analysis": {
      "char_filter": { ①
        "quotes": {
          "type": "mapping",
          "mappings": [ ②
            "\\u0091=>\\u0027",
            "\\u0092=>\\u0027",
            "\\u2018=>\\u0027",
            "\\u2019=>\\u0027",
            "\\u201B=>\\u0027"
          ]
        }
      },
      "analyzer": {
        "quotes_analyzer": {
          "tokenizer":     "standard",
          "char_filter": [ "quotes" ] ③
        }
      }
    }
  }
}
```

① 我们定制了一个叫做 `quotes` 的 `char_filter`，它会把各种引号变体映射成最普通的单引号

② 为了表述清晰，我们使用了 JSON Unicode 转义语法来描述每个字符，但其实我们也可以直接输字符： `"‘=>'"`

③ 我们使用我们定制的 `quotes` 字符过滤器来创建了一个新的叫做 `quotest_analyzer` 解析器

然后我们测试一下：

```bash
GET /my_index/_analyze?analyzer=quotes_analyzer
You’re my ‘favorite’ M‛Coy
```

这个段示例代码会返回下面这些分词结果，所有的引号都被转换成了单引号：`You're`, `my`, `favorite`, `M'Coy`。

你越是花精力去优化你分词器要处理的文本的质量，那你的搜索结果也将会变得越好。

***

Tokenizers produce the best results when the input text is clean, *valid* text, where valid means that it follows the punctuation rules that the Unicode algorithm expects. Quite often, though, the text we need to process is anything but clean. Cleaning it up before tokenization improves the quality of the output.

## Tokenizing HTML

Passing HTML through the `standard` tokenizer or the `icu_tokenizer` produces poor results. These tokenizers just don’t know what to do with the HTML tags. For example:

```bash
GET /_analyzer?tokenizer=standard
<p>Some d&eacute;j&agrave; vu <a href="http://somedomain.com>">website</a>
```

The `standard` tokenizer confuses HTML tags and entities, and emits the following tokens: `p`, `Some`, `d`, `eacute`, `j`, `agrave`, `vu`, `a`, `href`, `http`, `somedomain.com`, `website`, `a`. Clearly not what was intended!

*Character filters* can be added to an analyzer to preprocess the text *before* it is passed to the tokenizer. In this case, we can use the `html_strip` character filter to remove HTML tags and to decode HTML entities such as `&eacute;` into the corresponding Unicode characters.

Character filters can be tested out via the `analyze` API by specifying them in the query string:

```bash
GET /_analyzer?tokenizer=standard&char_filters=html_strip
<p>Some d&eacute;j&agrave; vu <a href="http://somedomain.com>">website</a>
```

To use them as part of the analyzer, they should be added to a `custom` analyzer definition:

```bash
PUT /my_index
{
    "settings": {
        "analysis": {
            "analyzer": {
                "my_html_analyzer": {
                    "tokenizer":     "standard",
                    "char_filter": [ "html_strip" ]
                }
            }
        }
    }
}
```

Once created, our new `my_html_analyzer` can be tested with the `analyze` API:

```bash
GET /my_index/_analyzer?analyzer=my_html_analyzer
<p>Some d&eacute;j&agrave; vu <a href="http://somedomain.com>">website</a>
```

This emits the tokens that we expect: `Some`, `déjà`, `vu`, `website`.

## Tidying Up Punctuation

The `standard` tokenizer and `icu_tokenizer` both understand that an apostrophe *within* a word should be treated as part of the word, while single quotes that *surround* a word should not. Tokenizing the text `You're my 'favorite'`. would correctly emit the tokens `You're, my, favorite`.

Unfortunately, Unicode lists a few characters that are sometimes used as apostrophes:

`U+0027`
    Apostrophe (`'`)—the original ASCII character
    
`U+2018`
    Left single-quotation mark (`‘`)—opening quote when single-quoting
    
`U+2019`
    Right single-quotation mark (`’`)—closing quote when single-quoting, but also the preferred character to use as an apostrophe
    
Both tokenizers treat these three characters as an apostrophe (and thus as part of the word) when they appear within a word. Then there are another three apostrophe-like characters:

`U+201B`
    Single high-reversed-9 quotation mark (`‛`)—same as `U+2018` but differs in appearance
    
`U+0091`
    Left single-quotation mark in ISO-8859-1—should not be used in Unicode
    
`U+0092`
    Right single-quotation mark in ISO-8859-1—should not be used in Unicode
    
Both tokenizers treat these three characters as word boundaries—a place to break text into tokens. Unfortunately, some publishers use `U+201B` as a stylized way to write names like `M‛coy`, and the second two characters may well be produced by your word processor, depending on its age.

Even when using the “acceptable” quotation marks, a word written with a single right quotation mark—`You’re`—is not the same as the word written with an apostrophe—`You're`—which means that a query for one variant will not find the other.

Fortunately, it is possible to sort out this mess with the `mapping` character filter, which allows us to replace all instances of one character with another. In this case, we will replace all apostrophe variants with the simple `U+0027` apostrophe:

```bash
PUT /my_index
{
  "settings": {
    "analysis": {
      "char_filter": { ①
        "quotes": {
          "type": "mapping",
          "mappings": [ ②
            "\\u0091=>\\u0027",
            "\\u0092=>\\u0027",
            "\\u2018=>\\u0027",
            "\\u2019=>\\u0027",
            "\\u201B=>\\u0027"
          ]
        }
      },
      "analyzer": {
        "quotes_analyzer": {
          "tokenizer":     "standard",
          "char_filter": [ "quotes" ] ③
        }
      }
    }
  }
}
```

① We define a custom `char_filter` called `quotes` that maps all apostrophe variants to a simple apostrophe.

② For clarity, we have used the JSON Unicode escape syntax for each character, but we could just have used the characters themselves: `"‘=>'"`.

③ We use our custom `quotes` character filter to create a new analyzer called `quotes_analyzer`.

As always, we test the analyzer after creating it:

```bash
GET /my_index/_analyze?analyzer=quotes_analyzer
You’re my ‘favorite’ M‛Coy
```

This example returns the following tokens, with all of the in-word quotation marks replaced by apostrophes: `You're`, `my`, `favorite`, `M'Coy`.

The more effort that you put into ensuring that the tokenizer receives good-quality input, the better your search results will be.