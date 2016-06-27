# 5.3 Normalizing Tokens 规范化标记

~~简述下分词和标记的关系： tokenizer 分词器会以 text 文本输入，进行 tokenization 分词操作，把一个完整 text 文本切分成多个 token 标记，输出一个标记流。 token 标记这个概念在中文里不知道怎么翻译才好，所以直译成标记了，怕不好理解，着重强调下。~~

把正文分好词之后，你当这就完事儿了？小伙贼，图样图森破，你这才搞定了一半而已。要使得分完词之后的标记变得更容易被搜索出来，它们就还需要经历一个叫做 *规范化* 的处理过程来移除相关单词之间的无关紧要的差异，比如大小写。有时候我们或许还需要移除一下有点意义的差异，来使得类似 `esta`, `ésta`, 和 `está` 可以作为同一个词被搜索出来。你肯定不愿意你在搜`déjà vu`的时候只搜出了 `déjà vu` 而没有 `deja vu`，对吧？

这就是标记过滤器干的事情了，它从分词器里接收一个标记流。你可以同时拥有多个标记过滤器，然后每个都做着各自特定的工作。每个标记过滤器都接收前一个标记过滤器的输出结果为自己的输入。

***

Breaking text into tokens is only half the job. To make those tokens more easily searchable, they need to go through a *normalization* process to remove insignificant differences between otherwise identical words, such as uppercase versus lowercase. Perhaps we also need to remove significant differences, to make `esta`, `ésta`, and `está` all searchable as the same word. Would you search for `déjà vu`, or just for `deja vu`?

This is the job of the token filters, which receive a stream of tokens from the tokenizer. You can have multiple token filters, each doing its particular job. Each receives the new token stream as output by the token filter before it.