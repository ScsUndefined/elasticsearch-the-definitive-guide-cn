# 定制你的 \_all 字段

在 Metadata: \_all Field 一章节中已经介绍过了，\_all 字段存储了所有其他字段的数据。但有时把所有的其他字段都放到一个字段中的做法远不够领过。因此最好你能够根据你的实际情况定制一个你自己想要的 \_all 型的字段。

Elasticsearch 提供了一个叫做 copy_to 的参数来让你在映射数据的时候使用，以满足你的这类需求：

```bash
PUT /my_index
{
    "mappings": {
        "person": {
            "properties": {
                "first_name": {
                    "type":     "string",
                    "copy_to":  "full_name" ①
                },
                "last_name": {
                    "type":     "string",
                    "copy_to":  "full_name" ②
                },
                "full_name": {
                    "type":     "string"
                }
            }
        }
    }
}
```
[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/45_Custom_all.json) 
 
①,② first_name 和 last_name 字段的值也同时被拷贝到了 full_name 字段

这么来映射的话，我们就可以用 first_name 字段来查询名字，用 last_name 字段来查询姓氏，然后可以用 full_name 来查询全名。

first_name 和 last_name 字段的映射规则不会对 full_name 造成影响。在入索引的时候，first_name 和 last_name 的原始值会被直接拷贝过来用 full_name 的映射规则存储到 full_name 字段中。

> **警告**
> 
> copy_to 并不适用于 multi-field。如果你试图这么来映射的话，Elasticsearch 会报错的。
>
> 这是因为 Multi-fields 只是简单地把主字段的值以不同的方式重新入了一遍索引；它们本身并不持有原始的数据信息。也就没啥好让你拷贝的了。~~直接指向它的主字段就好了吧，毕竟主字段和副字段用的原始数据都是一致的~~
>
> 你可以用它的主字段就好了:
> 
> ```bash
PUT /my_index
{
    "mappings": {
        "person": {
            "properties": {
                "first_name": {
                    "type":     "string",
                    "copy_to":  "full_name", ①
                    "fields": {
                        "raw": {
                            "type": "string",
                            "index": "not_analyzed"
                        }
                    }
                },
                "full_name": {
                    "type":     "string"
                }
            }
        }
    }
}
```

> ① copy_to 参数写在主字段上而非副字段上。

***

# Custom \_all Fields

In Metadata: \_all Field, we explained that the special \_all field indexes the values from all other fields as one big string. Having all fields indexed into one field is not terribly flexible, though. It would be nice to have one custom \_all field for the person’s name, and another custom \_all field for the address.

Elasticsearch provides us with this functionality via the copy_to parameter in a field mapping:

```bash
PUT /my_index
{
    "mappings": {
        "person": {
            "properties": {
                "first_name": {
                    "type":     "string",
                    "copy_to":  "full_name" ①
                },
                "last_name": {
                    "type":     "string",
                    "copy_to":  "full_name" ②
                },
                "full_name": {
                    "type":     "string"
                }
            }
        }
    }
}
```
[VIEW IN SENSE](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/en/elasticsearch/guide/current/snippets/110_Multi_Field_Search/45_Custom_all.json) 
 
①,② The values in the first_name and last_name fields are also copied to the full_name field.

With this mapping in place, we can query the first_name field for first names, the last_name field for last name, or the full_name field for first and last names.

Mappings of the first_name and last_name fields have no bearing on how the full_name field is indexed. The full_name field copies the string values from the other two fields, then indexes them according to the mapping of the full_name field only.

> **Warning**
> 
> The copy_to setting will not work on a multi-field. If you attempt to configure your mapping this way, Elasticsearch will throw an exception.
>
> Why? Multi-fields are simply indexing the "main" field a different way; they don’t have their own source. Which means there is no source to copy_to a different field.
>
> You can easily copy_to the "main" field to achieve the same effect:
> 
> ```bash
PUT /my_index
{
    "mappings": {
        "person": {
            "properties": {
                "first_name": {
                    "type":     "string",
                    "copy_to":  "full_name", ①
                    "fields": {
                        "raw": {
                            "type": "string",
                            "index": "not_analyzed"
                        }
                    }
                },
                "full_name": {
                    "type":     "string"
                }
            }
        }
    }
}
```

> ① copy_to is placed on the "main" field rather than the multi-field