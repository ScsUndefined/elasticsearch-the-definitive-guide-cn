# Summary

* [Introduction](README.md)
* [1 Foreword](s01/00_foreword.md)
* [2 Preface](s02/00_preface.md)
   * 2.1 Who Should Read This Book
   * 2.2 Why We Wrote This Book
   * 2.3 Elasticsearch Version
   * 2.4 How to Read This Book
   * 2.5 Navigating This Book
   * 2.6 Online Resources
   * 2.7 Conventions Used in This Book
   * 2.8 Using Code Examples
   * 2.9 Acknowledgments
* [3 Getting Started](s03/00_getting_started.md)
   * 3.1 You Know, for Search…
       * 3.1.1 Installing and Running Elasticsearch
       * 3.1.2 Talking to Elasticsearch
       * 3.1.3 Document Oriented
       * 3.1.4 Finding Your Feet
       * 3.1.5 Indexing Employee Documents
       * 3.1.6 Retrieving a Document
       * 3.1.7 Search Lite
       * 3.1.8 Search with Query DSL
       * 3.1.9 More-Complicated Searches
       * 3.1.10 Full-Text Search
       * 3.1.11 Phrase Search
       * 3.1.12 Highlighting Our Searches
       * 3.1.13 Analytics
       * 3.1.14 Tutorial Conclusion
       * 3.1.15 Distributed Nature
       * 3.1.16 Next Steps
   * 3.2 Life Inside a Cluster
       * 3.2.1 An Empty Cluster
       * 3.2.2 Cluster Health
       * 3.2.3 Add an Index
       * 3.2.4 Add Failover
       * 3.2.5 Scale Horizontally
       * 3.2.6 Coping with Failure
   * 3.3 Data In, Data Out
       * 3.3.1 What Is a Document?
       * 3.3.2 Document Metadata
       * 3.3.3 Indexing a Document
       * 3.3.4 Retrieving a Document
       * 3.3.5 Checking Whether a Document Exists
       * 3.3.6 Updating a Whole Document
       * 3.3.7 Creating a New Document
       * 3.3.8 Deleting a Document
       * 3.3.9 Dealing with Conflicts
       * 3.3.10 Optimistic Concurrency Control
       * 3.3.11 Partial Updates to Documents
       * 3.3.12 Retrieving Multiple Documents
       * 3.3.13 Cheaper in Bulk
   * 3.4 Distributed Document Store
       * 3.4.1 Routing a Document to a Shard
       * 3.4.2 How Primary and Replica Shards Interact
       * 3.4.3 Creating, Indexing, and Deleting a Document
       * 3.4.4 Retrieving a Document
       * 3.4.5 Partial Updates to a Document
       * 3.4.6 Multidocument Patterns
   * 3.5 Searching—The Basic Tools
       * 3.5.1 The Empty Search
       * 3.5.2 Multi-index, Multitype
       * 3.5.3 Pagination
       * 3.5.4 Search Lite
   * 3.6 Mapping and Analysis
       * 3.6.1 Exact Values Versus Full Text
       * 3.6.2 Inverted Index
       * 3.6.3 Analysis and Analyzers
       * 3.6.4 Mapping
       * 3.6.5 Complex Core Field Types
   * 3.7 Full-Body Search
       * 3.7.1 Empty Search
       * 3.7.2 Query DSL
       * 3.7.3 Queries and Filters
       * 3.7.4 Most Important Queries
       * 3.7.5 Combining queries together
       * 3.7.6 Validating Queries
   * 3.8 Sorting and Relevance
       * 3.8.1 Sorting
       * 3.8.2 String Sorting and Multifields
       * 3.8.3 What Is Relevance?
       * 3.8.4 Doc Values Intro
   * 3.9 Distributed Search Execution
       * 3.9.1 Query Phase
       * 3.9.2 Fetch Phase
       * 3.9.3 Search Options
       * 3.9.4 Scroll
   * 3.10 Index Management
       * 3.10.1 Creating an Index
       * 3.10.2 Deleting an Index
       * 3.10.3 Index Settings
       * 3.10.4 Configuring Analyzers
       * 3.10.5 Custom Analyzers
       * 3.10.6 Types and Mappings
       * 3.10.7 The Root Object
       * 3.10.8 Dynamic Mapping
       * 3.10.9 Customizing Dynamic Mapping
       * 3.10.10 Default Mapping
       * 3.10.11 Reindexing Your Data
       * 3.10.12 Index Aliases and Zero Downtime
   * 3.11 Inside a Shard
       * 3.11.1 Making Text Searchable
       * 3.11.2 Dynamically Updatable Indices
       * 3.11.3 Near Real-Time Search
       * 3.11.4 Making Changes Persistent
       * 3.11.5 Segment Merging
* [4 Search in Depth 搜索进阶](s04/00_search_in_depth.md)
   * 4.1 Structured Search
       * 4.1.1 Finding Exact Values
       * 4.1.2 Combining Filters
       * 4.1.3 Finding Multiple Exact Values
       * 4.1.4 Ranges
       * 4.1.5 Dealing with Null Values
       * 4.1.6 All About Caching
   * 4.2 Full-Text Search
       * 4.2.1 Term-Based Versus Full-Text
       * 4.2.2 The match Query
       * 4.2.3 Multiword Queries
       * 4.2.4 Combining Queries
       * 4.2.5 How match Uses bool
       * 4.2.6 Boosting Query Clauses
       * 4.2.7 Controlling Analysis
       * 4.2.8 Relevance Is Broken!
   * [4.3 Multifield Search 跨字段搜索](s04/03_multifield_search.md)
       * [4.3.1 Multiple Query Strings 多个查询关键词](s04/03_01_multiple_query_strings.md)
       * [4.3.2 Single Query String 单个查询关键词](s04/03_02_single_query_string.md)
       * [4.3.3 Best Fields 最优字段查询](s04/03_03_best_fields.md)
       * [4.3.4 Tuning Best Fields Queries 对最优字段查询进行微调](s04/03_04_tuning_best_fields_queries.md)
       * [4.3.5 multi_match Query multi_match查询](s04/03_05_multimatch_query.md)
       * [4.3.6 Most Fields 最多字段查询](s04/03_06_most_fields.md)
       * [4.3.7 Cross-fields Entity Search 对跨字段型实体进行搜索](s04/03_07_cross-fields_entity_search.md)
       * [4.3.8 Field-Centric Queries 围绕字段来进行查询](s04/03_08_field-centric_queries.md)
       * [4.3.9 Custom \_all Fields 定制你自己的 \_all 字段](s04/03_09_custom_allfields.md)
       * [4.3.10 cross-fields Queries cross-fields查询](s04/03_10_cross-fields_queries.md)
       * 4.3.11 Exact-Value Fields
   * 4.4 Proximity Matching
       * 4.4.1 Phrase Matching
       * 4.4.2 Mixing It Up
       * 4.4.3 Multivalue Fields
       * 4.4.4 Closer Is Better
       * 4.4.5 Proximity for Relevance
       * 4.4.6 Improving Performance
       * 4.4.7 Finding Associated Words
   * 4.5 Partial Matching
       * 4.5.1 Postcodes and Structured Data
       * 4.5.2 prefix Query
       * 4.5.3 wildcard and regexp Queries
       * 4.5.4 Query-Time Search-as-You-Type
       * 4.5.5 Index-Time Optimizations
       * 4.5.6 Ngrams for Partial Matching
       * [4.5.7 Index-Time Search-as-You-Type](s04/05_07_index-time_search-as-you-type.md)
       * 4.5.8 Ngrams for Compound Words
   * 4.6 Controlling Relevance
       * 4.6.1 Theory Behind Relevance Scoring
       * 4.6.2 Lucene’s Practical Scoring Function
       * 4.6.3 Query-Time Boosting
       * 4.6.4 Manipulating Relevance with Query Structure
       * 4.6.5 Not Quite Not
       * 4.6.6 Ignoring TF/IDF
       * 4.6.7 function_score Query
       * 4.6.8 Boosting by Popularity
       * 4.6.9 Boosting Filtered Subsets
       * 4.6.10 Random Scoring
       * 4.6.11 The Closer, The Better
       * 4.6.12 Understanding the price Clause
       * 4.6.13 Scoring with Scripts
       * 4.6.14 Pluggable Similarity Algorithms
       * 4.6.15 Changing Similarities
       * 4.6.16 Relevance Tuning Is the Last 10%
* [5 Dealing with Human Language 处理人类语言](s05/00_dealing_with_human_language.md)
   * [5.1 Getting Started with Languages 入门知识](s05/01_getting_started_with_languages.md)
       * [5.1.1 Using Language Analyzers 使用语言解析器](s05/01_01_using_language_analyzers.md)
       * [5.1.2 Configuring Language Analyzers 配置语言解析器](s05/01_02_configuring_language_analyzers.md)
       * [5.1.3 Pitfalls of Mixing Languages 多语言的坑](s05/01_03_pitfalls_of_mixing_languages.md)
       * [5.1.4 One Language per Document 单个文档对应一门主语言](s05/01_04_one_language_per_document.md)
       * [5.1.5 One Language per Field 单个字段对应一门主语言](s05/01_05_one_language_per_field.md)
       * [5.1.6 Mixed-Language Fields 单个字段对应多门语言](s05/01_06_mixed-language_fields.md)
   * [5.2 Identifying Words 识别字词](s05/02_identifying_words.md)
       * [5.2.1 standard Analyzer 标准解析器](s05/02_01_standard_analyzer.md)
       * [5.2.2 standard Tokenizer 标准分词器](s05/02_02_standard_tokenizer.md)
       * [5.2.3 Installing the ICU Plug-in 安装 ICU 插件](s05/02_03_installing_the_icu_plug-in.md)
       * [5.2.4 icu_tokenizer icu 分词器](s05/02_04_icu_tokenizer.md)
       * [5.2.5 Tidying Up Input Text 优化输入的文本](s05/02_05_tidying_up_input_text.md)
   * [5.3 Normalizing Tokens 规范化标记](s05/03_normalizing_tokens.md)
       * [5.3.1 In That Case](s05/03_01_in_that_case.md)
       * [5.3.2 You Have an Accent](s05/03_02_you_have_an_accent.md)
       * 5.3.3 Living in a Unicode World
       * 5.3.4 Unicode Case Folding
       * 5.3.5 Unicode Character Folding
       * 5.3.6 Sorting and Collations
   * 5.4 Reducing Words to Their Root Form
       * 5.4.1 Algorithmic Stemmers
       * 5.4.2 Dictionary Stemmers
       * 5.4.3 Hunspell Stemmer
       * 5.4.4 Choosing a Stemmer
       * 5.4.5 Controlling Stemming
       * 5.4.6 Stemming in situ
   * 5.5 Stopwords: Performance Versus Precision
       * 5.5.1 Pros and Cons of Stopwords
       * 5.5.2 Using Stopwords
       * 5.5.3 Stopwords and Performance
       * 5.5.4 Divide and Conquer
       * 5.5.5 Stopwords and Phrase Queries
       * 5.5.6 common_grams Token Filter
       * 5.5.7 Stopwords and Relevance
   * [5.6 Synonyms 同义词](s05/06_synonyms.md)
       * [5.6.1 Using Synonyms 使用同义词](s05/06_01_using_synonyms.md)
       * [5.6.2 Formatting Synonyms 同义词规则的书写格式](s05/06_02_formatting_synonyms.md)
       * [5.6.3 Expand or contract 拓展还是收缩](s05/06_03_expand_or_contract.md)
       * [5.6.4 Synonyms and The Analysis Chain 同义词与解析器链](s05/06_04_synonyms_and_the_analysis_chain.md)
       * [5.6.5 Multiword Synonyms and Phrase Queries 多词同义与短语查询](s05/06_05_multiword_synonyms_and_phrase_queries.md)
       * [5.6.6 Symbol Synonyms 符号同义词](s05/06_06_symbol_synonyms.md)
   * [5.7 Typoes and Mispelings 拼写错误](s05/07_typoes_and_mispelings.md)
       * [5.7.1 Fuzziness 怎么判断是否存在拼写错误](s05/07_01_fuzziness.md)
       * [5.7.2 Fuzzy Query 模糊查询](s05/07_02_fuzzy_query.md)
       * 5.7.3 Fuzzy match Query
       * 5.7.4 Scoring Fuzziness
       * 5.7.5 Phonetic Matching
* [6 Aggregations](s06/00_aggregations.md)
   * High-Level Concepts
   * Aggregation Test-Drive
   * Building Bar Charts
   * Looking at Time
   * Scoping Aggregations
   * Filtering Queries and Aggregations
   * Sorting Multivalue Buckets
   * Approximate Aggregations
   * Significant Terms
   * Doc Values and Fielddata
   * Closing Thoughts
* [7 Geolocation](s07/00_geolocation.md)
   * Geo Points
   * Geohashes
   * Geo Aggregations
   * Geo Shapes
* [8 Modeling Your Data](s08/00_modeling_your_data.md)
   * Handling Relationships
   * Nested Objects
   * Parent-Child Relationship
   * Designing for Scale
* [9 Administration, Monitoring, and Deployment](s09/00_administration,_monitoring,_and_deployment.md)
   * Monitoring
   * Production Deployment
   * Post-Deployment

