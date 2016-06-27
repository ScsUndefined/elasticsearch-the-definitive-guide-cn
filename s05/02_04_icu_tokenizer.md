# 5.2.4 icu_tokenizer icu 分词器

`icu_tokenizer` 和 `标准` 分词器一样都使用 Unicode 文本切分算法，但它通过基于词典的方法来对一些亚洲语言提供了一个更好的支持，通过词典它可以更好地识别出泰文、老挝文、汉语、日语和韩文，并使用订制的规则来把缅甸语和高棉语拆分成音节。~~“波尔布特万岁，红色高棉万万岁!!!”刚说完就被人打死了...~~

找个泰语来举个例子吧，对于下 `icu` 分词器和 `标准` 分词器，看看它们切分完词之后各自的输出结果究竟有什么区别:

```bash
GET /_analyze?tokenizer=standard
สวัสดี ผมมาจากกรุงเทพฯ
```

~~这段泰文意思是“你好 我来自曼谷”~~

`标准` 分词器产生了 2 个标记，每个句子对应一个： `สวัสดี`, `ผมมาจากกรุงเทพฯ`。这种切分结果仅在你搜索“I am from Bangkok”（我来自曼谷）这一整个句子的时候才有用，而如果你想要搜“Bangkok”（曼谷）的时候就没啥卵用了。

```bash
GET /_analyze?tokenizer=icu_tokenizer
สวัสดี ผมมาจากกรุงเทพฯ
```

而 `icu` 分词器，能够把这段文本切分成独立的词（`สวัสดี`, `ผม`, `มา`, `จาก`, `กรุงเทพฯ`，即`你好`，`我`，`来`，`从`，`曼谷`），使得它们更容易被搜索到。

`标注`分词器相比于`icu`分词器还会过度地切分汉语和日语，经常会把完整的词汇不必要地切分成单独的字。因为汉语和日语中词和词之间是没有类似空格之类的分隔符的，所以很难判断是不是应该把连着的字拆分成独立的字还是认为它们是一个词。举个例子：


  * 向 意思是 *面向，朝着*, 日 意思是 *太阳*, 然后 葵 指的是 *葵类植物*。 当它们写在一起的时候， 向日葵 特指一种 *一年生的菊科向日葵属草本植物*.
  
  * 五 意思是 *五* 或者 *第五个*, 月 表示 *月亮*, 而 雨 则表示 *下雨*。 第一第二个字连在一起写作 五月 表示 *五月份*, 再加上第三字, 五月雨 表示 *梅雨* ~~五月雨貌似是日语里的~~ 当加上第四个字, 式, 意思是 *形式，模式*,的时候，就又组成了另一个词 五月雨式 一个新的形容词，表示连贯的或者冷酷的。 ~~一库~~
  
尽管一个词中的每个字都可以拆出来，但是维持它们原本的样子比把它们拆开要更有意义：

```bash
GET /_analyze?tokenizer=standard
向日葵

GET /_analyze?tokenizer=icu_tokenizer
向日葵
```

`标准` 分词器在处理上面这个例子的时候，结果为是 3 个标记：`向`，`日`，`葵`。而 `icu` 分词器的处理结果是一个单个的标记 `向日葵`。

`标准` 分词器和 `icu` 分词器的另一个不同之处在于后者会把不同书写体系的字符拆分开来（比如，`βeta`会被拆分为`β`和`eta`），而前者则会视它们会一个单个的标记`βeta`。

***

The `icu_tokenizer` uses the same Unicode Text Segmentation algorithm as the `standard` tokenizer, but adds better support for some Asian languages by using a dictionary-based approach to identify words in Thai, Lao, Chinese, Japanese, and Korean, and using custom rules to break Myanmar and Khmer text into syllables.

For instance, compare the tokens produced by the `standard` and `icu_tokenizers`, respectively, when tokenizing “Hello. I am from Bangkok.” in Thai:

```bash
GET /_analyze?tokenizer=standard
สวัสดี ผมมาจากกรุงเทพฯ
```

The `standard` tokenizer produces two tokens, one for each sentence: `สวัสดี`, `ผมมาจากกรุงเทพฯ`. That is useful only if you want to search for the whole sentence “I am from Bangkok.”, but not if you want to search for just “Bangkok.”

```bash
GET /_analyze?tokenizer=icu_tokenizer
สวัสดี ผมมาจากกรุงเทพฯ
```

The `icu_tokenizer`, on the other hand, is able to break up the text into the individual words (`สวัสดี`, `ผม`, `มา`, `จาก`, `กรุงเทพฯ`), making them easier to search.

In contrast, the `standard` tokenizer “over-tokenizes” Chinese and Japanese text, often breaking up whole words into single characters. Because there are no spaces between words, it can be difficult to tell whether consecutive characters are separate words or form a single word. For instance:

  * 向 means *facing*, 日 means *sun*, and 葵 means *hollyhock*. When written together, 向日葵 means *sunflower*.
  
  * 五 means *five* or *fifth*, 月 means month, and 雨 means *rain*. The first two characters written together as 五月 mean *the month of May*, and adding the third character, 五月雨 means *continuous rain*. When combined with a fourth character, 式, meaning *style*, the word 五月雨式 becomes an adjective for anything consecutive or unrelenting.
  
Although each character may be a word in its own right, tokens are more meaningful when they retain the bigger original concept instead of just the component parts:

```bash
GET /_analyze?tokenizer=standard
向日葵

GET /_analyze?tokenizer=icu_tokenizer
向日葵
```

The `standard` tokenizer in the preceding example would emit each character as a separate token: `向`, `日`, `葵`. The `icu_tokenizer` would emit the single token `向日葵` (sunflower).

Another difference between the `standard` tokenizer and the `icu_tokenizer` is that the latter will break a word containing characters written in different scripts (for example, `βeta`) into separate tokens—`β`, `eta`—while the former will emit the word as a single token: `βeta`.