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


