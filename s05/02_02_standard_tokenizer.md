# 5.2.2 standard Tokenizer 标准分词器

所谓 `分词器`，就是接收一串文字，把它拆分成单个的字词或标记（或许丢弃了一些字符，比如标点符号什么的），然后输出一个 *标记流*。

最让人觉得有意思的就是 *识别* 字词的算法。`空格`分词器只是简单地在空格符，制表符，换行符等等的地方做切分操作 — 即假定几个相连的字母为一个单独的单词。比如：

```bash
GET /_analyze?tokenizer=whitespace
You're the 1st runner home!
```

这个请求会返回这样子的几个词：`You're`, `the`, `1st`, `runner`, `home!`

而 `字母` 分词器，则会在任何不是字母的地方做切分，那上面这段句子就会被切分为：`You`, `re`, `the`, `st`, `runner`, `home`

`标准` 分词器使用 Unicode 切分文本算法（在 [Unicode Standard Annex #29](http://unicode.org/reports/tr29/) 中被规定）来找出单词 *之间* 的边界，然后把边界中的内容都输出出来。它对于 Unicode 的支持能力使得它能成功地切分多种语言混合书写的文本。

标点符号有时候会，也有时候不会被认为是单词的一部分，这取决于单词出现在什么地方：

```bash
GET /_analyze?tokenizer=standard
You're my 'favorite'.
```

在这个示例中，`You're` 中的上撇号会被认为是词中的一部分，而 `'favourite'` 中的单引号则不会，所以最后切分完之后的词就会是这样：`You're`, `my`, `favorite`

> **提示**
>
> `uax_url_email` 分词器和 `标准` 分词器工作方式是一样的，除了它会把 email 地址和 URL 会当做单个的标记输出出来。而 `标准` 分词器则会尝试着把它们再细分成单个的词。比如 `joe-bloggs@foo-bar.com` 就会被切分成 `joe`, `bloggs`, `foo`, `bar.com`。

`标准`分词器是处理大多数语言的一个恰当的起始点，尤其是西方语言。它事实上也的确是大多数语言解析器的基石，比如 `英语`， `法语` 以及 `西班牙语` 解析器。它也支持亚洲语系，但颇有些局限性，你与其使用标准分词器，不如考虑换用 `icu_tokenizer`，这个分词器可以安装 ICU 插件来获得。

***

A *tokenizer* accepts a string as input, processes the string to break it into individual words, or tokens (perhaps discarding some characters like punctuation), and emits a *token stream* as output.

What is interesting is the algorithm that is used to *identify* words. The `whitespace` tokenizer simply breaks on whitespace—spaces, tabs, line feeds, and so forth—and assumes that contiguous nonwhitespace characters form a single token. For instance:

```bash
GET /_analyze?tokenizer=whitespace
You're the 1st runner home!
```

This request would return the following terms: `You're`, `the`, `1st`, `runner`, `home`!

The `letter` tokenizer, on the other hand, breaks on any character that is not a letter, and so would return the following terms: `You`, `re`, `the`, `st`, `runner`, `home`.

The `standard` tokenizer uses the Unicode Text Segmentation algorithm (as defined in [Unicode Standard Annex #29](http://unicode.org/reports/tr29/)) to find the boundaries *between* words, and emits everything in-between. Its knowledge of Unicode allows it to successfully tokenize text containing a mixture of languages.

Punctuation may or may not be considered part of a word, depending on where it appears:

```bash
GET /_analyze?tokenizer=standard
You're my 'favorite'.
```

In this example, the apostrophe in `You're` is treated as part of the word, while the single quotes in `'favorite'` are not, resulting in the following terms: `You're`, `my`, `favorite`.

> **Tip**
> 
> The `uax_url_email` tokenizer works in exactly the same way as the `standard` tokenizer, except that it recognizes email addresses and URLs and emits them as single tokens. The `standard` tokenizer, on the other hand, would try to break them into individual words. For instance, the email address `joe-bloggs@foo-bar.com` would result in the tokens `joe`, `bloggs`, `foo`, `bar.com`.

The `standard` tokenizer is a reasonable starting point for tokenizing most languages, especially Western languages. In fact, it forms the basis of most of the language-specific analyzers like the `english`, `french`, and `spanish` analyzers. Its support for Asian languages, however, is limited, and you should consider using the `icu_tokenizer` instead, which is available in the ICU plug-in.