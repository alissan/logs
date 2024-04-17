## elasticsearch version:

```
GET /
```

```
{
  "name": "c1",
  "cluster_name": "cluster1",
  "cluster_uuid": "abc",
  "version": {
    "number": "8.8.2",
    "build_flavor": "default",
    "build_type": "rpm",
    "build_hash": "98e1271edf932a480e4262a471281f1ee295ce6b",
    "build_date": "2023-06-26T05:16:16.196344851Z",
    "build_snapshot": false,
    "lucene_version": "9.6.0",
    "minimum_wire_compatibility_version": "7.17.0",
    "minimum_index_compatibility_version": "7.0.0"
  },
  "tagline": "You Know, for Search"
}
```

## create index with old mapping

```
DELETE test_1

PUT /test_1
{
  "settings": {
    "number_of_shards": 1
  },
  "mappings": {
    "_routing": {
      "required": false
    },
    "_source": {
      "excludes": [],
      "includes": [],
      "enabled": true
    },
    "date_detection": false,
    "numeric_detection": false,
    "dynamic": "runtime",
    "properties": {
      "ts": {
        "type": "date"
      },
      "suser": {
        "type": "keyword"
      }
    }
    }
}
```

### Output:

```
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "test_1"
}
```


## adding data with bash.command.test column

```
POST _bulk
{"index":{"_index":"test_1","_id":null}}
{"suser":"ali","bash.command.test":"ls -ltr"}
```

### Output:

```
{
  "took": 1197,
  "errors": false,
  "items": [
    {
      "index": {
        "_index": "test_1",
        "_id": "NSMa644ByNLoKOEE390y",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 2,
          "failed": 0
        },
        "_seq_no": 0,
        "_primary_term": 1,
        "status": 201
      }
    }
  ]
}
```

## mapping
```
GET test_1/_mapping
```

### Output:

```
{
  "test_1": {
    "mappings": {
      "dynamic": "runtime",
      "date_detection": false,
      "numeric_detection": false,
      "runtime": {
        "bash.command.test": {
          "type": "keyword"
        }
      },
      "properties": {
        "suser": {
          "type": "keyword"
        },
        "ts": {
          "type": "date"
        }
      }
    }
  }
}
```

## create index with new mapping (lowercased runtime fields)

```
DELETE test_2

PUT /test_2
{
  "settings": {
    "number_of_shards": 1
  },
  "mappings": {
    "_routing": {
      "required": false
    },
    "numeric_detection": false,
    "_source": {
      "excludes": [],
      "includes": [],
      "enabled": true
    },
    "dynamic": "runtime",
    "dynamic_templates" : [
        {
          "lowercase_dynamic_fields" : {
            "match" : "*",
            "match_mapping_type" : "string",
            "mapping" : {
              "normalizer" : "lowercase",
              "type" : "keyword"
            }
          }
        }
      ],
    "date_detection": false,
    "properties": {
      "ts": {
        "type": "date"
      },
      "suser": {
        "normalizer": "lowercase",
        "type": "keyword"
      }
    }
  }
}
```

### Output:

```
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "test_2"
}
```


## adding data to new index with bash.command.test field

```
POST _bulk
{"index":{"_index":"test_2","_id":null}}
{"suser":"ali","bash.command.test":"ls -ltr"}
```


### Output: 

```
{
  "took": 2,
  "errors": true,
  "items": [
    {
      "index": {
        "_index": "test_2",
        "_id": "Ya0c644ByNLoKOEEtVPN",
        "status": 500,
        "error": {
          "type": "illegal_state_exception",
          "reason": "Missing intermediate object bash"
        }
      }
    }
  ]
}
```

## mapping 

```
GET test_2/_mapping
```

### Output:

```
{
  "test_2": {
    "mappings": {
      "dynamic": "runtime",
      "dynamic_templates": [
        {
          "lowercase_dynamic_fields": {
            "match": "*",
            "match_mapping_type": "string",
            "mapping": {
              "normalizer": "lowercase",
              "type": "keyword"
            }
          }
        }
      ],
      "date_detection": false,
      "numeric_detection": false,
      "properties": {
        "suser": {
          "type": "keyword",
          "normalizer": "lowercase"
        },
        "ts": {
          "type": "date"
        }
      }
    }
  }
}
```
