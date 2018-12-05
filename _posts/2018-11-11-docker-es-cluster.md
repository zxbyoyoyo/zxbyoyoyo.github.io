---
title: Docker Compose builds Elasticsearch cluster
key: 20181111
tags: docker
aside:
  toc: true
excerpt_separator: <!--more-->
excerpt_type: text # text (default), html
---



使用Docker进行Elasticsearch集群搭建非常简单。下面将展示 如何快速简便地在docker上运行3节点elasticsearch集群进行测试。
<!--more-->

## 先决条件 ##

##### 系统配置 #####

需要设置`vm.max_map_count`内核参数：

```
$ sudo sysctl -w vm.max_map_count=262144 
```

要永久设置，请将其添加到`/etc/sysctl.conf`并重新加载`sudo sysctl -p`

##### docker-compose文件 #####

编辑文件docker-compose-v1.yml：

```yml
version: '2.2'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.2.4
    container_name: elasticsearch
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - http.cors.enabled=true
      - http.cors.allow-origin=*
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata1:/home/ruan/workspace/docker/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - esnet
  elasticsearch2:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.2.4
    container_name: elasticsearch2
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - http.cors.enabled=true
      - http.cors.allow-origin=*
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "discovery.zen.ping.unicast.hosts=elasticsearch"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata2:/home/ruan/workspace/docker/elasticsearch/data
    networks:
      - esnet
  elasticsearch3:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.2.4
    container_name: elasticsearch3
    environment:
      - cluster.name=docker-cluster
      - bootstrap.memory_lock=true
      - http.cors.enabled=true
      - http.cors.allow-origin=*
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "discovery.zen.ping.unicast.hosts=elasticsearch"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata3:/home/ruan/workspace/docker/elasticsearch/data
    networks:
      - esnet

  kibana:
    image: 'docker.elastic.co/kibana/kibana:6.3.2'
    container_name: kibana
    environment:
      SERVER_NAME: kibana.local
      ELASTICSEARCH_URL: http://elasticsearch:9200
    ports:
      - '5601:5601'
    networks:
      - esnet

  headPlugin:
    image: 'mobz/elasticsearch-head:5'
    container_name: head
    ports:
      - '9100:9100'
    networks:
      - esnet

volumes:
  esdata1:
    driver: local
  esdata2:
    driver: local
  esdata3:
    driver: local

networks:
  esnet:
```

现在确保我们在compose涉及到的路径存在，我的例子里是`/home/ruan/workspace/docker/elasticsearch/data`

## 创建Elasticsearch集群 ##

使用docker compose部署elasticsearch集群：

```shell
docker-compose -f docker-compose-v1.yml up -d
```

这将在前台运行，可以在控制台看到输出。

## 测试Elasticsearch ##

让我们运行几个查询，首先检查集群运行状况api：

```shell
$ curl http://127.0.0.1:9200/_cluster/health?pretty
{
  "cluster_name" : "docker-cluster",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 3,
  "active_primary_shards" : 1,
  "active_shards" : 2,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}
```

创建副本为2的索引test：

```shell
$ curl -H "Content-Type: application/json" -XPUT http://127.0.0.1:9200/test -d '{"number_of_replicas": 2}' 
```

将文档索引到elasticsearch：

```shell
$ curl -H "Content-Type: application/json" -XPUT http://127.0.0.1:9200/test/docs/1 -d '{"name": "ruan"}'
{"_index":"test","_type":"docs","_id":"1","_version":1,"result":"created","_shards":{"total":3,"successful":3,"failed":0},"_seq_no":0,"_primary_term":1}
```

查看索引：

```
$ curl http://127.0.0.1:9200/_cat/indices?v health status index                       uuid                   pri rep docs.count docs.deleted store.size pri.store.size green  open   test                        w4p2Q3fTR4uMSYBfpNVPqw   5   2          1            0      3.3kb          1.1kb green  open   .monitoring-es-6-2018.04.29 W69lql-rSbORVfHZrj4vug   1   1       1601           38        4mb            2mb 
```

## 测试Kibana ##

Kibana也包含在堆栈中，可通过[http://localhost:5601/访问](http://localhost:5601/)：

![img](https://objects.ruanbekker.com/assets/images/kibana-local-home.png)

## 测试Elasticsearch Head UI ##

我一般喜欢直接使用RESTFul API，但是如果你想使用UI与Elasticsearch进行交互，你可以通过[http://localhost:9100/](http://localhost:9100/)访问它，它应该如下所示：

![img](https://objects.ruanbekker.com/assets/images/elasticsearch-head-ui.png)

## 删除群集： ##

当它在前台运行时，你可以点击ctrl + c并且当我们在我们的compose中持久化数据时，当你再次启动集群时，数据仍将存在。

## 参考： ##

https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html

https://markheath.net/post/exploring-elasticsearch-with-docker