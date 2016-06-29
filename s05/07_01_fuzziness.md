# 5.7.1 Fuzziness 怎么判断是否存在拼写错误

*模糊匹配* 会把两个“貌似”相似的词当做同一个词来对待，但首先我们得明确这个 *模糊性* 该怎么鉴定。

俄罗斯科学家弗拉基米尔·莱文斯坦在1965年提出了 [莱文斯坦距离](http://en.wikipedia.org/wiki/Levenshtein_distance) 这个概念，用来表示一个词转变到另一个词的时候需要修改多少个字符。他最初把修改操作归结为三种：

* *替换* : \_f\_ox → \_b\_ox
* *新增* : sic → sic\_k\_
* *删除* : b\_l\_ack → back

而 [Frederick Damerau](http://en.wikipedia.org/wiki/Frederick_J._Damerau) 又在此基础上新加了一种修改操作类型：

* *交换* : \_st\_ar → \_ts\_ar

基于这个理论， 如果要把 `bieber` 修改成 `beaver` 就需要执行下面这几步 :

1. 把 `b` 替换为 `v` : bie_b_er → bie_v_er
2. 把 `i` 替换为 `a` : b_i_ever → b_a_ever
3. 调换 `a` 和 `e` 的位置 : b_ae_ver → b_ea_ver

即 3 个 莱文斯坦编辑距离。

很明显，`bieber` 和 `beaver` 隔太远了 — 所以它们很难被看作是由于拼写错误而产生的一对其实意思是相同的词。Damerau 发现 80% 的拼写错误对应的 莱文斯坦编辑距离 都只有 1 。也即，80% 的拼写错误都只需要简单的 *一步* 操作就可以把它还原成它正确的样子。

你可以通过 `fuzziness` 参数来告诉 Elasticsearch 究竟多少个莱文斯坦距离内的词可以被认为是存在拼写错误的。现在我们假设这个值是 2。

那现在这个值是否合理，还得看实际被处理的词的长短。如果词的长度太短，比如 `hat`，经过两次修改之后就可以变成 `mad` 。所以很明显， 2 个编辑距离对于 3 个字母的单词而言就有点过了。所以你也可以把 `fuzziness` 的值设置成 `AUTO`，这会最终导致：

* `0` 对于 1~2 个字母组成的单词，不会考虑是否存在拼写错误的情况
* `1` 对于 3~5 个字母组成的单词，会考虑距离它们 1 个莱文斯坦编辑距离的其他字母组合
* `2` 而对于 5 个以上字母组成的单词，会考虑距离它们最多 2 个莱文斯坦编辑距离的其他字母组合

当然，实际应用过程中如果你发现 `2` 个莱文斯坦距离依然有点过，并且导致搜索结果的质量变低。那你就可以把 `fuzziness` 设为 `1`。

***

*Fuzzy matching* treats two words that are “fuzzily” similar as if they were the same word. First, we need to define what we mean by *fuzziness*.

In 1965, Vladimir Levenshtein developed the [Levenshtein distance](http://en.wikipedia.org/wiki/Levenshtein_distance), which measures the number of single-character edits required to transform one word into the other. He proposed three types of one-character edits:

* *Substitution* of one character for another: \_f\_ox → \_b\_ox
* *Insertion* of a new character: sic → sic\_k\_
* *Deletion* of a character:: b\_l\_ack → back

[Frederick Damerau](http://en.wikipedia.org/wiki/Frederick_J._Damerau) later expanded these operations to include one more:

* *Transposition* of two adjacent characters: \_st\_ar → \_ts\_ar

For example, to convert the word `bieber` into `beaver` requires the following steps:

1. Substitute `v` for `b`: bie_b_er → bie_v_er
2. Substitute `a` for `i`: b_i_ever → b_a_ever
3. Transpose `a` and `e`: b_ae_ver → b_ea_ver

These three steps represent a [Damerau-Levenshtein](https://en.wikipedia.org/wiki/Damerau%E2%80%93Levenshtein_distance) edit distance of 3.

Clearly, `bieber` is a long way from `beaver` — they are too far apart to be considered a simple misspelling. Damerau observed that 80% of human misspellings have an edit distance of 1. In other words, 80% of misspellings could be corrected with a *single edit* to the original string.

Elasticsearch supports a maximum edit distance, specified with the `fuzziness` parameter, of 2.

Of course, the impact that a single edit has on a string depends on the length of the string. Two edits to the word `hat` can produce `mad`, so allowing two edits on a string of length 3 is overkill. The `fuzziness` parameter can be set to `AUTO`, which results in the following maximum edit distances:

* `0` for strings of one or two characters
* `1` for strings of three, four, or five characters
* `2` for strings of more than five characters

Of course, you may find that an edit distance of `2` is still overkill, and returns results that don’t appear to be related. You may get better results, and better performance, with a maximum `fuzziness` of `1`.