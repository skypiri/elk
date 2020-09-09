# ELK search

## Ubuntu에 설치하기

- AWS EC2 instance 생성 (Ubuntu 18.04 기반)
- Install JDK
```bash
sudo apt install openjdk-8-jdk
```


## Elasticsearch Install

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

## Elastic Search와 관계형 Database 비교

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

** curl -XPUT localhost:9200/classes **
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
** curl -XDELETE localhost:9200/classes **
```
ubuntu@ip-172-31-26-22:~$ curl -XDELETE localhost:9200/classes?pretty
{
  "acknowledged" : true
}
```




