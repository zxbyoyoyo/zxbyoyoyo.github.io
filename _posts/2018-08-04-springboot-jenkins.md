---
title: Deploy spring boot project with jenkins
key: 20180804
tags: jenkins
aside:
  toc: true
excerpt_separator: <!--more-->
excerpt_type: text # text (default), html
---
# Jenkins 是什么？ #

Jenkins 是一个可扩展的持续集成引擎。

**主要用于：**

持续、自动地构建/测试软件项目。 
监控一些定时执行的任务。
<!--more-->
**Jenkins 拥有的特性包括：**

- 易于安装-只要把`jenkins.war`部署到`servlet容器`，不需要数据库支持。 
- 易于配置-所有配置都是通过其提供的web界面实现。 
- 集成RSS/E-mail通过RSS发布构建结果或当构建完成时通过e-mail通知。 
- 生成JUnit/TestNG测试报告。 
- 分布式构建支持Jenkins能够让多台计算机一起构建/测试。 
- 文件识别:Jenkins能够跟踪哪次构建生成哪些jar，哪次构建使用哪个版本的jar等。 
- 插件支持:支持扩展插件，你可以开发适合自己团队使用的工具。

# 准备工作 #

## 环境 ##

```
JDK:1.8  
Jenkins:2.138.3
Centos:7.3  
```

# 安装 #

## 下载 ##

```shell
[zxb@server00 jenkins]$ pwd
/home/zxb/jenkins
[zxb@server00 jenkins]$ wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war
```

## 启动 ##

关闭防护墙

```shell
[zxb@server00 jenkins]$ systemctl stop firewalld.service
```

启动服务

```shell
[zxb@server00 jenkins]$ java -jar jenkins.war --httpPort=9090
```

Jenkins 就启动成功了！它的war包自带Jetty服务器

## 访问 ##

浏览器访问：<http://localhost:9090/>

第一次启动Jenkins时，出于安全考虑，Jenkins会自动生成一个随机的按照口令。

注意控制台输出的口令，复制下来，然后在浏览器输入密码：

```

*************************************************************

Jenkins initial setup is required. An admin user has been created and a password                                                                                generated.
Please use the following password to proceed to installation:

b6d3ff8c7f3445dc91f926657fdb59f3

This may also be found at: /home/zxb/.jenkins/secrets/initialAdminPassword

*************************************************************
```

进入用户自定义插件界面，建议选择安装官方推荐插件，因为安装后自己也得安装:
![enter description here](https://i.loli.net/2018/12/02/5c036a0309bbb.jpg "1543716678020")


接下来是进入插件安装进度界面:

![enter description here](https://i.loli.net/2018/12/02/5c036a3df33bf.jpg "1543716753916")

配置用户名密码:

![1543716939923.png](https://i.loli.net/2018/12/02/5c036a1292053.jpg "1543716939923")

初始化成功后进入 Jenkins 首页:

![1543716986513.png](https://i.loli.net/2018/12/02/5c036a569794d.jpg "1543716986513")

全局工具配置.Jdk,Mavem,git Docker,等配置，安装

![enter description here](https://i.loli.net/2018/12/02/5c036a8b7c3c0.jpg "1543717079594")

![enter description here](https://i.loli.net/2018/12/02/5c036a98e0533.jpg "1543719778519")

# 构建项目 #

## 新建项目 ##

![enter description here](https://i.loli.net/2018/12/02/5c036ab14e20d.jpg "1543722316470")

这里，选择构建一个自由风格的软件项目；

![enter description here](https://i.loli.net/2018/12/02/5c036acaaa7e0.jpg "1543718482278")

## 源码管理 ##

Jenkins支持多种源码管理服务器；
![enter description here](https://i.loli.net/2018/12/02/5c036ada85fdf.jpg "1543722381433")

## 构建配置 ##

选择 `Execute shell` 构建 输入一下命令并且保存

```
mvn clean package
```

![enter description here](https://i.loli.net/2018/12/02/5c036aea65d68.jpg "1543722710401")

## 立即构建 ##

![](https://i.loli.net/2018/12/02/5c036afc7d882.jpg "1543722758872")

## 查看日志 ##

![enter description here](https://i.loli.net/2018/12/02/5c036b1ec7d3d.jpg "1543723696798")

![enter description here](https://i.loli.net/2018/12/02/5c036b3c032c6.jpg "1543723736538")

```
由用户 zxbyoyoyo 启动
构建中 在工作空间 /home/zxb/.jenkins/workspace/sayHelloProject 中
 > /usr/local/bin/git rev-parse --is-inside-work-tree # timeout=10
Fetching changes from the remote Git repository
 > /usr/local/bin/git config remote.origin.url https://github.com/zxbyoyoyo/jenkins-for-spring-boot.git # timeout=10
Fetching upstream changes from https://github.com/zxbyoyoyo/jenkins-for-spring-boot.git
 > /usr/local/bin/git --version # timeout=10
 > /usr/local/bin/git fetch --tags --progress https://github.com/zxbyoyoyo/jenkins-for-spring-boot.git +refs/heads/*:refs/remotes/origin/*
 > /usr/local/bin/git rev-parse refs/remotes/origin/master^{commit} # timeout=10
 > /usr/local/bin/git rev-parse refs/remotes/origin/origin/master^{commit} # timeout=10
Checking out Revision bd0b826682893c07377ca514cb0c2265648b1709 (refs/remotes/origin/master)
 > /usr/local/bin/git config core.sparsecheckout # timeout=10
 > /usr/local/bin/git checkout -f bd0b826682893c07377ca514cb0c2265648b1709
Commit message: "test jenkins"
 > /usr/local/bin/git rev-list --no-walk bd0b826682893c07377ca514cb0c2265648b1709 # timeout=10
[sayHelloProject] $ /bin/bash -l /tmp/jenkins5475699506452979742.sh
[INFO] Scanning for projects...
[INFO] 
[INFO] ----------------< org.springframework:gs-rest-service >-----------------
[INFO] Building gs-rest-service 0.1.0
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 2, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] 
[INFO] --- maven-jar-plugin:3.0.2:jar (default-jar) @ gs-rest-service ---
[INFO] Building jar: /home/zxb/.jenkins/workspace/sayHelloProject/target/gs-rest-service-0.1.0.jar
[INFO] 
[INFO] --- spring-boot-maven-plugin:2.0.5.RELEASE:repackage (default) @ gs-rest-service ---
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  45.432 s
[INFO] Finished at: 2018-12-02T12:08:26+08:00
[INFO] ------------------------------------------------------------------------
Finished: SUCCESS
```

## 查看 jar ##

![enter description here](https://i.loli.net/2018/12/02/5c036b5d84947.jpg "1543723867980")

# 部署项目 #

## 部署脚本 ##

**把脚本放到 /etc/rc.d/init.d 下赋权限 chmod 777 spring-boot.sh**

```
#!/bin/bash

SpringBoot=$2

#启动参数
START_OPTS=$3

#JVM参数
JVM_OPTS="-Dname=$SpringBoot  -Duser.timezone=Asia/Shanghai -Xms512M -Xmx512M -XX:PermSize=256M -XX:MaxPermSize=512M -XX:+HeapDumpOnOutOfMemoryError -XX:+PrintGCDateStamps  -XX:+PrintGCDetails -XX:NewRatio=1 -XX:SurvivorRatio=30 -XX:+UseParallelGC -XX:+UseParallelOldGC"
APP_HOME=`pwd`
LOG_PATH=$APP_HOME/logs/$SpringBoot.log

if [ "$1" = "" ];
then
    echo -e "\033[0;31m 未输入操作名 \033[0m  \033[0;34m {start|stop|restart|status} \033[0m"
    exit 1
fi

if [ "$SpringBoot" = "" ];
then
    echo -e "\033[0;31m 未输入应用名 \033[0m"
    exit 1
fi

function start()
{
    count=`ps -ef |grep java|grep $SpringBoot|grep -v grep|wc -l`
    if [ $count != 0 ];then
        echo "$SpringBoot is running..."
    else
        echo "Start $SpringBoot success..."
        BUILD_ID=dontKillMe nohup java -jar  $JVM_OPTS $SpringBoot  $START_OPTS > /dev/null 2>&1 &
    fi
}

function stop()
{
    echo "Stop $SpringBoot"
    boot_id=`ps -ef |grep java|grep $SpringBoot|grep -v grep|awk '{print $2}'`
    count=`ps -ef |grep java|grep $SpringBoot|grep -v grep|wc -l`

    if [ $count != 0 ];then
        kill $boot_id
        count=`ps -ef |grep java|grep $SpringBoot|grep -v grep|wc -l`

        boot_id=`ps -ef |grep java|grep $SpringBoot|grep -v grep|awk '{print $2}'`
        kill -9 $boot_id
    fi
}

function restart()
{
    stop
    sleep 2
    start
}

function status()
{
    count=`ps -ef |grep java|grep $SpringBoot|grep -v grep|wc -l`
    if [ $count != 0 ];then
        echo "$SpringBoot is running..."
    else
        echo "$SpringBoot is not running..."
    fi
}

case $1 in
    start)
    start;;
    stop)
    stop;;
    restart)
    restart;;
    status)
    status;;
    *)

    echo -e "\033[0;31m Usage: \033[0m  \033[0;34m sh  $0  {start|stop|restart|status}  {SpringBootJarName} \033[0m
\033[0;31m Example: \033[0m
      \033[0;33m sh  $0  start esmart-test.jar \033[0m"
esac
```

## 部署语法 ##

![enter description here](https://i.loli.net/2018/12/02/5c036ef8ec66b.jpg "1543724160298")
```
#!/bin/bash -l

mvn clean package

cp /etc/rc.d/init.d/spring-boot.sh /home/zxb/.jenkins/workspace/sayHelloProject/target

cd /home/zxb/.jenkins/workspace/sayHelloProject/target

./spring-boot.sh restart gs-rest-service-0.1.0.jar
```

**请注意配置构建脚本的时候的写法,没有BUILD_ID=dontKillMe 是不可以的**

```shell
BUILD_ID=dontKillMe nohup java -jar $SpringBoot > /dev/null 2>&1 &
```

## 立即构建 ##

```
由用户 zxbyoyoyo 启动
构建中 在工作空间 /home/zxb/.jenkins/workspace/sayHelloProject 中
 > /usr/local/bin/git rev-parse --is-inside-work-tree # timeout=10
Fetching changes from the remote Git repository
 > /usr/local/bin/git config remote.origin.url https://github.com/zxbyoyoyo/jenkins-for-spring-boot.git # timeout=10
Fetching upstream changes from https://github.com/zxbyoyoyo/jenkins-for-spring-boot.git
 > /usr/local/bin/git --version # timeout=10
 > /usr/local/bin/git fetch --tags --progress https://github.com/zxbyoyoyo/jenkins-for-spring-boot.git +refs/heads/*:refs/remotes/origin/*
 > /usr/local/bin/git rev-parse refs/remotes/origin/master^{commit} # timeout=10
 > /usr/local/bin/git rev-parse refs/remotes/origin/origin/master^{commit} # timeout=10
Checking out Revision bd0b826682893c07377ca514cb0c2265648b1709 (refs/remotes/origin/master)
 > /usr/local/bin/git config core.sparsecheckout # timeout=10
 > /usr/local/bin/git checkout -f bd0b826682893c07377ca514cb0c2265648b1709
Commit message: "test jenkins"
 > /usr/local/bin/git rev-list --no-walk bd0b826682893c07377ca514cb0c2265648b1709 # timeout=10
[sayHelloProject] $ /bin/bash -l /tmp/jenkins3415421897285261259.sh
[INFO] Scanning for projects...
[INFO] 
[INFO] ----------------< org.springframework:gs-rest-service >-----------------
[INFO] Building gs-rest-service 0.1.0
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:3.0.0:clean (default-clean) @ gs-rest-service ---
[INFO] Deleting /home/zxb/.jenkins/workspace/sayHelloProject/target
[INFO] 
[INFO] --- maven-resources-plugin:3.0.2:resources (default-resources) @ gs-rest-service ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /home/zxb/.jenkins/workspace/sayHelloProject/src/main/resources
[INFO] skip non existing resourceDirectory /home/zxb/.jenkins/workspace/sayHelloProject/src/main/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.7.0:compile (default-compile) @ gs-rest-service ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 3 source files to /home/zxb/.jenkins/workspace/sayHelloProject/target/classes
[INFO] 
[INFO] --- maven-resources-plugin:3.0.2:testResources (default-testResources) @ gs-rest-service ---
[INFO] Not copying test resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.7.0:testCompile (default-testCompile) @ gs-rest-service ---
[INFO] Not compiling test sources
[INFO] 
[INFO] --- maven-surefire-plugin:2.21.0:test (default-test) @ gs-rest-service ---
[INFO] Tests are skipped.
[INFO] 
[INFO] --- maven-jar-plugin:3.0.2:jar (default-jar) @ gs-rest-service ---
[INFO] Building jar: /home/zxb/.jenkins/workspace/sayHelloProject/target/gs-rest-service-0.1.0.jar
[INFO] 
[INFO] --- spring-boot-maven-plugin:2.0.5.RELEASE:repackage (default) @ gs-rest-service ---
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  14.742 s
[INFO] Finished at: 2018-12-02T13:03:41+08:00
[INFO] ------------------------------------------------------------------------
Stop gs-rest-service-0.1.0.jar
Start gs-rest-service-0.1.0.jar success...
Finished: SUCCESS
```

## 查看进程 ##

```
[root@server00 target]# ps -ef|grep java
zxb        8043   2563  1 11:08 pts/0    00:02:11 java -jar jenkins.war --httpPort=9090
zxb       16996      1 28 13:01 pts/0    00:00:13 java -jar -Dname=gs-rest-service-0.1.0.jar -Duser.timezone=Asia/Shanghai -Xms512M -Xmx512M -XX:PermSize=256M -XX:MaxPermSize=512M -XX:+HeapDumpOnOutOfMemoryError -XX:+PrintGCDateStamps -XX:+PrintGCDetails -XX:NewRatio=1 -XX:SurvivorRatio=30 -XX:+UseParallelGC -XX:+UseParallelOldGC gs-rest-service-0.1.0.jar
root      17077   6289  0 13:01 pts/2    00:00:00 grep --color=auto java
```

## 访问项目 ##

![1543727093197.png](https://i.loli.net/2018/12/02/5c036b6ec7561.jpg "1543727093197")



jenkins执行shell读不到环境变量问题 https://blog.csdn.net/zzusimon/article/details/57080337