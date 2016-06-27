# 5.6.2 Formatting Synonyms 同义词规则的书写格式

同义词规则的最简单书写格式，就是像下面这样简单地用逗号隔开：

"jump,leap,hop"

像这样子定义好之后，只要遇到上面这三个词中的任意一个，它就会被替换成所有被列举出来的同义词。就像这样：

```
原始词:            被替换成:
────────────────────────────────
jump            → (jump,leap,hop)
leap            → (jump,leap,hop)
hop             → (jump,leap,hop)
```

还有中写法，就是用 `=>` 符号，它可以指明单向型的同义词规则，即左边为目标词汇列表，右边为目标词汇对应的同义词列表：

"u s a,united states,united states of america => usa"
"g b,gb,great britain => britain,england,scotland,wales"

```
原始词:            被替换成:
────────────────────────────────
u s a           → (usa)
united states   → (usa)
great britain   → (britain,england,scotland,wales)
```

~~所以 (jump, leap, hop) 也即 jump, leap, hop => jump, leap, hop~~

如果对于同义词指定了多种同义词规则，那么它们就会被自动合并。它们的指定顺序对于 Elasticsearch 来说是没有意义的，但是如果它们存在冲突，那么 Elasticsearch 就会优先采用长的同义词匹配规则。我们用下面这个例子来说明一下：

"united states            => usa",
"united states of america => usa"

如果这些规则冲突了，那么 Elasticsearch 不会把 `United States of America` 转换成 `(usa),(of),(america)`而是会返回`(usa)`。

***

In their simplest form, synonyms are listed as comma-separated values:

"jump,leap,hop"
If any of these terms is encountered, it is replaced by all of the listed synonyms. For instance:

```
Original terms:   Replaced by:
────────────────────────────────
jump            → (jump,leap,hop)
leap            → (jump,leap,hop)
hop             → (jump,leap,hop)
```

Alternatively, with the `=>` syntax, it is possible to specify a list of terms to match (on the left side), and a list of one or more replacements (on the right side):

"u s a,united states,united states of america => usa"
"g b,gb,great britain => britain,england,scotland,wales"

```
Original terms:   Replaced by:
────────────────────────────────
u s a           → (usa)
united states   → (usa)
great britain   → (britain,england,scotland,wales)
```

If multiple rules for the same synonyms are specified, they are merged together. The order of rules is not respected. Instead, the longest matching rule wins. Take the following rules as an example:

"united states            => usa",
"united states of america => usa"
If these rules conflicted, Elasticsearch would turn `United States of America` into the terms `(usa),(of),(america)`. Instead, the longest sequence wins, and we end up with just the term `(usa)`.
