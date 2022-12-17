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
> This cannot be undone
