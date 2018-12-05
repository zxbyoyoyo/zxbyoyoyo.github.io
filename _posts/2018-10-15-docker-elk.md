---
title: Docker builds elk
key: 20181015
tags: docker
aside:
  toc: true
excerpt_separator: <!--more-->
excerpt_type: text # text (default), html
---




ELK是开源日志界的三大剑客，本文主要讲怎么在docker里头跑起来这一套东东。
<!--more-->
## 镜像 ##

这里采用[docker-elk](https://github.com/deviantony/docker-elk)的镜像。

## 运行 ##

```
cd docker-elk
eval "$(docker-machine env default)"
docker-compose up -d
...
Successfully built 68f1e0777077
Creating dockerelk_kibana_1
Attaching to dockerelk_elasticsearch_1, dockerelk_logstash_1, dockerelk_kibana_1
kibana_1        | Stalling for Elasticsearch
kibana_1        | Starting Kibana
elasticsearch_1 | [2016-02-03 12:01:46,067][INFO ][node                     ] [Caretaker] version[2.2.0], pid[1], build[8ff36d1/2016-01-27T13:32:39Z]
elasticsearch_1 | [2016-02-03 12:01:46,068][INFO ][node                     ] [Caretaker] initializing ...
elasticsearch_1 | [2016-02-03 12:01:46,615][INFO ][plugins                  ] [Caretaker] modules [lang-expression, lang-groovy], plugins [], sites []
elasticsearch_1 | [2016-02-03 12:01:46,635][INFO ][env                      ] [Caretaker] using [1] data paths, mounts [[/usr/share/elasticsearch/data (/dev/sda1)]], net usable_space [14gb], net total_space [18.1gb], spins? [possibly], types [ext4]
elasticsearch_1 | [2016-02-03 12:01:46,635][INFO ][env                      ] [Caretaker] heap size [1015.6mb], compressed ordinary object pointers [true]
elasticsearch_1 | [2016-02-03 12:01:49,038][INFO ][node                     ] [Caretaker] initialized
elasticsearch_1 | [2016-02-03 12:01:49,040][INFO ][node                     ] [Caretaker] starting ...
elasticsearch_1 | [2016-02-03 12:01:49,120][INFO ][transport                ] [Caretaker] publish_address {172.17.0.6:9300}, bound_addresses {[::]:9300}
elasticsearch_1 | [2016-02-03 12:01:49,130][INFO ][discovery                ] [Caretaker] elasticsearch/jOBlX_T5TYmgeE6jHP-z0Q
elasticsearch_1 | [2016-02-03 12:01:52,207][INFO ][cluster.service          ] [Caretaker] new_master {Caretaker}{jOBlX_T5TYmgeE6jHP-z0Q}{172.17.0.6}{172.17.0.6:9300}, reason: zen-disco-join(elected_as_master, [0] joins received)
elasticsearch_1 | [2016-02-03 12:01:52,250][INFO ][http                     ] [Caretaker] publish_address {172.17.0.6:9200}, bound_addresses {[::]:9200}
elasticsearch_1 | [2016-02-03 12:01:52,251][INFO ][node                     ] [Caretaker] started
elasticsearch_1 | [2016-02-03 12:01:52,259][INFO ][gateway                  ] [Caretaker] recovered [0] indices into cluster_state
kibana_1        | {"type":"log","@timestamp":"2016-02-03T13:21:44+00:00","tags":["warning","config"],"pid":1,"key":"bundled_plugin_ids","val":["plugins/dashboard/index","plugins/discover/index","plugins/doc/index","plugins/kibana/index","plugins/markdown_vis/index","plugins/metric_vis/index","plugins/settings/index","plugins/table_vis/index","plugins/vis_types/index","plugins/visualize/index"],"message":"Settings for \"bundled_plugin_ids\" were not applied, check for spelling errors and ensure the plugin is loaded."}
kibana_1        | {"type":"log","@timestamp":"2016-02-03T13:21:44+00:00","tags":["status","plugin:sense","info"],"pid":1,"name":"plugin:sense","state":"green","message":"Status changed from uninitialized to green - Ready","prevState":"uninitialized","prevMsg":"uninitialized"}
kibana_1        | {"type":"log","@timestamp":"2016-02-03T13:21:44+00:00","tags":["status","plugin:kibana","info"],"pid":1,"name":"plugin:kibana","state":"green","message":"Status changed from uninitialized to green - Ready","prevState":"uninitialized","prevMsg":"uninitialized"}
kibana_1        | {"type":"log","@timestamp":"2016-02-03T13:21:44+00:00","tags":["status","plugin:elasticsearch","info"],"pid":1,"name":"plugin:elasticsearch","state":"yellow","message":"Status changed from uninitialized to yellow - Waiting for Elasticsearch","prevState":"uninitialized","prevMsg":"uninitialized"}
kibana_1        | {"type":"log","@timestamp":"2016-02-03T13:21:44+00:00","tags":["status","plugin:kbn_vislib_vis_types","info"],"pid":1,"name":"plugin:kbn_vislib_vis_types","state":"green","message":"Status changed from uninitialized to green - Ready","prevState":"uninitialized","prevMsg":"uninitialized"}
kibana_1        | {"type":"log","@timestamp":"2016-02-03T13:21:44+00:00","tags":["status","plugin:markdown_vis","info"],"pid":1,"name":"plugin:markdown_vis","state":"green","message":"Status changed from uninitialized to green - Ready","prevState":"uninitialized","prevMsg":"uninitialized"}
kibana_1        | {"type":"log","@timestamp":"2016-02-03T13:21:44+00:00","tags":["status","plugin:metric_vis","info"],"pid":1,"name":"plugin:metric_vis","state":"green","message":"Status changed from uninitialized to green - Ready","prevState":"uninitialized","prevMsg":"uninitialized"}
kibana_1        | {"type":"log","@timestamp":"2016-02-03T13:21:44+00:00","tags":["status","plugin:spyModes","info"],"pid":1,"name":"plugin:spyModes","state":"green","message":"Status changed from uninitialized to green - Ready","prevState":"uninitialized","prevMsg":"uninitialized"}
kibana_1        | {"type":"log","@timestamp":"2016-02-03T13:21:44+00:00","tags":["status","plugin:statusPage","info"],"pid":1,"name":"plugin:statusPage","state":"green","message":"Status changed from uninitialized to green - Ready","prevState":"uninitialized","prevMsg":"uninitialized"}
kibana_1        | {"type":"log","@timestamp":"2016-02-03T13:21:44+00:00","tags":["status","plugin:table_vis","info"],"pid":1,"name":"plugin:table_vis","state":"green","message":"Status changed from uninitialized to green - Ready","prevState":"uninitialized","prevMsg":"uninitialized"}
kibana_1        | {"type":"log","@timestamp":"2016-02-03T13:21:44+00:00","tags":["listening","info"],"pid":1,"message":"Server running at http://0.0.0.0:5601"}
kibana_1        | {"type":"log","@timestamp":"2016-02-03T13:21:49+00:00","tags":["status","plugin:elasticsearch","info"],"pid":1,"name":"plugin:elasticsearch","state":"yellow","message":"Status changed from yellow to yellow - No existing Kibana index found","prevState":"yellow","prevMsg":"Waiting for Elasticsearch"}
elasticsearch_1 | [2016-02-03 13:21:50,104][INFO ][cluster.metadata         ] [Caretaker] [.kibana] creating index, cause [api], templates [], shards [1]/[1], mappings [config]
elasticsearch_1 | [2016-02-03 13:21:50,489][INFO ][cluster.routing.allocation] [Caretaker] Cluster health status changed from [RED] to [YELLOW] (reason: [shards started [[.kibana][0]] ...]).
kibana_1        | {"type":"log","@timestamp":"2016-02-03T13:21:53+00:00","tags":["status","plugin:elasticsearch","info"],"pid":1,"name":"plugin:elasticsearch","state":"green","message":"Status changed from yellow to green - Kibana index ready","prevState":"yellow","prevMsg":"No existing Kibana index found"}
```

## 查看kibana ##

[http://192.168.99.100](http://192.168.99.100/):5601/
![图片描述](https://i.loli.net/2018/12/03/5c052fe0efa20.jpg)

## 查看sense ##

[http://192.168.99.100](http://192.168.99.100/):5601/app/sense
![图片描述](https://i.loli.net/2018/12/03/5c052fe327e6c.jpg)

## 默认端口 ##

- 5000: Logstash TCP input.
- 9200: Elasticsearch HTTP
- 9300: Elasticsearch TCP transport
- 5601: Kibana