# 4.3.6 Most Fields 

Full-text search is a battle between recall—returning all the documents that are relevant—and precision—not returning irrelevant documents. The goal is to present the user with the most relevant documents on the first page of results.

To improve recall, we cast the net wide—we include not only documents that match the user’s search terms exactly, but also documents that we believe to be pertinent to the query. If a user searches for “quick brown fox,” a document that contains fast foxes may well be a reasonable result to return.

If the only pertinent document that we have is the one containing fast foxes, it will appear at the top of the results list. But of course, if we have 100 documents that contain the words quick brown fox, then the fast foxes document may be considered less relevant, and we would want to push it further down the list. After including many potential matches, we need to ensure that the best ones rise to the top.

A common technique for fine-tuning full-text relevance is to index the same text in multiple ways, each of which provides a different relevance signal. The main field would contain terms in their broadest-matching form to match as many documents as possible. For instance, we could do the following:

  * Use a stemmer to index jumps, jumping, and jumped as their root form: jump. Then it doesn’t matter if the user searches for jumped; we could still match documents containing jumping.
  * Include synonyms like jump, leap, and hop.
  * Remove diacritics, or accents: for example, ésta, está, and esta would all be indexed without accents as esta.

However, if we have two documents, one of which contains jumped and the other jumping, the user would probably expect the first document to rank higher, as it contains exactly what was typed in.

We can achieve this by indexing the same text in other fields to provide more-precise matching. One field may contain the unstemmed version, another the original word with diacritics, and a third might use shingles to provide information about word proximity. These other fields act as signals that increase the relevance score of each matching document. The more fields that match, the better.

A document is included in the results list if it matches the broad-matching main field. If it also matches the signal fields, it gets extra points and is pushed up the results list.

We discuss synonyms, word proximity, partial-matching and other potential signals later in the book, but we will use the simple example of stemmed and unstemmed fields to illustrate this technique.

## Multifield Mapping

The first thing to do is to set up our field to be indexed twice: once in a stemmed form and once in an unstemmed form. To do this, we will use multifields, which we introduced in String Sorting and Multifields:

```bash
DELETE /my_index

PUT /my_index
{
    "settings": { "number_of_shards": 1 }, ①
    "mappings": {
        "my_type": {
            "properties": {
                "title": { ②
                    "type":     "string",
                    "analyzer": "english",
                    "fields": {
                        "std":   { ③
                            "type":     "string",
                            "analyzer": "standard"
                        }
                    }
                }
            }
        }
    }
}
```
[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/30_Most_fields.json) 


① See Relevance Is Broken!.

② The title field is stemmed by the english analyzer.

③ The title.std field uses the standard analyzer and so is not stemmed.

Next we index some documents:

```bash
PUT /my_index/my_type/1
{ "title": "My rabbit jumps" }

PUT /my_index/my_type/2
{ "title": "Jumping jack rabbits" }
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/30_Most_fields.json) 

Here is a simple match query on the title field for jumping rabbits:

```bash
GET /my_index/_search
{
   "query": {
        "match": {
            "title": "jumping rabbits"
        }
    }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/30_Most_fields.json) 

This becomes a query for the two stemmed terms jump and rabbit, thanks to the english analyzer. The title field of both documents contains both of those terms, so both documents receive the same score:

```bash
{
  "hits": [
     {
        "_id": "1",
        "_score": 0.42039964,
        "_source": {
           "title": "My rabbit jumps"
        }
     },
     {
        "_id": "2",
        "_score": 0.42039964,
        "_source": {
           "title": "Jumping jack rabbits"
        }
     }
  ]
}
```

If we were to query just the title.std field, then only document 2 would match. However, if we were to query both fields and to combine their scores by using the bool query, then both documents would match (thanks to the title field) and document 2 would score higher (thanks to the title.std field):

```bash
GET /my_index/_search
{
   "query": {
        "multi_match": {
            "query":  "jumping rabbits",
            "type":   "most_fields", ①
            "fields": [ "title", "title.std" ]
        }
    }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/30_Most_fields.json) 

① We want to combine the scores from all matching fields, so we use the most_fields type. This causes the multi_match query to wrap the two field-clauses in a bool query instead of a dis_max query.

```bash
{
  "hits": [
     {
        "_id": "2",
        "_score": 0.8226396, ①
        "_source": {
           "title": "Jumping jack rabbits"
        }
     },
     {
        "_id": "1",
        "_score": 0.10741998, ②
        "_source": {
           "title": "My rabbit jumps"
        }
     }
  ]
}
``` 

①,② Document 2 now scores much higher than document 1.

We are using the broad-matching title field to include as many documents as possible—to increase recall—but we use the title.std field as a signal to push the most relevant results to the top.

The contribution of each field to the final score can be controlled by specifying custom boost values. For instance, we could boost the title field to make it the most important field, thus reducing the effect of any other signal fields:

```bash
GET /my_index/_search
{
   "query": {
        "multi_match": {
            "query":       "jumping rabbits",
            "type":        "most_fields",
            "fields":      [ "title^10", "title.std" ] ①
        }
    }
}
```

[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/30_Most_fields.json) 

① The boost value of 10 on the title field makes that field relatively much more important than the title.std field.