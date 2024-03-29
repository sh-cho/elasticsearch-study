## es-study

### versions
- elasticsearch: 7.13.4
- kibana: 7.13.4

### How to run

```
docker compose up -d
```
Run with docker compose

```
docker compose stop
docker compose rm  # optional
```
stop, remove

```
docker compose start
```
start after stop

#### url

- ES: http://localhost:9200/
- kibana: http://localhost:5601/

## Instructions
- https://github.com/javacafe-project/

### 1.3 Setup environment
#### 1.3.1 Setup ES

```sh
curl -XPUT 'http://localhost:9200/_snapshot/javacafe' -H 'Content-Type: application/json' -d '
{
  "type": "fs",
  "settings": {
    "location": "/usr/share/elasticsearch/backup/search_example",
    "compress": true
  }
}
'
```
Create snapshot (1) `javacafe`

```json
{
  "snapshots" : [
    {
      "snapshot" : "movie-search",
      "uuid" : "Kz5k4fusS7KBZy55wLeZ0Q",
      "version_id" : 6040399,
      "version" : "6.4.3",
      "indices" : [
        "movie_search"  // check
      ],
      // ...
    }
  ]
}
```
Check: http://localhost:9200/_snapshot/javacafe/_all?pretty

```sh
curl -XPUT 'http://localhost:9200/_snapshot/apache-web-log' -H 'Content-Type: application/json' -d '
{
  "type": "fs",
  "settings": {
    "location": "/usr/share/elasticsearch/backup/agg_example",
    "compress": true
  }
}
'
```
Create snapshot (2) `apache-web-log`

```json
{
  "snapshots" : [
    {
      "snapshot" : "default",
      "uuid" : "yzmzEx6uSMS55j60z4buBA",
      "version_id" : 6040399,
      "version" : "6.4.3",
      "indices" : [
        "apache-web-log"  // check
      ],
      // ...
    },
    {
      "snapshot" : "applied-mapping",
      "uuid" : "SgXhqApiSHiauC6fbjSHMw",
      "version_id" : 6040399,
      "version" : "6.4.3",
      "indices" : [
        "apache-web-log-applied-mapping"  // check
      ],
      // ...
    }
  ]
}
```
Check: http://localhost:9200/_snapshot/apache-web-log/_all?pretty

```sh
POST _snapshot/javacafe/movie-search/_restore
POST _snapshot/apache-web-log/default/_restore
```

(How to restore -> see 6.6 (311p))

`POST _snapshot/{snapshot_name}/{index_name}/_restore`

#### 1.3.2 Setup Kibana

```sh
PUT movie_kibana_execute/_doc/1
{
  "message": "helloworld"
}
```
```json
{
  "_index" : "movie_kibana_execute",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```
This creates index and document

```sh
GET movie_kibana_execute/_search
{
  "query": {
    "match_all": {}
  }
}
```
```json
{
  "took" : 6,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "movie_kibana_execute",
        "_type" : "_doc",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "message" : "helloworld"
        }
      }
    ]
  }
}
```

### 2.2 Elasticsearch API
#### 2.2.1 Index managing API

```sh
PUT /movie
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 2
  },
  "mappings": {
    "properties": {
      "movieCd": {"type": "integer"},
      "movieNm": {"type": "text"},
      "movieNmEn": {"type": "text"},
      "prdtYear": {"type": "integer"},
      "openDt": {"type": "date"},
      "typeNm": {"type": "keyword"},
      "prdtStatNm": {"type": "keyword"},
      "nationAlt": {"type": "keyword"},
      "genreAlt": {"type": "keyword"},
      "repNationNm": {"type": "keyword"},
      "repGenreNm": {"type": "keyword"}
    }
  }
}
```

```json
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "movie"
}
```
Create index

```
DELETE /movie
```
```
{
  "acknowledged" : true
}
```
Delete index

> **Warning**
>
> This cannot be undone

#### 2.2.2 document managing API

```sh
PUT /movie/_doc/1
{
  "movieCd": "1",
  "movieNm": "살아남은 아이",
  "movieNmEn": "Last Child",
  "prdtYear": "2017",
  "openDt": "2017-10-20",
  "typeNm": "장편",
  "prdtStatNm": "기타",
  "nationAlt": "한국",
  "genreAlt": "드라마,가족",
  "repNationNm": "한국",
  "repGenreNm": "드라마"
}
```

```json
{
  "_index" : "movie",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 3,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```
Create and set id manually

```
POST /movie/_search?q=typeNm:장편
```
Query with parameter (URI)

```sh
POST /movie/_search
{
  "query": {
    "term": {
      "typeNm": "장편"
    }
  }
}
```
Query (Request body)

Options
- size: How many document? (default: 10)
- from
- [fields](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-fields.html#search-fields-param): to retrieve specific fields
  - Response always returns an array
  - By default, returns only values of mapped fields
- [_source](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-fields.html#source-filtering): select what fields of the source are returned.
  - `false`: document source is not included in the response
  - `["obj1.*", "obj2.*"]`: specifying fields
  - > Using `fields` is typically better (as it says)
- sort
- query
- filter

(see more: https://www.elastic.co/guide/en/elasticsearch/reference/current/search-your-data.html)

#### 2.2.4 Aggregation API

```sh
POST /movie/_search?size=0
{
  "aggs": {
    "genre": {
      "terms": {"field": "genreAlt"}
    }
  }
}
```

### 3.1 Mapping API

#### 3.1.1 Create Mapping index
```sh
PUT movie_search
{
  "settings": {
    "number_of_shards": 5,
    "number_of_replicas": 1
  },
  "mappings": {
    "properties": {
      "movieCd": {
        "type": "keyword"
      },
      "movieNm": {
        "type": "text",
        "analyzer": "standard"
      },
      "movieNmEn": {
        "type": "text",
        "analyzer": "standard"
      },
      "prdtYear": {
        "type": "integer"
      },
      "openDt": {
        "type": "integer"
      },
      "typeNm": {
        "type": "keyword"
      },
      "prdtStatNm": {
        "type": "keyword"
      },
      "nationAlt": {
        "type": "keyword"
      },
      "genreAlt": {
        "type": "keyword"
      },
      "repNationNm": {
        "type": "keyword"
      },
      "repGenreNm": {
        "type": "keyword"
      },
      "companies": {
        "properties": {
          "companyCd": {
            "type": "keyword"
          },
          "companyNm": {
            "type": "keyword"
          }
        }
      },
      "directors": {
        "properties": {
          "peopleNm": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```
Create index

```sh
GET movie_search/_mapping
```
```json
{
  "movie_search" : {
    "mappings" : {
      "properties" : {
        "companies" : {
          "properties" : {
            "companyCd" : {
              "type" : "keyword"
            },
            "companyNm" : {
              "type" : "keyword"
            }
          }
        },
        "directors" : {
          "properties" : {
            "peopleNm" : {
              "type" : "keyword"
            }
          }
        },
        "genreAlt" : {
          "type" : "keyword"
        },
        "movieCd" : {
          "type" : "keyword"
        },
        "movieNm" : {
          "type" : "text",
          "analyzer" : "standard"
        },
        "movieNmEn" : {
          "type" : "text",
          "analyzer" : "standard"
        },
        "nationAlt" : {
          "type" : "keyword"
        },
        "openDt" : {
          "type" : "integer"
        },
        "prdtStatNm" : {
          "type" : "keyword"
        },
        "prdtYear" : {
          "type" : "integer"
        },
        "repGenreNm" : {
          "type" : "keyword"
        },
        "repNationNm" : {
          "type" : "keyword"
        },
        "typeNm" : {
          "type" : "keyword"
        }
      }
    }
  }
}
```
Get mapping info

[mapping parameters](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-params.html):

- analyzer
- normalizer
- boost
- coerce
- copy_to
- ...

### 3.2 Meta fields

#### 3.2.1 `_index`

```sh
POST movie_search/_search
{
 "size":0,
  "aggs": {
    "indices": {
      "terms": {
        "field": "_index",
        "size": 10
      }
    }
  }
}
```
```json
{
  "took" : 7,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "indices" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [ ]
    }
  }
}
```

