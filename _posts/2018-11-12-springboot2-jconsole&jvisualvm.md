---
title: Using jconsole and jvisualVM to monitor remote spring boot program
key: 20181112
tags: ["jvm","spring boot"]
aside:
  toc: true
excerpt_separator: <!--more-->
excerpt_type: text # text (default), html
---



利用jconsole和jvisualVM 监控远程spring boot程序

<!--more-->

1. **linux端,配置spring boot jar启动参数**

开启远程debug

```shell
   nohup java -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=3286,suspend=n \
   -jar gs-rest-service-0.1.0.jar > gs-rest-service.log 2>&1 &
```


开启远程监控
   

```shell
   nohup java -Djava.rmi.server.hostname=192.168.61.100 \
   -Dcom.sun.management.jmxremote \
   -Dcom.sun.management.jmxremote.port=9999 \
   -Dcom.sun.management.jmxremote.authenticate=false \
   -Dcom.sun.management.jmxremote.ssl=false \
   -Dcom.sun.management.jmxremote.rmi.port=12349 \
   -jar gs-rest-service-0.1.0.jar > gs-rest-service.log 2>&1 &	
```

2. **windows端，打开JVM监控客户端**

  打开jconsole客户端

  ```powershell
  C:\Users\59690>jconsole 192.168.61.100:9999
  ```

  打开jvisualvm客户端

  ```powershell
  C:\Users\59690>jvisualvm --console new --openjmx 192.168.61.100:9999
  ```


