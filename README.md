## Run commande ตามนี้

```
docker-compose up
```

## Host การเข้าถึง

```
elasticsearch => localhost:9200
kibana => localhost:5601
```

## เมื่อรัน kibana ขึ้นแล้วให้ก็อบคำสั่งนี้ลง Dev Tools

```
PUT the_avengers
{
    "settings": {
        "analysis": {
            "filter": {
                "thai_stop": {
                    "type": "stop",
                    "stopwords": "_thai_"
                },
                "word_delimiter_index_filter": {
                    "type": "word_delimiter",
                    "generate_word_parts": true,
                    "generate_number_parts": true,
                    "catenate_words": false,
                    "catenate_numbers": false,
                    "catenate_all": true,
                    "split_on_case_change": true
                },
                "word_delimiter_search_filter": {
                    "type": "word_delimiter",
                    "generate_word_parts": false,
                    "generate_number_parts": false,
                    "catenate_words": false,
                    "catenate_numbers": false,
                    "catenate_all": false,
                    "split_on_case_change": false
                },
                "shingle_filter": {
                    "type": "shingle",
                    "min_shingle_size": 2,
                    "max_shingle_size": 2,
                    "output_unigrams": true,
                    "output_unigrams_if_no_shingles": false,
                    "token_separator": ""
                },
                "shorten_filter": {
                    "type": "pattern_replace",
                    "pattern": "^(.{20})(.*)?",
                    "replacement": "$1",
                    "replace": "all"
                },
                "edge_ngram_filter": {
                    "type": "edgeNGram",
                    "min_gram": 2,
                    "max_gram": 16,
                    "side": "front"
                }
            },
            "analyzer": {
                "thai_generic_index_analyzer": {
                    "tokenizer": "thai",
                    "filter": [
                        "lowercase",
                        "thai_stop",
                        "word_delimiter_index_filter",
                        "edge_ngram_filter",
                        "shingle_filter",
                        "unique"
                    ]
                },
                "thai_generic_search_analyzer": {
                    "tokenizer": "thai",
                    "filter": [
                        "lowercase",
                        "thai_stop",
                        "word_delimiter_search_filter",
                        "shingle_filter",
                        "unique",
                        "shorten_filter"
                    ]
                },
                "thai_related_analyzer": {
                    "tokenizer": "thai",
                    "filter": [
                        "lowercase",
                        "thai_stop",
                        "word_delimiter_search_filter",
                        "unique",
                        "shorten_filter"
                    ]
                }
            }
        }
    },  
    "search": {
       "properties": {
            "section": {
               "type": "keyword"
            },
            "title": {
                "type": "text",
                "analyzer": "thai_generic_index_analyzer",
                "search_analyzer": "thai_generic_search_analyzer"
            },
            "description": {
                "type": "text",
                "analyzer": "thai_generic_index_analyzer",
                "search_analyzer": "thai_generic_search_analyzer"
            },
            "categories": {
                "type": "text",
                "analyzer": "thai_generic_index_analyzer",
                "search_analyzer": "thai_generic_search_analyzer"
            },
            "tags": {
                "type": "text"
            },
            "created": {
                "type": "date",
                "include_in_all": false,
                "format": "yyyy-MM-dd HH:mm:ss"
            },
            "can_search": {
                "type": "boolean"
            }
        }
    }
}

DELETE the_avengers

PUT the_avengers/search/1
{
    "section" : "hero",
    "title" : "Iron Man",
    "categories" : "name",
    "tags" : "avengers",
    "created" : "2019-01-12",
    "can_search" : true
}

PUT test/search/2
{
    "section" : "hero",
    "title" : "Captain America",
    "categories" : "name",
    "tags" : "avengers",
    "created" : "2019-01-13",
    "can_search" : false
}

PUT test/search/3
{
    "section" : "hero",
    "title" : "Thor",
    "categories" : "name",
    "tags" : "avengers",
    "created" : "2019-01-14",
    "can_search" : true
}

PUT test/search/4
{
    "section" : "hero",
    "title" : "Captain Marvel",
    "categories" : "name",
    "tags" : "avengers",
    "created" : "2019-01-13",
    "can_search" : false
}

GET the_avengers/search/1

GET /_search
{
  "query": { 
    "bool": { 
      "must": [
        { "match": { "tags": "avengers"}}  
      ],
      "filter": [ 
        { "term":  { "can_search": true }}
      ]
    }
  }
}

```