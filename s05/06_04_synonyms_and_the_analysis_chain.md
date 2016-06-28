# 5.6.4 Synonyms and The Analysis Chain

在 [Formatting Synonyms](https://www.elastic.co/guide/en/elasticsearch/guide/current/synonym-formats.html) 一章中，我们使用 `u s a`　来举例阐述一些同义词相关的知识。那为什么我们没有使用 `U.S.A.`呢？这其实是有原因的，那就是因为 `同义词`　标记过滤器只能接收到在它前面的标记过滤器或者分词器的输出结果，而看不到最原始的输入文本。

设想一下，我们有一个，由一个 `标注` 分词器以及一个`小写化`　标记过滤器后跟一个 `同义词` 标记过滤器组成的，解析器。这个解析器如果处理 `U.S.A.` 那过程就会像下面这样：

```
最原始的输入文本             → "U.S.A."
标准             分词器     → (U),(S),(A)
小写化           标记过滤器  → (u),(s),(a)
同义词           标记过滤器  → (usa)
```

如果我们设定了一个同义词规则，然后规则的右边是　`U.S.A.`，不管用什么关键字进行查询都出不来结果了，因为当 `my_synonym_filter` （自定义的同义词过滤器）看到标记的时候，句号已经被移除了，并且字母也已经被小写化了。

这其实是一个非常需要注意的地方。如果我们想同时使用同义词特性与词根提取特性，那是否　`jumps`, `jumped`, `jump`, `leaps`, `leaped`, 和 `leap` 这些词都会被索引成一个 `jump` ？我们或许可以把同义词过滤器防止到词根提取过滤器前面，然后把所有同义词以及变体都列举出来：

"jumps,jumped,leap,leaps,leaped => jump"

但更简洁的做法是把同义词过滤器放到词根化过滤器的后面，然后只要把词根形式的同义词列举出来：

"leap => jump"

## 大小写敏感的同义词

通常情况下，`同义词` 标记过滤器都会放置在 `小写化` 标记过滤器的后面，也因此同义词规则里的所有词都是以小写格式来指定的，但有时候这个做法也会一些奇怪的合并操作。比如，`CAT`扫描 ~~也就是医院里的CT扫描~~ 和 `cat` 意思是不同的，同样的，`PET`（正电子发射型断层显像技术）和 `pet` 意思也不一样。还有，用于表示姓名的 `Little` 和形容词性的 `little` 意思（尽管当一个句子以它开头时，首字母会被大写，然后看上去就和 `Little` 一样了）也又不同。

如果你需要借助大小写来甄别字词的意思，那你需要把你的同义词过滤器放置在在 `小写化` 过滤器之前了。当然这也就以为着你的同义词规则需要精准地列出你想要匹配的词的每种变体（比如，`Little,LITTLE,little`）。

除了上述做法，你也可以有两个同义词过滤器，一个匹配大小写敏感的同义词一个匹配大小写不敏感的同义词。举例来说，大小写敏感的同义词规则大概就像下面这样：

```
"CAT,CAT scan           => cat_scan"
"PET,PET scan           => pet_scan"
"Johnny Little,J Little => johnny_little"
"Johnny Small,J Small   => johnny_small"
```

然后大小不敏感的同义词规则大概就像：

```
"cat                    => cat,pet"
"dog                    => dog,pet"
"cat scan,cat_scan scan => cat_scan"
"pet scan,pet_scan scan => pet_scan"
"little,small"
```

大小写敏感的同义词规则不仅会处理 `CAT scan`，而且有时候也可能会匹配到 `CAT scan` 中的 `CAT` 从而导致 `CAT scan` 被转化成了同义词 `cat_scan scan` ~~这大概取决于分词是否准确吧~~。因此在大小写不敏感的同义词规则里，我们有一条看上去怪怪的同义词规则 `cat_scan scan`，就是用来修正大小写敏感的同义词过滤器产生的坏的替换结果的。

> **提示**
> 
> 你马上就会看到它将变得多么复杂了 ~~我开始方了~~。和以往一样，analyze API 仍旧是你检测是的解析器是否配置正确的好帮手，具体参阅 [Testing Analyzers](https://www.elastic.co/guide/en/elasticsearch/guide/current/analysis-intro.html#analyze-api).

***

The example we showed in [Formatting Synonyms](https://www.elastic.co/guide/en/elasticsearch/guide/current/synonym-formats.html), used `u s a` as a synonym. Why did we use that instead of `U.S.A.`? The reason is that the `synonym` token filter sees only the terms that the previous token filter or tokenizer has emitted.

Imagine that we have an analyzer that consists of the `standard` tokenizer, with the `lowercase` token filter followed by a `synonym` token filter. The analysis process for the text `U.S.A.` would look like this:

```
original string                  → "U.S.A."
standard           tokenizer     → (U),(S),(A)
lowercase          token filter  → (u),(s),(a)
synonym            token filter  → (usa)
```

If we had specified the synonym as `U.S.A.`, it would never match anything because, by the time `my_synonym_filter` sees the terms, the periods have been removed and the letters have been lowercased.

This is an important point to consider. What if we want to combine synonyms with stemming, so that `jumps`, `jumped`, `jump`, `leaps`, `leaped`, and `leap` are all indexed as the single term `jump`? We could place the synonyms filter before the stemmer and list all inflections:

"jumps,jumped,leap,leaps,leaped => jump"

But the more concise way would be to place the synonyms filter after the stemmer, and to list just the root words that would be emitted by the stemmer:

"leap => jump"

## Case-Sensitive Synonyms

Normally, synonym filters are placed after the `lowercase` token filter and so all synonyms are written in lowercase, but sometimes that can lead to odd conflations. For instance, a `CAT` scan and a `cat` are quite different, as are `PET` (positron emission tomography) and a `pet`. For that matter, the surname `Little` is distinct from the adjective `little` (although if a sentence starts with the adjective, it will be uppercased anyway).

If you need use case to distinguish between word senses, you will need to place your synonym filter before the `lowercase` filter. Of course, that means that your synonym rules would need to list all of the case variations that you want to match (for example, `Little,LITTLE,little`).

Instead of that, you could have two synonym filters: one to catch the case-sensitive synonyms and one for all the case-insensitive synonyms. For instance, the case-sensitive rules could look like this:

```
"CAT,CAT scan           => cat_scan"
"PET,PET scan           => pet_scan"
"Johnny Little,J Little => johnny_little"
"Johnny Small,J Small   => johnny_small"
```

And the case-insensitive rules could look like this:

```
"cat                    => cat,pet"
"dog                    => dog,pet"
"cat scan,cat_scan scan => cat_scan"
"pet scan,pet_scan scan => pet_scan"
"little,small"
```

The case-sensitive rules would `CAT scan` but would match only the `CAT` in `CAT scan`. For this reason, we have the odd-looking rule `cat_scan scan` in the case-insensitive list to catch bad replacements.

> **Tip**
> 
> You can see how quickly it can get complicated. As always, the analyze API is your friend—use it to check that your analyzers are configured correctly. See [Testing Analyzers](https://www.elastic.co/guide/en/elasticsearch/guide/current/analysis-intro.html#analyze-api).
