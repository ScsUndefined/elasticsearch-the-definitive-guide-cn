# 4.5.7 Index-Time Search-as-You-Type

The first step to setting up index-time search-as-you-type is to define our analysis chain, which we discussed in [Configuring Analyzers](https://www.elastic.co/guide/en/elasticsearch/guide/current/configuring-analyzers.html), but we will go over the steps again here.

## Preparing the Index

The first step is to configure a custom `edge_ngram` token filter, which we will call the `autocomplete_filter`:

```bash
{
    "filter": {
        "autocomplete_filter": {
            "type":     "edge_ngram",
            "min_gram": 1,
            "max_gram": 20
        }
    }
}
```

This configuration says that, for any term that this token filter receives, it should produce an n-gram anchored to the start of the word of minimum length 1 and maximum length 20.

Then we need to use this token filter in a custom analyzer, which we will call the `autocomplete` analyzer:

```bash
{
    "analyzer": {
        "autocomplete": {
            "type":      "custom",
            "tokenizer": "standard",
            "filter": [
                "lowercase",
                "autocomplete_filter" ①
            ]
        }
    }
}
```

① Our custom edge-ngram token filter

This analyzer will tokenize a string into individual terms by using the `standard` tokenizer, lowercase each term, and then produce edge n-grams of each term, thanks to our `autocomplete_filter`.

The full request to create the index and instantiate the token filter and analyzer looks like this:

```bash
PUT /my_index
{
    "settings": {
        "number_of_shards": 1, ①
        "analysis": {
            "filter": {
                "autocomplete_filter": { ②
                    "type":     "edge_ngram",
                    "min_gram": 1,
                    "max_gram": 20
                }
            },
            "analyzer": {
                "autocomplete": {
                    "type":      "custom",
                    "tokenizer": "standard",
                    "filter": [
                        "lowercase",
                        "autocomplete_filter" ③
                    ]
                }
            }
        }
    }
}
```

VIEW IN SENSE 


See Relevance Is Broken!.



First we define our custom token filter.



Then we use it in an analyzer.

You can test this new analyzer to make sure it is behaving correctly by using the analyze API:

GET /my_index/_analyze
{
  "analyzer": "autocomplete",
  "text": "quick brown"
}
VIEW IN SENSE 
The results show us that the analyzer is working correctly. It returns these terms:

q
qu
qui
quic
quick
b
br
bro
brow
brown
To use the analyzer, we need to apply it to a field, which we can do with the update-mapping API:

PUT /my_index/_mapping/my_type
{
    "my_type": {
        "properties": {
            "name": {
                "type":     "string",
                "analyzer": "autocomplete"
            }
        }
    }
}
VIEW IN SENSE 
Now, we can index some test documents:

POST /my_index/my_type/_bulk
{ "index": { "_id": 1            }}
{ "name": "Brown foxes"    }
{ "index": { "_id": 2            }}
{ "name": "Yellow furballs" }
VIEW IN SENSE 
Querying the Fieldedit
If you test out a query for “brown fo” by using a simple match query

GET /my_index/my_type/_search
{
    "query": {
        "match": {
            "name": "brown fo"
        }
    }
}
VIEW IN SENSE 
you will see that both documents match, even though the Yellow furballs doc contains neither brown nor fo:

{

  "hits": [
     {
        "_id": "1",
        "_score": 1.5753809,
        "_source": {
           "name": "Brown foxes"
        }
     },
     {
        "_id": "2",
        "_score": 0.012520773,
        "_source": {
           "name": "Yellow furballs"
        }
     }
  ]
}
As always, the validate-query API shines some light:

GET /my_index/my_type/_validate/query?explain
{
    "query": {
        "match": {
            "name": "brown fo"
        }
    }
}
VIEW IN SENSE 
The explanation shows us that the query is looking for edge n-grams of every word in the query string:

name:b name:br name:bro name:brow name:brown name:f name:fo
The name:f condition is satisfied by the second document because furballs has been indexed as f, fu, fur, and so forth. In retrospect, this is not surprising. The same autocomplete analyzer is being applied both at index time and at search time, which in most situations is the right thing to do. This is one of the few occasions when it makes sense to break this rule.

We want to ensure that our inverted index contains edge n-grams of every word, but we want to match only the full words that the user has entered (brown and fo). We can do this by using the autocomplete analyzer at index time and the standard analyzer at search time. One way to change the search analyzer is just to specify it in the query:

GET /my_index/my_type/_search
{
    "query": {
        "match": {
            "name": {
                "query":    "brown fo",
                "analyzer": "standard" 
            }
        }
    }
}
VIEW IN SENSE 


This overrides the analyzer setting on the name field.

Alternatively, we can specify the analyzer and search_analyzer in the mapping for the name field itself. Because we want to change only the search_analyzer, we can update the existing mapping without having to reindex our data:

PUT /my_index/my_type/_mapping
{
    "my_type": {
        "properties": {
            "name": {
                "type":            "string",
                "analyzer":  "autocomplete", 
                "search_analyzer": "standard" 
            }
        }
    }
}
VIEW IN SENSE 


Use the autocomplete analyzer at index time to produce edge n-grams of every term.



Use the standard analyzer at search time to search only on the terms that the user has entered.

If we were to repeat the validate-query request, it would now give us this explanation:

name:brown name:fo
Repeating our query correctly returns just the Brown foxes document.

Because most of the work has been done at index time, all this query needs to do is to look up the two terms brown and fo, which is much more efficient than the match_phrase_prefix approach of having to find all terms beginning with fo.

Completion Suggester
Using edge n-grams for search-as-you-type is easy to set up, flexible, and fast. However, sometimes it is not fast enough. Latency matters, especially when you are trying to provide instant feedback. Sometimes the fastest way of searching is not to search at all.

The completion suggester in Elasticsearch takes a completely different approach. You feed it a list of all possible completions, and it builds them into a finite state transducer, an optimized data structure that resembles a big graph. To search for suggestions, Elasticsearch starts at the beginning of the graph and moves character by character along the matching path. Once it has run out of user input, it looks at all possible endings of the current path to produce a list of suggestions.

This data structure lives in memory and makes prefix lookups extremely fast, much faster than any term-based query could be. It is an excellent match for autocompletion of names and brands, whose words are usually organized in a common order: “Johnny Rotten” rather than “Rotten Johnny.”

When word order is less predictable, edge n-grams can be a better solution than the completion suggester. This particular cat may be skinned in myriad ways.

Edge n-grams and Postcodesedit
The edge n-gram approach can also be used for structured data, such as the postcodes example from earlier in this chapter. Of course, the postcode field would need to be analyzed instead of not_analyzed, but you could use the keyword tokenizer to treat the postcodes as if they were not_analyzed.

Tip
The keyword tokenizer is the no-operation tokenizer, the tokenizer that does nothing. Whatever string it receives as input, it emits exactly the same string as a single token. It can therefore be used for values that we would normally treat as not_analyzed but that require some other analysis transformation such as lowercasing.

This example uses the keyword tokenizer to convert the postcode string into a token stream, so that we can use the edge n-gram token filter:

{
    "analysis": {
        "filter": {
            "postcode_filter": {
                "type":     "edge_ngram",
                "min_gram": 1,
                "max_gram": 8
            }
        },
        "analyzer": {
            "postcode_index": { 
                "tokenizer": "keyword",
                "filter":    [ "postcode_filter" ]
            },
            "postcode_search": { 
                "tokenizer": "keyword"
            }
        }
    }
}
VIEW IN SENSE 


The postcode_index analyzer would use the postcode_filter to turn postcodes into edge n-grams.



The postcode_search analyzer would treat search terms as if they were not_analyzed.