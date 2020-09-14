# ELK search

ElasticSearch는 Java 오픈소스 분산 검색 엔진 (Apache Lucene 기반)
방대한 양의 데이터를 신속하게, 거의 실시간으로 저장/검색/분석할 수 있다

ElasticSearch는 검색을 위해 단독으로 사용되기도 하며
ELK(ElasticSearch / Logstach / Kibana) 스택으로 사용되기도 한다

- LogStash 
  - 다양한 소스(DB, csv 파일 등)의 로그 또는 트랜젝션 데이터를 수집, 집계, 파싱하여 ElasticSearch로 전달
- ElasticSearch
  - Logstach로부터 받은 데이터를 검색 및 집계를 하여 필요한 관심있는 정보를 획득
- Kibana
  - ElasticSearch의 빠른 검색을 통해 데이터를 시각화 및 모니터링
  


## Elastic Search와 관계형 Database 비교

ElasticSearch는 왜 빠를까요? 그 이유는 inverted index (역색인)에 있다
- index : 책 앞에 있는 목차
- inverted index : 책 뒤에 있는 키워드 목차

Full text search에 강점

데이터가 기록되는 방법

|PK|Text|
|--|--|
|Doc 1| blue sky green red sun land|
|Doc 2 | blue ocean green land|
|Doc 3| red flower blue sky|

|text|Document|
|--|--|
|blue|Doc 1, Doc 2, Doc 3|
|sky|Doc 1, Doc 3|
|green|Doc 1, Doc 2|
|red|Doc 1, Doc 3|
|sun|Doc 1|
|land|Doc 1, Doc 2|
|ocean|Doc 2|
|flower|Doc 3|

|Elastic Search | RelationDB | 
|:--:|:--:|
|index|Database|
|Type|Table|
|Document|Row|
|Field|Column|
|Mapping|Schema|

| Elastic Search  | RelationDB   | CRUD|
|:---:|:---:| :--:|
| GET  | Select   | Read|
| PUT | Update | Update|
| POST | Insert | Create|
| DELETE | Delete| Delete|


| Elastic Search | RelationDB |
| :-- | :--|
|curl -XGET localhost:9200/classes/1|select * from class where id=1|
|curl -XPOST localhost:9200/classes/class/1 -d '{xxx}' | insert into class values (xxx)|
|curl -XPUT localhost:9200/classes/class/1 -d '{xxx}' | update class set xxx where id=1|
|curl -XDELETE localhost:9200/classes/class/1|delete from class where id=1|

## Ubuntu에 설치하기

- AWS EC2 instance 생성 (Ubuntu 18.04 기반)
- Install JDK
```bash
sudo apt install openjdk-8-jdk
```


### Elasticsearch Install

```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.9.1-amd64.deb
sudo dpkg -i elasticsearch-7.9.1-amd64.deb
```
- Install at /usr/share/elasticsearch

```
ubuntu@ip-172-31-0-227:~$ ls /usr/share/elasticsearch/
NOTICE.txt  README.asciidoc  bin  jdk  lib  modules  plugins
```

- Config files at /etc/elasticsearch

```

ubuntu@ip-172-31-0-227:~$ sudo ls /etc/elasticsearch/
elasticsearch.keystore  jvm.options    log4j2.properties  roles.yml  users_roles
elasticsearch.yml       jvm.options.d  role_mapping.yml   users
```

Elasticsearch의 동작 여부 확인

**curl -XGET localhost:9200**

```
ubuntu@ip-172-31-26-22:~$ curl -XGET localhost:9200
curl: (7) Failed to connect to localhost port 9200: Connection refused

ubuntu@ip-172-31-26-22:~$ sudo service elasticsearch start

ubuntu@ip-172-31-26-22:~$ curl -XGET localhost:9200
{
  "name" : "ip-172-31-26-22",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "xpIdagZdQXaIjHvoOlsAjw",
  "version" : {
    "number" : "7.9.1",
    "build_flavor" : "default",
    "build_type" : "deb",
    "build_hash" : "083627f112ba94dffc1232e8b42b73492789ef91",
    "build_date" : "2020-09-01T21:22:21.964974Z",
    "build_snapshot" : false,
    "lucene_version" : "8.6.2",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```



## ElasticSearch 기본 사용해보기

### Verify Index

**curl -XGET localhost:9200/classes**
```
ubuntu@ip-172-31-26-22:~$ curl -XGET localhost:9200/classes
{"error":{"root_cause":[{"type":"index_not_found_exception","reason":"no such index [classes]","resource.type":"index_or_alias","resource.id":"classes","index_uuid":"_na_","index":"classes"}],"type":"index_not_found_exception","reason":"no such index [classes]","resource.type":"index_or_alias","resource.id":"classes","index_uuid":"_na_","index":"classes"},"status":404}
```

결과를 좀 더 깔끔하게 보기 위해서
**curl -XGET localhost:9200/classes?pretty**

```
ubuntu@ip-172-31-26-22:~$ curl -XGET localhost:9200/classes?pretty
{
  "error" : {
    "root_cause" : [
      {
        "type" : "index_not_found_exception",
        "reason" : "no such index [classes]",
        "resource.type" : "index_or_alias",
        "resource.id" : "classes",
        "index_uuid" : "_na_",
        "index" : "classes"
      }
    ],
    "type" : "index_not_found_exception",
    "reason" : "no such index [classes]",
    "resource.type" : "index_or_alias",
    "resource.id" : "classes",
    "index_uuid" : "_na_",
    "index" : "classes"
  },
  "status" : 404
}
```

### Create Index

**curl -XPUT localhost:9200/classes**
```
ubuntu@ip-172-31-26-22:~$ curl -XPUT localhost:9200/classes?pretty
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "classes"
}
```
생성 후 다시 index를 확인해보면...
```
ubuntu@ip-172-31-26-22:~$ curl -XGET localhost:9200/classes?pretty
{
  "classes" : {
    "aliases" : { },
    "mappings" : { },
    "settings" : {
      "index" : {
        "creation_date" : "1599627991352",
        "number_of_shards" : "1",
        "number_of_replicas" : "1",
        "uuid" : "UOTpzaK0SCKcE2SQ0amEOQ",
        "version" : {
          "created" : "7090199"
        },
        "provided_name" : "classes"
      }
    }
  }
}
```

### Delete Index
**curl -XDELETE localhost:9200/classes**
```
ubuntu@ip-172-31-26-22:~$ curl -XDELETE localhost:9200/classes?pretty
{
  "acknowledged" : true
}
```

### Create Document

**curl -XPOST localhost:9200/classes/class/1 -d '{"title":"Algorithm", "professor":"John"}'**

**ElasticSearch v6.0부터는 Content Type의 체크가 강화되어 명시해줘야 함**
```
 ubuntu@ip-172-31-26-22:~$ curl -XPOST http://localhost:9200/classes/class/1/?pretty -d '{"title":"Algorithm", "professor":"John"}' -H 'Content-Type: application/json'                                        {
  "_index" : "classes",
  "_type" : "class",
  "_id" : "1",
  "_version" : 5,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 4,
  "_primary_term" : 1
}
```

### Kibana 설치

다운로드 : https://artifacts.elastic.co/downloads/kibana/kibana-7.9.1-amd64.deb
```

ubuntu@ip-172-31-26-22:~$ sudo dpkg -i kibana-7.9.1-amd64.deb
Selecting previously unselected package kibana.
(Reading database ... 107638 files and directories currently installed.)
Preparing to unpack kibana-7.9.1-amd64.deb ...
Unpacking kibana (7.9.1) ...
Setting up kibana (7.9.1) ...
Processing triggers for systemd (237-3ubuntu10.33) ...
Processing triggers for ureadahead (0.100.0-21) ...
```

### Config Kibana (/etc/kibana/kibana.yml)

```
#--------------------------------------------------
# 2. Bulk 색인
# 다량의 도큐먼트를 한꺼번에 색인 할 때는 반드시 bulk API를 사용
#--------------------------------------------------

POST my_index/_bulk
{"index":{"_id":1}}
{"message":"The quick brown fox"}
{"index":{"_id":2}}
{"message":"The quick brown fox jumps over the lazy dog"}
{"index":{"_id":3}}
{"message":"The quick brown fox jumps over the quick dog"}
{"index":{"_id":4}}
{"message":"Brown fox brown dog"}
{"index":{"_id":5}}
{"message":"Lazy jumping dog"}

#--------------------------------------------------
# 3. 풀텍스트 검색 (_search)
#--------------------------------------------------

# 3-1 인덱스의 전체 도큐먼트 검색 : match_all
GET my_index/_search
{
  "query":{
    "match_all":{ }
  }
}

GET my_index/_search
{
  "size" : 3,
  "query":{
    "match_all":{ }
  }
}


# 3-2 match 쿼리 : dog 검색
GET my_index/_search
{
  "query": {
    "match": {
      "message": "dog"
    }
  }
}

# 3-3 match 쿼리 : quick 또는 dog 검색 (or)
GET my_index/_search
{
  "query": {
    "match": {
      "message": "quick dog"
    }
  }
}

# 3-4 match 쿼리 : quick 과 dog 검색 (and)
GET my_index/_search
{
  "query": {
    "match": {
      "message": {
        "query": "quick dog",
        "operator": "and"
      }
    }
  }
}

# 3-5 match_phrase 쿼리 : "lazy dog" 구문 검색
GET my_index/_search
{
  "query": {
    "match_phrase": {
      "message": "lazy dog"
    }
  }
}


#--------------------------------------------------
# 4. 복합 쿼리 - bool 쿼리를 이용한 서브쿼리 조합
# - must : 쿼리가 참인 도큐먼트들을 검색
# - must_not : 쿼리가 거짓인 도큐먼트들을 검색
# - should : 검색 결과 중 이 쿼리에 해당하는 도큐먼트의 점수를 높임
# - filter : 쿼리가 참인 도큐먼트를 검색하지만 스코어를 계산하지 않음. must 보다 검색 속도가 빠르고 캐싱됨.
#--------------------------------------------------

# 4-1 "quick" 와 "lazy dog" 가 포함된 모든 문서 검색
GET my_index/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": { "message": "quick" }
        },
        {
          "match_phrase": { "message": "lazy dog" }
        }
      ]
    }
  }
}

# 4-2 "fox" 를 포함하는 모든 도큐먼트 중 "lazy" 가 포함된 결과에 가중치 부여
GET my_index/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "message": "fox"
          }
        }
      ],
      "should": [
        {
          "match": {
            "message": "lazy"
          }
        }
      ]
    }
  }
}

# 4-3 "fox" 와 "quick" 을 포함하는 쿼리의 must & filter 스코어 비교
GET my_index/_search
{
  "query": {
    "match": {
      "message": "fox"
    }
  }
}

GET my_index/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "message": "fox"
          }
        },
        {
          "match": {
            "message": "quick"
          }
        }
      ]
    }
  }
}

GET my_index/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "message": "fox"
          }
        }
      ],
      "filter": [
        {
          "match": {
            "message": "quick"
          }
        }
      ]
    }
  }
}

#--------------------------------------------------
# 5. range 쿼리
# - gte (Greater-than or equal to) : 이상 (같거나 큼)
#	- gt (Greater-than) : 초과 (큼)
#	- lte (Less-than or equal to) : 이하 (같거나 작음)
#	- lt (Less-than) : 미만 (작음)
#--------------------------------------------------
POST phones/_bulk
{"index":{"_id":1}}
{"model":"Samsung GalaxyS 5","price":475,"date":"2014-02-24"}
{"index":{"_id":2}}
{"model":"Samsung GalaxyS 6","price":795,"date":"2015-03-15"}
{"index":{"_id":3}}
{"model":"Samsung GalaxyS 7","price":859,"date":"2016-02-21"}
{"index":{"_id":4}}
{"model":"Samsung GalaxyS 8","price":959,"date":"2017-03-29"}
{"index":{"_id":5}}
{"model":"Samsung GalaxyS 9","price":1059,"date":"2018-02-25"}

# 5-1 price 필드 값이 700 이상, 900 미만인 데이터를 검색
GET phones/_search
{
  "query": {
    "range": {
      "price": {
        "gte": 700,
        "lt": 900
      }
    }
  }
}

# 5-2 date필드의 날짜가 2016년 1월 1일 이후인 도큐먼트들을 검색
GET phones/_search
{
  "query": {
    "range": {
      "date": {
        "gt": "2016-01-01"
      }
    }
  }
}

# 5-2 date필드의 날짜가 오늘 (2019년 6월 19일) 부터 2년 전 이후인 도큐먼트들을 검색
GET phones/_search
{
  "query": {
    "range": {
      "date": {
        "gt": "now-4y"
      }
    }
  }
}

#--------------------------------------------------
# 6. 텍스트 분석 - Analysis (_analyze API)
#--------------------------------------------------

# 6-1 Tokenizer 을 통해 문장을 검색어 텀(term)으로 쪼갬
GET my_index/_analyze
{
  "tokenizer": "standard",
  "text": "Brown fox brown dog"
}

# 6-2 Filter(토큰필터) 를 통해 쪼개진 텀들을 가공
# 6-2-1. lowercase - 소문자로 변경
GET my_index/_analyze
{
  "tokenizer": "standard",
  "filter": [
    "lowercase"
  ],
  "text": "Brown fox brown dog"
}

# 6-2-2. unique - 중복 텀 제거
GET my_index/_analyze
{
  "tokenizer": "standard",
  "filter": [
    "lowercase",
    "unique"
  ],
  "text": "Brown brown brown fox brown dog"
}

# 6-3 (Tokenizer + Filter) 대신 Analyzer 사용
GET my_index/_analyze
{
  "analyzer": "standard",
  "text": "Brown fox brown dog"
}

#--------------------------------------------------
# 7. 분석 과정 이해하기
#--------------------------------------------------

# 7-1 복합적인 문장 분석 - T:standard, F:lowercase
GET my_index/_analyze
{
  "tokenizer": "standard",
  "filter": [
    "lowercase"
  ],
  "text": "THE quick.brown_FOx jumped! $19.95 @ 3.0"
}

# 7-2 복합적인 문장 분석 - T:letter, F:lowercase
GET my_index/_analyze
{
  "tokenizer": "letter",
  "filter": [
    "lowercase"
  ],
  "text": "THE quick.brown_FOx jumped! $19.95 @ 3.0"
}

# 7-3 Email, URL 분석 - T:standard
GET my_index/_analyze
{
  "tokenizer": "standard",
  "text": "elastic@example.com website: https://www.elastic.co"
}

# 7-4 Email, URL 분석 - T:uax_url_email
GET my_index/_analyze
{
  "tokenizer": "uax_url_email",
  "text": "elastic@example.com website: https://www.elastic.co"
}

# 8-5 한글 형태소 분석기 nori 설치
# $ bin/elasticsearch-plugin install analysis-nori

# 8-6 nori_tokenizer 를 이용한 한글 분석
GET _analyze
{
  "tokenizer": "standard",
  "text": ["동해물과 백두산이"]
}

GET _analyze
{
  "tokenizer": "nori_tokenizer",
  "text": ["동해물과 백두산이"]
}


#--------------------------------------------------
# 8. 인덱스 생성
# - settings : analyzer, 샤드 수, 리프레시 주기 등을 설정
# - mappings : 각 필드별 데이터 명세를 정의
#--------------------------------------------------

# 8-1 사용자 정의 analyzer
PUT my_index_2
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "letter",
          "filter": [
            "lowercase",
            "stop"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "message": {
        "type": "text",
        "analyzer": "my_analyzer"
      }
    }
  }
}

# 8-2 사용자 정의 analyzer 필드에 데이터 색인
PUT my_index_2/_doc/1
{
  "message": "THE quick.brown_FOx jumped! $19.95 @ 3.0"
}

# 8-3 데이터 검색
GET my_index_2/_search
{
  "query": {
    "match": {
      "message": "brown"
    }
  }
}

PUT my_index_3/_doc/1
{
  "message": "THE quick.brown_FOx jumped! $19.95 @ 3.0"
}

GET my_index_3/_search
{
  "query": {
    "match": {
      "message": "brown"
    }
  }
}

GET my_index_3/_search
{
  "query": {
    "match": {
      "message": "quick.brown_FOx"
    }
  }
}



GET my_index_2/_search
{
  "query": {
    "match": {
      "message": "the"
    }
  }
}

#--------------------------------------------------
# 9. 애그리게이션 - 집계 (Aggregation)
# - metrics : min, max, sum, avg 등의 계산
# - bucket : 특정 기준으로 도큐먼트들을 그룹화
#--------------------------------------------------
PUT my_stations/_bulk
{"index": {"_id": "1"}}
{"date": "2019-06-01", "line": "1호선", "station": "종각", "passangers": 2314}
{"index": {"_id": "2"}}
{"date": "2019-06-01", "line": "2호선", "station": "강남", "passangers": 5412}
{"index": {"_id": "3"}}
{"date": "2019-07-10", "line": "2호선", "station": "강남", "passangers": 6221}
{"index": {"_id": "4"}}
{"date": "2019-07-15", "line": "2호선", "station": "강남", "passangers": 6478}
{"index": {"_id": "5"}}
{"date": "2019-08-07", "line": "2호선", "station": "강남", "passangers": 5821}
{"index": {"_id": "6"}}
{"date": "2019-08-18", "line": "2호선", "station": "강남", "passangers": 5724}
{"index": {"_id": "7"}}
{"date": "2019-09-02", "line": "2호선", "station": "신촌", "passangers": 3912}
{"index": {"_id": "8"}}
{"date": "2019-09-11", "line": "3호선", "station": "양재", "passangers": 4121}
{"index": {"_id": "9"}}
{"date": "2019-09-20", "line": "3호선", "station": "홍제", "passangers": 1021}
{"index": {"_id": "10"}}
{"date": "2019-10-01", "line": "3호선", "station": "불광", "passangers": 971}

# 9-1 전체 passangers 필드값의 합계를 가져오는 metrics aggregation
GET my_stations/_search
{
  "size": 0, 
  "aggs": {
    "all_passangers": {
      "sum": {
        "field": "passangers"
      }
    }
  }
}

# 9-2 "station": "강남" 인 도큐먼트의 passangers 필드값의 합계를 가져오는 metrics aggregation
GET my_stations/_search
{
  "query": {
    "match": {
      "station": "강남"
    }
  }, 
  "size": 0, 
  "aggs": {
    "gangnam_passangers": {
      "sum": {
        "field": "passangers"
      }
    }
  }
}

# 9-3 date_histogram으로 date 필드를 1개월 간격으로 구분하는 bucket aggregation
GET my_stations/_search
{
  "size": 0,
  "aggs": {
    "date_his": {
      "date_histogram": {
        "field": "date",
        "interval": "month"
      }
    }
  }
}

# 9-4 stations.keyword 필드 별로 passangers 필드의 평균값을 계산하는 bucket & metrics aggregation
GET my_stations/_search
{
  "size": 0,
  "aggs": {
    "stations": {
      "terms": {
        "field": "station.keyword"
      }
      , "aggs": {
        "avg_psg_per_st": {
          "avg": {
            "field": "passangers"
          }
        }
      }
    }
  }
}

#--------------------------------------------------
# 10. Geo - 위치정보
# - geo_point : { "lat": 41.12, "lon": -71.34 } 같은 형식으로 입력
#--------------------------------------------------

# 10-1 geo_point 타입의 location 필드 선언
PUT my_geo
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_point"
      }
    }
  }
}

# 10-2 예제 데이터 입력
PUT my_geo/_bulk
{"index": {"_id": "1"}}
{"station": "강남", "location": {"lon": 127.027926, "lat":37.497175 }, "line": "2호선"}
{"index": {"_id": "2"}}
{"station": "종로3가", "location": {"lon":126.991806, "lat":37.571607}, "line": "3호선"}
{"index": {"_id": "3"}}
{"station": "여의도", "location": {"lon":126.924191, "lat":37.521624}, "line": "5호선"}
{"index": {"_id": "4"}}
{"station": "서울역", "location": {"lon":126.972559, "lat":37.554648}, "line": "1호선"}

# 10-3 geo_bounding_box : 두 점을 기준으로 하는 네모 안에 있는 도큐먼트들을 가져옴
GET my_geo/_search
{
  "query": {
    "geo_bounding_box": {
      "location": {
        "bottom_right": {
          "lat": 37.4899,
          "lon": 127.0388
        },
        "top_left": {
          "lat": 37.5779,
          "lon": 126.9617
        }
      }
    }
  }
}

# 10-4 geo_distance : 한 점을 기준으로 반경 안에 있는 도큐먼트들을 가져옴
GET my_geo/_search
{
  "query": {
    "geo_distance": {
      "distance": "5km",
      "location": {
        "lat": 37.5358,
        "lon": 126.9559
      }
    }
  }
}
```
