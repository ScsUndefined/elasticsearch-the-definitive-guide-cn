# 5.1.3 Pitfalls of Mixing Languages 多语言的坑

如果你只需要处理一种语言，那你赶紧阿弥陀佛吧。要知道，要找到一个正确的策略来处理一个用多门语言书写而成的文档将会是一个相当有挑战性的事情。

## 在入索引的阶段

所谓多语言的文档大致分为下面三种：

  * 单个文档对应一门主语言，然后穿插了部分其他语言书写的内容。（参见 [One Language per Document](https://www.elastic.co/guide/en/elasticsearch/guide/current/one-lang-docs.html)）

  * 单个字段对应一门主语言，然后穿插了部分其他语言书写的内容。（参见 [One Language per Field](https://www.elastic.co/guide/en/elasticsearch/guide/current/one-lang-fields.html)）
  
  * 每个字段都是由多个语言混合编写而成。（参见 [Mixed-Language Fields](https://www.elastic.co/guide/en/elasticsearch/guide/current/mixed-lang-fields.html)）

那我们的目标是，尽管不是总能达到这个目标，应该分开处理各个语言书写的内容。在同一个倒排索引中混合多个语言会有问题。

### 不能正确地提取词根

德语的提取词根的规则有别于英语，法语，瑞典语等等。把同一个词根提取规则应用到不同的语言中会导致有些词被正确地提取了词根，有些则不正确，甚至有些词压根就没有被提取词根。这甚至可能导致不同语言中表示不同含义的单词提取词根后，变成了同一个单词，致使混淆了它们的真正意思，并给用户造成了一个混乱不清的搜索结果。 ~~阔怕，我开始方了 [╯□╰]~~ 

把多种词根提取规则应用在同一个文本上会导致最后的结果变成一堆垃圾，因为第二个词根提取动作会基于上一次的词根提取结果，这可能会导致有些词已经处于词根形式，结果又被提取了一次，问题反而更复杂了。

> 多书写体
>
> 只允许同时使用一种词根提取法的原则也会遇上意外情况，当多门语言隶属于不同书写体系的时候，那这个原则被打破也没有问题。举个例子，在以色列，有可能一个文档中包含了 希伯来语， 阿拉伯语， 俄文（斯拉夫语系），以及英文：
> 
> אזהרה - Предупреждение - تحذير - Warning
> 
> 这四门语言分别属于四种不同书写体系，所以即使为这段文本设置多个词根提取规则也没有关系，因为针对不同书写体的词根提取规则并不会互相影响。

### 无法恰当地计算字词的相关性

在 [什么是相关性？ （What Is Relevance?）](https://www.elastic.co/guide/en/elasticsearch/guide/current/relevance-intro.html)一文中我们解释了当一个词在一系列文档中出现得越频繁，那它的权重就越低。所以为了准确地计算相关性，我们首先需要有准确的各个词的出现频率。

如果一段文字，主要是英文，中间又穿插了一丢丢德文，那如果把这段文字放到一群英文文档中计算词频，那么就会导致德文字词的出现频率变低，从而增加它们的权重，但如果把这段文字放到一群德文文档中计算词频，那这些德文字词的权重就会相对变得低得多。

## 在检索阶段

在多语言的业务场景下，仅仅关注你的文档还远远不够，你还需要考虑你的用户会怎么检索这些文档。通常情况下，你可以从 2 种渠道来得知用户的主语言，通过用户选择的交互界面（比如你的网站同时有德文版和法语版，网址域名分别以`.de`和`.fr`结尾，通过分析用户选择的网站语言版本就可以得知用户的主语言是什么），再或者可以通过用户浏览器的 HTTP header 中的 [accept-language](http://www.w3.org/International/questions/qa-lang-priorities.en.php) 中得知 ~~谷妹真心不容易...~~。

用户的查询操作也大致可以分为下面三类：

  * 查询自己主语言的结果（比如输入“耐克”来搜索当季的耐克在中国大陆上市的新品。在中文环境下，输入的是中文，希望得到的结果也是中文）
  
  * 用其他语言来查询自己主语言的结果（比如输入“nike”来搜索当季的耐克在中国大陆上市的新品。在中文环境下，输入的是英文，但是希望得到的结果是中文）
  
  * 用其他语言来查询该语言的结果（比如输入“nike”来搜索当季的耐克在美国本土上市的新品，这通常是一个双语言的人才会干的事，比如香港同胞，或者一个网咖里的歪果仁，来自米国本土，却出差到了上海，在上海的一家网咖里上网。在中文环境下，输入的是英文，希望得到的查询结果也是英文）
 
分析用户的查询动作，然后只返回一种语言的搜索结果的做法是比较合适的（比如用户是通过西班牙版网站来查询商品的，那你分析出用户访问的是西班牙语网站，然后只返回西班牙语版的产品信息即可），或者你能识别出，虽然用户访问的是西班牙语网站，但是它的主语言是其他语言，那你就只返回该用户的主语言版本的产品信息。

查询结果的语言版本一般都更倾向于采用用户所使用的语言。比如一个说英语的，即使他在网站上输入一个法语词“deja vu”，但他可能也更希望搜索结果是英文的维基百科页面而不是法语版的维基页面。 ~~然而我大天朝只能访问英文版维基百科，而打不开中文版，sad！凭啥香港银就可以？凭啥台湾银就可以？恩，他们是异端，我们应该把他们踢出我大天朝~~

~~法语词，deja vu，即幻觉记忆（也译为既视感，即视感，自法语“Déjà vu”或“paramnésie”，译名“即视感”直译自和制汉语），指人在清醒的状态下第一次见到某场景，却感到“似曾相识”，是一种常见于大多数人的生理现象。~~

~~有人把这种现象当作灵魂漫游或前世记忆的证明。脑科学界普遍认为这是因为记忆的存储出现了短暂的混乱，导致大脑把刚刚得到的信息当成了久远的回忆。所以这种情况多半是在人们感到疲倦、压力，或是被不熟悉事物环绕的情况下出现，因为此时大脑无法一一处理接收来的资讯量。相较于老年人，年轻族群比较容易出现“幻觉记忆”；一来是年轻人的行程比较飘忽不定，周遭常出现陌生的东西，二来则是年轻人的生活较为忙碌，大脑常常会“打结”。~~

~~每次看到漂亮妹纸我就觉得仿佛与她似曾相识，原来只是自己一厢情愿的幻觉么？...~~

## 识别语言

有时候，你可能已经知道你的文档的语言版本了。比如你的文档可能是你们单位自己新建的，或者被翻译成指定语言的。类似这种通过人工方法预先得知文档的语言版本，大概是最可靠的识别文档语言的方法了。

但有时候，你的文档可能来自于系统外部，可能是各种各样的语言，或者你的文档的语言版本被弄错了，比如一个英文文档，被标记成了中文文档。这类似这样的情况下，你就需要一个启发式的方法来识别文档的主语言。听了之后是不是神烦？被吓呆了？小伙贼，不要方，你已经不需要重复造轮了，已经有许多针对各类语言的类库能帮你解决这类问题了。

在这里，我不得不安利下 Google 的 [chromium-compact-language-detector](https://github.com/mikemccand/chromium-compact-language-detector) 庫，来自于  [Mike McCandless](http://blog.mikemccandless.com/2013/08/a-new-version-of-compact-language.html)，以([Apache License 2.0](http://www.apache.org/licenses/LICENSE-2.0)) [Compact Language Detector](https://code.google.com/p/cld2/) (CLD) 许可证的形式开源。它短小精悍，快准恨，有时只需要两个句子就可以从 160 多种语言中判断出这主要是由什么语言书写的。它甚至可以探测出一段文本使用到的多门语言。支持多种编程语言，包括 Python, Perl, JavaScript, PHP, C#/.NET, 和 R。

要从用户的查询请求中识别出用户的语言可不简单哦。CLD 是被设计成基于至少 200 个字符的语言探测器。文本长度越短，比如查询的关键字，就越容易产生错误的结果。因此除了搜索文本，最好综合考虑其他一些信息来启发式地判断用户的语言，比如查询请求的来源地信息，用户选择的语言信息，以及 HTTP header 中的 `accept-language`。

***

If you have to deal with only a single language, count yourself lucky. Finding the right strategy for handling documents written in several languages can be challenging.

## At Index Time

Multilingual documents come in three main varieties:

  * One predominant language per document, which may contain snippets from other languages (See [One Language per Document](https://www.elastic.co/guide/en/elasticsearch/guide/current/one-lang-docs.html).)
  
  * One predominant language per field, which may contain snippets from other languages (See [One Language per Field](https://www.elastic.co/guide/en/elasticsearch/guide/current/one-lang-fields.html).)
  
  * A mixture of languages per field (See [Mixed-Language Fields](https://www.elastic.co/guide/en/elasticsearch/guide/current/mixed-lang-fields.html).)

The goal, although not always achievable, should be to keep languages separate. Mixing languages in the same inverted index can be problematic.

### Incorrect stemming

The stemming rules for German are different from those for English, French, Swedish, and so on. Applying the same stemming rules to different languages will result in some words being stemmed correctly, some incorrectly, and some not being stemmed at all. It may even result in words from different languages with different meanings being stemmed to the same root word, conflating their meanings and producing confusing search results for the user.

Applying multiple stemmers in turn to the same text is likely to result in rubbish, as the next stemmer may try to stem an already stemmed word, compounding the problem.

> Stemmer per Script
>
> The one exception to the only-one-stemmer rule occurs when each language is written in a different script. For instance, in Israel it is quite possible that a single document may contain Hebrew, Arabic, Russian (Cyrillic), and English:
> 
> אזהרה - Предупреждение - تحذير - Warning
> 
> Each language uses a different script, so the stemmer for one language will not interfere with another, allowing multiple stemmers to be applied to the same text.

### Incorrect inverse document frequencies

In [What Is Relevance?](https://www.elastic.co/guide/en/elasticsearch/guide/current/relevance-intro.html), we explained that the more frequently a term appears in a collection of documents, the less weight that term has. For accurate relevance calculations, you need accurate term-frequency statistics.

A short snippet of German appearing in predominantly English text would give more weight to the German words, given that they are relatively uncommon. But mix those with documents that are predominantly German, and the short German snippets now have much less weight.

## At Query Time

It is not sufficient just to think about your documents, though. You also need to think about how your users will query those documents. Often you will be able to identify the main language of the user either from the language of that user’s chosen interface (for example, `mysite.de` versus `mysite.fr`) or from the [accept-language](http://www.w3.org/International/questions/qa-lang-priorities.en.php) HTTP header from the user’s browser.

User searches also come in three main varieties:

  * Users search for words in their main language.
  
  * Users search for words in a different language, but expect results in their main language.
  
  * Users search for words in a different language, and expect results in that language (for example, a bilingual person, or a foreign visitor in a web cafe).

Depending on the type of data that you are searching, it may be appropriate to return results in a single language (for example, a user searching for products on the Spanish version of the website) or to combine results in the identified main language of the user with results from other languages.

Usually, it makes sense to give preference to the user’s language. An English-speaking user searching the Web for “deja vu” would probably prefer to see the English Wikipedia page rather than the French Wikipedia page.

## Identifying Language

You may already know the language of your documents. Perhaps your documents are created within your organization and translated into a list of predefined languages. Human pre-identification is probably the most reliable method of classifying language correctly.

Perhaps, though, your documents come from an external source without any language classification, or possibly with incorrect classification. In these cases, you need to use a heuristic to identify the predominant language. Fortunately, libraries are available in several languages to help with this problem.

Of particular note is the [chromium-compact-language-detector](https://github.com/mikemccand/chromium-compact-language-detector) library from [Mike McCandless](http://blog.mikemccandless.com/2013/08/a-new-version-of-compact-language.html), which uses the open source ([Apache License 2.0](http://www.apache.org/licenses/LICENSE-2.0)) [Compact Language Detector](https://code.google.com/p/cld2/) (CLD) from Google. It is small, fast, and accurate, and can detect 160+ languages from as little as two sentences. It can even detect multiple languages within a single block of text. Bindings exist for several languages including Python, Perl, JavaScript, PHP, C#/.NET, and R.

Identifying the language of the user’s search request is not quite as simple. The CLD is designed for text that is at least 200 characters in length. Shorter amounts of text, such as search keywords, produce much less accurate results. In these cases, it may be preferable to take simple heuristics into account such as the country of origin, the user’s selected language, and the HTTP `accept-language` headers.