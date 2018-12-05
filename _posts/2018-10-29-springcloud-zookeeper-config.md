---
title: Use zookeeper to manage project configuration
key: 20181029
tags: ["spring boot"]
aside:
  toc: true
excerpt_separator: <!--more-->
excerpt_type: text # text (default), html
---

摘要： 为什么要使用zookeeper来管理项目配置？ 每当我们的项目需要更改配置文件的时候，我们都会重新打包并重新部署项目。如果一个项目还好说，如果是分布式的服务，那么我们需要停掉左右机器上的服务，上传新的包，然后重启。
<!--more-->
为什么要使用zookeeper来管理项目配置？

    每当我们的项目需要更改配置文件的时候，我们都会重新打包并重新部署项目。如果一个项目还好说，如果是分布式的服务，那么我们需要停掉左右机器上的服务，上传新的包，然后重启。如果是10台，如果是100台如果是1000台.......
    基于zookeeper配置中心，我们只需要更改zk中相应项目的配置，直接重启服务就可以了，省去了冲洗打包，上传，部署等大量操作。且配置中心化易于管理。

准备工作

    基于SpringBoot
    基于SpringCloud
    基于zooKeeper 集群服务

假设你是基于SpringBoot开发，因为Spring对zk的集成整合在了SpringCloud的子项目中，所以我们需要引入SpringCloud，点击链接参考官方文档配置你的pom.xml文件，我使用的是目前Dalston SR2这个release。

然后在pom.xml的依赖中引入如下依赖：

    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-zookeeper-config</artifactId>
    </dependency>
    <!-- zookeeper 依赖的健康检查的jar，所以需要引入actuator这个依赖 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

Spring官网对zookeeper这个子项目有个说明，就是项目的配置文件不可以叫application，需要改成bootstrap开头，否则没办法正常读取配置。例如原先叫application.yaml / application.properties，现在改成bootstrap.yaml / bootstrap.properties。

配置ZooKeeper

    #ZooKeeper的连接字符串，如果是集群，逗号分隔节点，格式：ip:port[,ip2:port2,.....]
    spring.cloud.zookeeper.connect-string = ${ZOOKEEPER_CONNECT_STRING}
    #指定zookeeper目录的根目录
    spring.cloud.zookeeper.config.root = <ZK_directory>
    #启用zk的配置
    spring.cloud.zookeeper.config.enabled =  <true | false>
     #定义了你的项目的名称，zk会在你指定的
    根目录下寻找以这个项目名命名的目录下的配置
    spring.application.name = <Define Name>

关于如何在zookeeper下创建目录，并填写配置，请参考其他资料。

举例说明

    spring.cloud.zookeeper.config.enabled = true

在你指定zookeeper下的根目录，zk会在这个根目录下寻找跟你项目名称相同的文件夹，然后找到其下面的所有配置。
例如指定的根目录root（即/root），项目名称叫project,（即/root/project），上面的配置对应就是 /root/project/spring/cloud/zookeeper/config/enabled这个key,enabled的值是true

启动项目即可


参考：
https://blog.csdn.net/CSDN_Stephen/article/details/78856323
https://blog.csdn.net/timedifier2/article/details/53542172
https://cloud.spring.io/spring-cloud-zookeeper/single/spring-cloud-zookeeper.html#spring-cloud-zookeeper-config
