# 5.2.3 Installing the ICU Plug-in 安装 ICU 插件
适用于 Elasticsearch 的 ICU 解析插件使用了 *International Components for Unicode*[^1] （ICU）类库（参见  [site.project.org](http://site.icu-project.org/)）以提供一系列丰富的工具集来处理 Unicode。这其中就包括了 `icu_tokenizer`(icu 分词器)，该分词器尤其适用于处理亚洲语系，还有许多标记过滤器，这些标记过滤器在对非英语的其他语言进行匹配操作和排序操作的时候都非常重要。

> **注意**
> 
> ICU 插件是一个非常重要的，处理非英语语言的工具，所以五星推荐推荐你安装并使用它。 ~~拒绝安利，从我做起~~ 但不幸的是，因为它是基于外部的一个叫做 ICU 的类库的，而不同版本的 ICU 插件又可能不兼容先前的版本。所以当你要升级的时候，可能需要重新索引你的数据。

要安装这个插件的话，首先要关闭你的 Elasticsearch 节点，然后在 Elasticsearch 的安装根目录下运行下面这段命令：

```bash
./bin/plugin -install elasticsearch/elasticsearch-analysis-icu/$VERSION ①
```

① 当前最新的 `$VERSION` 即版本号，可以在这个网站上查到 *[https://github.com/elasticsearch/elasticsearch-analysis-icu](https://github.com/elasticsearch/elasticsearch-analysis-icu)*.

一旦安装好之后，再启动 Elasticsearch 的时候，你就应该看到一行类似下面这样的启动日志：

```
[INFO][plugins] [Mysterio] loaded [marvel, analysis-icu], sites [marvel]
```

如果你正在使用多节点的集群，那你集群中的每个节点都要装一遍插件。 ~~crazy~~

[^1] [WIKIPEDIA : International Components for Unicode](https://en.wikipedia.org/wiki/International_Components_for_Unicode)

***

The [ICU analysis plug-in](https://github.com/elasticsearch/elasticsearch-analysis-icu) for Elasticsearch uses the *International Components for Unicode* (ICU) libraries (see [site.project.org](http://site.icu-project.org/)) to provide a rich set of tools for dealing with Unicode. These include the `icu_tokenizer`, which is particularly useful for Asian languages, and a number of token filters that are essential for correct matching and sorting in all languages other than English.

> **Note**
> 
> The ICU plug-in is an essential tool for dealing with languages other than English, and it is highly recommended that you install and use it. Unfortunately, because it is based on the external ICU libraries, different versions of the ICU plug-in may not be compatible with previous versions. When upgrading, you may need to reindex your data.

To install the plug-in, first shut down your Elasticsearch node and then run the following command from the Elasticsearch home directory:

./bin/plugin -install elasticsearch/elasticsearch-analysis-icu/$VERSION ①

① The current `$VERSION` can be found at *[https://github.com/elasticsearch/elasticsearch-analysis-icu](https://github.com/elasticsearch/elasticsearch-analysis-icu)*.

Once installed, restart Elasticsearch, and you should see a line similar to the following in the startup logs:

```
[INFO][plugins] [Mysterio] loaded [marvel, analysis-icu], sites [marvel]
```

If you are running a cluster with multiple nodes, you will need to install the plug-in on every node in the cluster.