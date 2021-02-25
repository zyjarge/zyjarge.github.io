---
title: "【大数据】元数据管理工具Apache Altas与Hive的整合"
date: 2020-01-25T22:50:05+08:00
categories: ['大数据','数据治理']
draft: false
---

# 1. 前情提要：

　　前面写了一篇文章讲了关于如何搭建Atlas服务端环境，但是想真正进行数据治理还需要搭建客户端环境，即需要在希望采集元数据的前端环境中，部署Atlas的客户端，将元数据上报到服务端，从而在服务端进行统一管理，本篇文章将介绍如何将Atlas与Hive进行整合，之所以选择Hive作为首个进行元数据的管理对象，主要考虑原因有：

- Hive本身是数据平台的一个核心组件，几乎所有大数据平台都会用到Hive，选择他比较具有代表性。
- Hive基本是数据平台中各个角色（研发、产品、测试、运营等）都会接触到的一个组件，也是大家平时对元数据提出问题比较多的一个组件。

# 2. 原理介绍

　　这里简单介绍一下Atlas是如何进行元数据采集的，首先看下Atlas官网的系统架构图。

![architecture](/img/2020/atlas_integrate_hive/architecture.png)

　　我们目前只需看这张概览图中的左上部分，这部分就是atlas支持的元数据的数据源，（目前可以支持如下这些数据源：Hive、Sqoop、Falcon、Storm、Hbase），这些数据源是作为消息的生产者将数据打到Kafka中，atlas的服务端在Kafka队列的另一端作为消费者，将这些元数据统一录入到系统中。

　　那具体对于Hive来说，是通过Hive的Hook机制进行采集的，说白了就是在hive的post execution阶段，系统都会根据配置执行对应的钩子程序，这个原理有点类似Spring中的AOP机制。



# 3. 部署Hive Hook

　　在了解了基本原理之后，我们开始实际部署工作。

1. 在`${ATLAS_SOURCE}/distro/target/`目录下找到hive-hook的压缩包`apache-atlas-${ver}-hive-hook.tar.gz`，将这个压缩包复制到要采集元数据的Hive节点上，解压到任意路径，我这里定义该路径为HIVE_HOOK_HOME。

2. 修改hive/conf下的hive-site.xml文件：

   ```xml
   <property>
       <name>hive.exec.post.hooks</name>
       <value>org.apache.atlas.hive.hook.HiveHook</value>
   </property>
   ```

3. 将` apache-atlas-hive-hook-${project.version}/hook/hive`目录下的所有内容复制到``<atlas package>`/hook/hive`目录下，PS：我在执行这步操作的时候，发现atlas对应目录下已经存在所有的文件了，就跳过了，后期经过验证没有产生问题。

4. 修改hive/conf下的`hive-env.sh`，添加如下内容：

   ```bash
   export HIVE_AUX_JARS_PATH=${HIVE_HOOK_HOME}/hook/hive
   ```

5. 复制${ATLAS_HOME}/conf/atlas-application.properties 到hive的conf路径下，并修改其中的与kafka有关的几个参数：

   ```bash
   # kafka对应的zk地址，我们这里使用的是atlas自带的zk服务
   atlas.kafka.zookeeper.connect=atlas:2181
   # kafka的bootstrap server的地址，也是同样使用atlas自带的kafka服务
   atlas.kafka.bootstrap.servers=atlas:9092
   ```

   除了上面的两个以外还有其他的一些，例如链接kafka超时时间，每次同步zk的时间间隔等，可根据自己的实际业务需要进行调整。

6. 至此，我们的HIve-hook就已经配置完成了。正常的话，我们通过hive命令创建一个测试的数据库或表格，可以在atlas的服务端中查询到。

   执行如下测试SQL：

   ```sql
   create table test14 as (select id,name from test13 union all select id,name from test12);
   ```

   则会显示如下的血缘信息：

   ![image-20210226022646061](/img/2020/atlas_integrate_hive/test_import.png)

7. atlas提供了一个工具脚本在hook-bin/import-hive.sh，可以通过执行该脚本将之前创建的库表信息导入到atlas中。



# 4. 部署小插曲

　　跟之前安装atlas的服务端一样，在整个部署过程中也出现一些”小插曲“，我把排查问题的方法记录下。

## 4.1 执行import-hive.sh报错

　　在执行import-hive.sh的时候报错，错误信息为：

```java
org.apache.atlas.AtlasException: Failed to load application properties
at org.apache.atlas.ApplicationProperties.get(ApplicationProperties.java:147)
at org.apache.atlas.ApplicationProperties.get(ApplicationProperties.java:100)
at org.apache.atlas.hive.bridge.HiveMetaStoreBridge.main(HiveMetaStoreBridge.java:123)
Caused by: org.apache.commons.configuration.ConversionException: 'atlas.graph.index.search.solr.wait-searcher' doesn't map to a List object: true, a java.lang.Boolean
```

　　这个问题主要是由于我所使用的hive的commons-configuration包与hive-hook的所使用的有版本冲突导致，我采取的解决办法在import-hive.sh脚本中调整一下多个环境CP的加载顺序，将atlas调整在最前，这样根据jvm的类加载的最先机制，就可以优先使用hive-hook中的版本，同时还不会影响hive自己的版本。

```bash
# 将import-hive.sh中的ATLASCPPATH调整在最前
CP="${ATLASCPPATH}:${HIVE_CP}:${HADOOP_CP}"
```



## 4.2 在hive中创建table，在服务端无法查询到

　　出现这个问题，脑中第一个想法就是看下元数据信息是否已经发送到kafka中，经查验消息并没有插入到消息队列中，也就是说问题出在客户端这边。

　　再继续进行排查，但是没有运行日志，是无法知道内部运行情况的，所以下一步我在hive的log4j2的配置文件中，键入的hive-hook的输出日志配置。将org.apache.atlas下的日志信息，输出到一个临时的日志中，方便进行排查。

　　具体log4j的配置如下：

```properties
appender.AtlasFILE.type = RollingFile
appender.AtlasFILE.name = AtlasFILE
appender.AtlasFILE.fileName = /tmp/atlas_hook.log
appender.AtlasFILE.filePattern = /tmp/atlas-%d{MM-dd-yy-HH-mm-ss}-%i.log.gz
appender.AtlasFILE.layout.type = PatternLayout
appender.AtlasFILE.layout.pattern = %d %p %C{1.}:%L %m%n
appender.AtlasFILE.policies.type = Policies
appender.AtlasFILE.policies.time.type = TimeBasedTriggeringPolicy
appender.AtlasFILE.policies.time.interval = 200000
appender.AtlasFILE.policies.time.modulate = true
appender.AtlasFILE.policies.size.type = SizeBasedTriggeringPolicy
appender.AtlasFILE.policies.size.size=100MB
appender.AtlasFILE.strategy.type = DefaultRolloverStrategy
appender.AtlasFILE.strategy.max = 5

logger.rolling.name = org.apache.atlas
logger.rolling.level = debug
logger.rolling.additivity = true
logger.rolling.appenderRef.rolling.ref = AtlasFILE
```

　　把logger的级别调到debug后，根据打印出的运行日志发现，客户端已经拦截到了我们执行的SQL，并且已经正确产生了要发送到kafka的消息报文，但是诡异的是日志显示发送正常没有报任何异常，后来无意中发现是由于atlas-application.properties中的kafka服务的地址填写错误，错写成了生产环境中的kafka，但实际我们使用的是atlas自带的kafka服务，在改过之后就可以正常发送消息了。

# 5.总结

　　在经历了几番挫折之后，总算是把hive与atlas整合好了，说下自己的几点感受：

1. atlas的设计还是很好的，在多个不同的数据源之前，统一采用kafka作为中间消息传递队列，使元数据源与服务端采用异步方式进行沟通，减少元数据采集对正常业务效率的影响。
2. 在遇到问题之后找不到好的解决方案时，最好能打开执行日志并调整到DEBUG级别，往往根据日志信息可以排除出问题所在。