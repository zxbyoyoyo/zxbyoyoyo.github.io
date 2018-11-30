---
title: Elastic单机多节点集群搭建
key: 20181110
tags: Elasticsearch
aside:
  toc: true
excerpt_separator: <!--more-->
excerpt_type: text # text (default), html
---


最近在学习ES相关内容，为了方便自己使用，在本地虚拟机上搭建了一个3节点的ES集群，在搭建过程中，遇到了许多坑，网上的资料也比较分散，所以详细整理一下搭建过程发出来供参考。
<!--more-->
#### 一、linux java环境配置

1. ######  下载jdk1.8 for linux

   ```shell
   [root@es ~] mkdir -pv /home/software
   [root@es ~] cd /home/software/
   [root@es software] wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u191-b12/2787e4a523244c269598db4e85c51e0c/jdk-8u191-linux-x64.tar.gz
   ```

2. ######  解压安装jdk

   ```shell
   [root@es software] tar xvf jdk-8u191-linux-x64.tar.gz
   ```

3. ######  配置环境变量

    vim /etc/profile文件，在最后添加如下几行

    ```shell
    [root@es software] vim /etc/profile
           export JAVA_HOME=/home/software/jdk1.8.0_191
           export PATH=PATH:JAVA_HOME/bin
           export CLASSPATH=.:JAVA_HOME/jre/lib:JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar  [root@es software] source /etc/profile
    ```

4. **配置符号连接**

   删除原有符号连接

   ```sh
   [root@es software]# rm  -rf  /usr/bin/java
   [root@es software]# rm  -rf  /usr/bin/javac
   ```

   创建新符号连接

   ```shell
   [root@es software]# ln -s /home/software/jdk1.8.0_191/bin/java  /usr/bin/java
   [root@es software]# ln -s /home/software/jdk1.8.0_191/bin/javac  /usr/bin/javac
   ```

5. **查看是否创建成功**

   ```shell
   [root@es software]# java -version
   java version "1.8.0_191"
   Java(TM) SE Runtime Environment (build 1.8.0_191-b12)
   Java HotSpot(TM) 64-Bit Server VM (build 25.191-b12, mixed mode)
   ```

#### 二、系统环境配置

不修改系统配置情况下，直接启动es会出现下面两个异常：

1. 异常1：max virutal memory areas vm.max_map_count [65530] is too low, increase to at least [262144]解决：切换到root用户，修改操作系统的内核配置文件sysctl.conf

   ```shell
   [root@es software] vim /etc/sysctl.conf
   #在配置文件最后面添加如下内容
   vm.max_map_count=655360

   [root@es software] sysctl -p #使修改之后的配置文件生效
   ```

   解释：max_map_count文件包含限制一个进程可以拥有的VMA(虚拟内存区域)的数量。虚拟内存区域是一个连续的虚拟地址空间区域。
   在进程的生命周期中，每当程序尝试在内存中映射文件，链接到共享内存段，或者分配堆空间的时候，这些区域将被创建。
   当进程达到了VMA上线但又只能释放少量的内存给其他的内核进程使用时，操作系统会抛出内存不足的错

2. 异常2：max number of threads [3750] for user [xxx] is too low, increase to at least [4096]

   解决：切换到root用户，修改limits.conf

   ```shell
   [root@es software] vim /etc/security/limits.conf
   # 在文件末尾添加如下内容:
     * soft nofile 65536
     * hard nofile 131072
     * soft nproc 4096
     * hard nproc 4096
   ```

   解释：limits.conf:用来保护系统的资源访问，和sysctl.conf很像，但是limits.conf是针对于用户，而sysctl.conf是针对于操作系统。


#### 三、Elasticsearch 集群搭建

1. **下载elasticsearch for linux**

   ```shell
   [root@es software] cd /home/software/
   [root@es software] wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.4.3.zip
   ```

2. **解压到主从目录**

   ```shell
   [root@es software] mkdir -pv /home/software/elk/{es_master,es_slave1,es_slave2}
   [root@es software] echo /home/software/elk/es_master /home/software/elk/es_slave1 /home/software/elk/es_slave2  | xargs -n 1 cp -r -v /home/software/elasticsearch-6.4.3/.
   ```


3. **创建es主从节点的数据目录和日志目录**

   ```shell
   [root@es software] mkdir -pv /data/elk/{es_master,es_slave1,es_slave2}/{data,logs}
   ```

4. **创建一个es用户并授权相关目录**

   ```shell
   [root@es software] useradd elk
   [root@es software] passwd elk

   [root@es software] chown -R elk:elk /home/software/elk/{es_master,es_slave1,es_slave2}
   [root@es software] chown -R elk:elk /data/elk/{es_master,es_slave1,es_slave2}
   ```

5. **修改主从节点的 jvm.options**

   ```shell
   [root@es software] vim /home/software/elk/es_master/config/jvm.options
   	#修改如下两个选项：
       -Xms512m  #elasticsearch启动时jvm所分配的初始堆内存大小
       -Xmx512m  #elasticsearch启动之后允许jvm分配的最大堆内存大小，生产环境中可能需要调大
   	#注意：如果内存足够大，可以不用修改，默认为1G
   [root@es software] vim /home/software/elk/es_slave1/config/jvm.options
       -Xms512m
       -Xmx512m
   [root@es software] vim /home/software/elk/es_slave2/config/jvm.options
       -Xms512m
       -Xmx512m
   ```

6. **修改主从节点的配置文件elasticsearch.yml**

   ```shell
   [root@es software]  vim /home/software/elk/es_master/config/elasticsearch.yml
       cluster.name: elasticsearch #表示集群标识，同一个集群中的多个节点使用相同的标识
       node.name: "es-master" #节点名称
       path.data: /data/elk/es_master/data #数据存储目录
       path.logs: /data/elk/es_master/logs #日志目录

       network.host: 192.168.188.100 # 对外访问ip
       network.bind_host: 192.168.188.100
       network.publish_host: 192.168.188.100
       http.port: 9200    #对外访问端口
   	transport.tcp.port: 9300    #各节点通信
   	#设置集群中master节点的初始列表，可以通过这些节点来自动发现新加入集群的节点
       discovery.zen.ping.unicast.hosts: ["192.168.188.100:9300","192.168.188.100:9301","192.168.188.100:9302"]
       #设置这个参数来保证集群中的节点可以知道其它N个有master资格的节点.默认为1,对于大的集群来说,可以设置大一点的值(2-4),对于配置node.master为false的节点启动后不会作为主节点的候选
       discovery.zen.minimum_master_nodes: 1

       bootstrap.memory_lock: false #服务启动的时候锁定足够的内存，防止数据写入swap
       bootstrap.system_call_filter: false

   ##--------------------------------------------------------------------------------
   [root@es software]  vim /home/software/elk/es_slave1/config/elasticsearch.yml

       cluster.name: elasticsearch
       node.name: "es-slave1" ]
       path.data: /data/elk/es_slave1/data
       path.logs: /data/elk/es_slave1/logs

       network.host: 192.168.188.100
       network.bind_host: 192.168.188.100
       network.publish_host: 192.168.188.100
       http.port: 9201    
   	transport.tcp.port: 9301  
       discovery.zen.ping.unicast.hosts: ["192.168.188.100:9300","192.168.188.100:9301","192.168.188.100:9302"]
       discovery.zen.minimum_master_nodes: 1
       bootstrap.memory_lock: false
       bootstrap.system_call_filter: false

   [root@es software]  vim /home/software/elk/es_slave2/config/elasticsearch.yml
       cluster.name: elasticsearch
       node.name: "es-slave2"
       node.master: false
       path.data: /data/elk/es_slave2/data
       path.logs: /data/elk/es_slave2/logs

       network.host: 192.168.188.100
       network.bind_host: 192.168.188.100
       network.publish_host: 192.168.188.100
       http.port: 9202    
   	transport.tcp.port: 9302  
       discovery.zen.ping.unicast.hosts: ["192.168.188.100:9300","192.168.188.100:9301","192.168.188.100:9302"]
       discovery.zen.minimum_master_nodes: 1
       bootstrap.memory_lock: false
       bootstrap.system_call_filter: false
   ```



#### 四、启动ES集群

1. 切换到elk用户，并启动各节点

   ```shell
   [root@es software] cd /home/software/elk/es_master
   [root@es es_master] ./bin/elasticsearch -d #-d参数表示以后台进程启动，默认情况下会在控制台输出日志

   [root@es software] cd /home/software/elk/es_slave1
   [root@es es_slave1] ./bin/elasticsearch -d

   [root@es software] cd /home/software/elk/es_slave2
   [root@es es_slave2] ./bin/elasticsearch -d

   ```

2. 看是否启动成功

   ```shell
   [root@es ~] ps -ef| grep elasticsearch
   [root@es ~] lsof -i:9200
   [root@es ~] lsof -i:9201
   [root@es ~] lsof -i:9202
   注意：启动异常时，可进入elasticsearch.yml中配置的日志目录，查看相关的日志信息
   ```

3. 测试是否可以访问

   ```shell
   curl 192.168.0.12:9200
   curl 192.168.0.12:9201
   curl 192.168.0.12:9202
   ```

​       验证完毕之后，ES集群就启动完毕。

#### 五、安装及配置ES前端图形化操作工具

- 下载kibana

  ```shell
  [root@es-master software] wget https://artifacts.elastic.co/downloads/kibana/kibana-6.4.3-linux-x86_64.tar.gz
  ```

- 解压

  ```shell
  [root@es-master software] tar xvf kibana-6.4.3-linux-x86_64.tar.gz
  [root@es-master software] cd /usr/local
  [root@es-master local]# mv kibana-6.4.0-linux-x86_64 kibana-6.4.0
  ```

- 修改kibana的配置文件kibana.yml

  ```
  [root@es-master local]# cd kibana-6.4.0/config
  [root@es-master config]# vim kibana.yml
      #对外服务监听端口
      server.port: 5601
      #绑定可以访问5601端口服务的IP地址，如果设置为0.0.0.0表示接收任何地址的请求
      server.host: "192.168.188.100"
      #用来处理ES请求的服务URL
      elasticsearch.url: "http://192.168.0.11:9200"
  ```

- 启动kibana

  ```shell
  [root@es-master config]# cd /usr/local/kibana-6.4.0/bin
  #以后台进程启动，kibana默认是控制台方式启动，Ctrl+C就会退出
  [root@es-master bin]# nohup ./kibana &
  #查看日志是否启动正常
  [root@es-master bin]# tail -f nohup.out
  ```

  注意：

  `elasticsearch.url`虽然格式上不支持的填写多个es节点地址，但是官方也给出了另外一种方案：搭建一个只用来“协调”的es节点，让这个节点加入到es集群中，然后kibana连接这个“协调”节点，这个“协调”节点，不参加主节点选举，也不存储数据，只是用来处理传入的HTTP请求，并将操作重定向到集群中的其他es节点，然后收集并返回结果。这个“协调”节点本质上也起了一个负载均衡的作用。<https://www.elastic.co/guide/en/kibana/6.4/production.html#load-balancing>

#### 六、ElasticSearch和kibana的停止

- **停止ES服务**

  ```shell
  [root@es-master bin]# ps -ef| grep elasticsearch | grep -v grep | awk '{print $2}'
  [root@es-master bin]# kill -9 pid[上一步所输出的pid]
  ```

- **停止Kibana服务**

  ```shell
  [root@es-master bin]# ps -ef| grep node | grep -v grep | grep -v elasticsearch | awk '{print $2}'
  [root@es-master bin]# kill -9 pid[上一步所输出的pid]
  ```

 参考：

​    linux Vmware虚拟机设置静态IP地址[参考1](https://www.cnblogs.com/jsonhc/p/7685393.html)、[参考2](https://www.cnblogs.com/zhanjindong/p/3250393.html)

   《死磕 Elasticsearch 方法论》https://blog.csdn.net/laoyang360/article/details/79293493
   《npm 中文文档》https://www.npmjs.com.cn/

   《 ELK & ElasticSearch 5.1 基础概念及配置文件详解》 https://blog.csdn.net/zxf_668899/article/details/54582849

   ElasticSearch6.4.0集群搭建 https://segmentfault.com/a/1190000016326389