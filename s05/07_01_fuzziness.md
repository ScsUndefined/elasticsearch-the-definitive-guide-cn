# 5.7.1 Fuzziness

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
