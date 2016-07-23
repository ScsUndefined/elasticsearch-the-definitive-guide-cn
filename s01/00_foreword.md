# 前言

直到现在，项目初创时的那段煎熬岁月对我而言依旧清晰可闻。当你发了第一个版本之后，并创建了一个网上聊天房，希望能快点有人来使用你的项目，并进入聊天房给你反馈，然而在这个人还没出现的时候，你会感到异常孤寂，愈发孤寂则愈发渴望，愈发渴望则更显寂寞。

当终于有人加入到聊天房的时候，我激动坏了，那人叫克林特。不过...聊了几句才发现他原来是个 Perl 程序员，对处理讣告网站做了不少开发工作。我还记得那时候我问自己，为啥不吸引不到一些主流语言的开发者呢？比如 Ruby 或者 Python(那个时候挺热门的~~现在国内也挺热门的诶~~)。

后来发现我还是图样图森破。Client 对 Elasticsearch 的成功有着莫大的贡献。他是第一个把 Elasticsearch 用于生成环境的用户（当时的版本号才 0.4）。在 Elascitsearch 项目的早期阶段，Client 提供了诸多反馈，Elasticsearch 也相应地做出诸多关键性改进，多亏了他，Elasticsearch 才能被打磨成如今这般模样。Client 对何为简洁，有着自己独到的看法，并且很少出错，他对 Elasticsearch 项目的各个层面都有着重大的影响，不管是管理（~~不知道这里的管理指的是开发人员对 Elasticsearch 项目的管控还是 Elasctisearch 对数据的管理~~），还是 API 的设计或者 day-to-day 的可用性功能。也因此当我们因为这个开源项目最终走到一起开办了一家公司之后，我们就立刻联系到了他，问他，要不一起干呗？~~我开始污了...(*/ω＼*)~~

另一件在我们开创了公司之后就立刻开始做的事情是提供公开的培训。真的是没法用文字来表达我们当时的紧张哦，就怕没有人愿意报名参加我们的培训。

结果还不是我们过虑咯。

年轻人，不是我跟你们吹哦，报名想要参加培训的人就跟香飘飘奶茶一样可以绕地球一圈！培训业务以前是现在依然我们开办的一个灰常成功的业务。然后我们还因此还揪准了一个叫做 Zach 的小伙子。我们通过他写的博客（介绍了如何使用 Elasticsearch，并且暗自羡慕他居然能把复杂的概念用浅显的话语准确地表述出来）以及他为 Elascitsearch 写的 PHP Client 来进一步了解了他。我们还发现 Zach 居然是自己掏腰包来报名参加我们的培训的，天了噜，人家居然愿意为你提供的培训服务自讨腰包，你还能再对这种用户再苛求什么？\~\~\~因此我们主动跟他进行了接触，问他是否愿意入伙。~~并递上了一块舒肤佳 (*/ω＼*)~~

~~原来公司的创办历程就跟邪教一样...好有趣~~

Client 和 Zach 都对 Elasticsearch 的成功起到了关键性的作用。而且他们都非常擅长沟通，既能把 Elasticseach 顶层的抽象简化过的地方解释清楚，也能把 Elasticsearch 或者 Apache Lucene 的底层内部的复杂点都解释清楚。~~所以说沟通能力还是很重要的，这一点我得好好学习，感觉自己有时候不够谦虚，发现自己稍微比别人多懂一点某方面的东西就沾沾自喜，看不起别人，哎...感觉尊重别人才是正常沟通的基础，呃 扯远了~~现在 Clint 负责了 Elasticsearch Perl Client 的开发，而 Zach 则负责了 PHP Client 的开发。两个项目的代码都非常优质。

最后，他俩对 Elasticsearch 项目的日常进展都起到一个辅助的作用。之所以 Elsticsearch 现如今如此得受欢迎，一个主要的原因就是项目开发人员能够与项目的使用者同心同德，而 Clint 和 Zach 都对一点做出了不朽的贡献。

*Shay Banon*

~~哎，相比之下，就让我想起了 Apache 一直要退出 JCP 执行委员会，理由就是因为觉得委员会只顾及大公司的利益而枉顾广大 Java 开源社区的利益，而现在也疯传 Orcale 输了与 Google 的官司之后就放缓了 Java 版本迭代工作，感觉一点都不与广大的Java 开发人员同心同德，so sad~~

***

One of the most nerve-wracking periods when releasing the first version of an open source project occurs when the IRC channel is created. You are all alone, eagerly hoping and wishing for the first user to come along. I still vividly remember those days.

One of the first users that jumped on IRC was Clint, and how excited was I. Well… for a brief period, until I found out that Clint was actually a Perl user, no less working on a website that dealt with obituaries. I remember asking myself why couldn’t we get someone from a more "hyped" community, like Ruby or Python (at the time), and a slightly nicer use case.

How wrong I was. Clint ended up being instrumental to the success of Elasticsearch. He was the first user to roll out Elasticsearch into production (version 0.4 no less!), and the interaction with Clint was pivotal during the early days in shaping Elasticsearch into what it is today. Clint has a unique insight into what is simple, and he is very rarely wrong, which has a huge impact on various usability aspects of Elasticsearch, from management, to API design, to day-to-day usability features. It was a no brainer for us to reach out to Clint and ask if he would join our company immediately after we formed it.

Another one of the first things we did when we formed the company was offer public training. It’s hard to express how nervous we were about whether or not people would even sign up for it.

We were wrong.

The trainings were and still are a rave success with waiting lists in all major cities. One of the people who caught our eye was a young fellow, Zach, who came to one of our trainings. We knew about Zach from his blog posts about using Elasticsearch (and secretly envied his ability to explain complex concepts in a very simple manner) and from a PHP client he wrote for the software. What we found out was that Zach had actually paid to attend the Elasticsearch training out of his own pocket! You can’t really ask for more than that, and we reached out to Zach and asked if he would join our company as well.

Both Clint and Zach are pivotal to the success of Elasticsearch. They are wonderful communicators who can explain Elasticsearch from its high-level simplicity, to its (and Apache Lucene’s) low-level internal complexities. It’s a unique skill that we dearly cherish here at Elastic. Clint is also responsible for the Elasticsearch Perl client, while Zach is responsible for the PHP one - both wonderful pieces of code.

And last, both play an instrumental role in most of what happens daily with the Elasticsearch project itself. One of the main reasons why Elasticsearch is so popular is its ability to communicate empathy to its users, and Clint and Zach are both part of the group that makes this a reality.

*Shay Banon*
