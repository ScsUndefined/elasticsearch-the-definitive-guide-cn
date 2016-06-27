# 5.1 Getting Started with Languages 入门知识

Elasticsearch 搭载有一系列优良的，基础的，开箱即用的语言解析器，可以应对世界上大多数的通用语言：

阿拉伯语， 亚美尼亚语， 巴斯克语， 巴西语， 保加利亚语， 加泰隆语， 汉语， 捷克语， 丹麦文， 荷兰语， 英语， 芬兰语， 法语， 加里西亚语， 德语， 希腊语， 印度语， 匈牙利语， 印尼语， 爱尔兰语， 意大利语， 日文， 韩文， 库尔德语， 挪威语， 波斯语， 葡萄牙文， 罗马尼亚语， 俄文， 西班牙语， 瑞典语， 土耳其语，泰国语。

这些解析器通常做下面四件事：
  
  * 讲一段文本分解成各个独立的字词：
  
    `The quick brown foxes` → [`The`, `quick`, `brown`, `foxes`]

  * 小写化：

    `The` → `the`

  * 消除常用的停词：

    [`The`, `quick`, `brown`, `foxes`] → [`quick`, `brown`, `foxes`]

  * 提取词根：

    `foxes` → `fox`

每个解析器可能也会执行一些只针对该语言特性的，独有的格式化动作，使得该语言的信息变得更容易被搜索：

  * `英语` 解析器会移除所有格形式 `'s`：

    `John's` → `john`

  * `法语` 解析器会移除 *元音*，比如 `l'` 和 `qu'`，和 *变音音符*，比如 `¨` 或 `^`：

    `l'église` → `eglis`

  * `德语`解析器会进行针对德语语法的格式化操作，把`ä` 和 `ae` 格式化成 `a`，或者把`ß` 格式化为 `ss`，等等：
    
    `äußerst` → `ausserst`

***

Elasticsearch ships with a collection of language analyzers that provide good, basic, out-of-the-box support for many of the world’s most common languages:

Arabic, Armenian, Basque, Brazilian, Bulgarian, Catalan, Chinese, Czech, Danish, Dutch, English, Finnish, French, Galician, German, Greek, Hindi, Hungarian, Indonesian, Irish, Italian, Japanese, Korean, Kurdish, Norwegian, Persian, Portuguese, Romanian, Russian, Spanish, Swedish, Turkish, and Thai.

These analyzers typically perform four roles:

  * Tokenize text into individual words:

    `The quick brown foxes` → [`The`, `quick`, `brown`, `foxes`]

  * Lowercase tokens:

    `The` → `the`

  * Remove common stopwords:

    [`The`, `quick`, `brown`, `foxes`] → [`quick`, `brown`, `foxes`]

  * Stem tokens to their root form:

    `foxes` → `fox`

Each analyzer may also apply other transformations specific to its language in order to make words from that language more searchable:

  * The `english` analyzer removes the possessive `'s`:

    `John's` → `john`

  * The `french` analyzer removes *elisions* like `l'` and `qu'` and *diacritics* like `¨` or `^`:

    `l'église` → `eglis`

  * The `german` analyzer normalizes terms, replacing `ä` and `ae` with `a`, or `ß` with `ss`, among others:

    `äußerst` → `ausserst`