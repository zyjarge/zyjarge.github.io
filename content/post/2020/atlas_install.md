---
title: "元数据管理工具 Apache Atlas 安装"
date: 2020-10-25T11:19:59+08:00
draft: false
Categories: ['大数据']
---

![atlas_logo](/img/2020/atlas_install/atlas_logo.svg)

# Atlas 安装：

## 1. atlas安装主要信息如下：

- 版本：atlas-2.1.0 ： https://www.apache.org/dyn/closer.cgi/atlas/2.1.0/apache-atlas-2.1.0-sources.tar.gz
- 模式：采用atlas自带的habse和solr作为存储和搜索引擎
- 其他依赖： jdk 1.8+kafka

## 2. 安装步骤

### 2.1 下载安装包、解压、设置PATH：

```bash
wget https://www.apache.org/dyn/closer.cgi/atlas/2.1.0/apache-atlas-2.1.0-sources.tar.gz
tar xvf apache-atlas-2.1.0-sources.tar.gz -C /usr/lib/
vim /etc/profile
# 添加atlas的路径到PATH
PATH=$ATLAS_HOME/bin:$PATH
```

### 2.2 构建打包：

由于采用了内置的hbase和solr，因此采用如下命令进行打包：

```bash
mvn clean -DskipTests package -Pdist,embedded-hbase-solr
```

由于在构建过程中需要下载hbase和solr两个大家伙，使用命令行下载比较慢，这里有个小技巧，可以提前使用下载工具下载好，放入如下目录，然后再进行构建的时候，会判断安装包已经存在跳过下载直接解压安装。

```bash
hbase-2.0.2.tar.gz：放入 apache-atlas-sources-2.1.0/distro/hbase/
solr-7.5.0.tgz: 放入 apache-atlas-sources-2.1.0/distro/solr/
```

在构建成功之后，会在distro目录下产生如下格式的安装包：

```bash
distro/target/apache-atlas-{project.version}-bin.tar.gz
distro/target/apache-atlas-{project.version}-hbase-hook.tar.gz
distro/target/apache-atlas-{project.version}-hive-hook.gz
distro/target/apache-atlas-{project.version}-kafka-hook.gz
distro/target/apache-atlas-{project.version}-sources.tar.gz
distro/target/apache-atlas-{project.version}-sqoop-hook.tar.gz
distro/target/apache-atlas-{project.version}-storm-hook.tar.gz
```

### 2.3 部署服务端

将`distro/target/apache-atlas-{project.version}-bin.tar.gz`解压到部署路径并建立简写的软连接，这里以/opt/atlas为例，即：

```bash
tar xvf `distro/target/apache-atlas-2.1.0-bin.tar.gz`-C /opt
ln -s apache-atlas-2.1.0-bin atlas
```

修改conf/atlas-application.properties中的部分参数：

```properties
# 指定数据存储引擎，这里使用hbase
atlas.graph.storage.backend=hbase2 
# 使用内置hbase，因此地址指向localhost
atlas.graph.storage.hostname=localhost
# 搜索+索引使用solr
atlas.graph.index.search.backend=solr
# 使用内置的zookeeper
atlas.graph.index.search.solr.zookeeper-url=localhost:2181
```



### 2.4 启动atlas服务

使用内置的hbase和solr需要配置如下环境变量：

```bash
export MANAGE_LOCAL_HBASE=true
export MANAGE_LOCAL_SOLR=true
```

启动服务：

```bash
bin/atlas_start.py
```

### 2.5 解决问题

　　安装官网的文档介绍，到此应该就可以正常启动atlas了，但是我这里遇到了一些小插曲。

#### 插曲1：启动后无法访问管理页面

　　症状就是日志提示atlas已经启动，但是管理页面无法打开，除此之外没有任何提示。采用最简单粗暴的方法，从入口的脚本一步一步的排查，主要做了如下几个事：

1. 首先，了解脚本atlas_start.py的大概逻辑，其实大概逻辑很简单，就是根据配置信息设置系统变量、以及启动内置的hbase和solr服务等。

2. 打断点的方式一步一步排查，发现是内置的solr和hbase服务没有正常启动。

3. 在查看代码过程中，发现该脚本中引用另一个脚本里atlas_config.py有个DEBUG变量，改为`DEBUG = True`可以打印更多的日志信息。

4. 从debug的日志中，我们可以找到启动solr的具体命令，将下面的命令行直接粘贴在shell中执行，会出现如下提示：

   ```bash
   /opt/apache-atlas-2.1.0/solr/bin/solr start -z localhost:2181 -p 9838
   ```

   > WARNING: Starting Solr as the root user is a security risk and not considered best practice. Exiting.
   >          Please consult the Reference Guide. To override this check, start with argument '-force'

5. 到这里基本找到问题了，由于我是使用root账号启动，而使用root账号启动solr需要添加`-force`参数，而且整个atlas的启动脚本是使用python进行封装的，因此对于上面的信息是无法直接看到的。

6. 找到问题就好说了，在atlas_config.py中找到启动solr的`run_solr`方法，代码中把启动solr的命令存在一个cmd的数组中，我们只需要再最后追加一个`-force`参数即可。

   ```bash
   cmd.append('-force')
   ```

7. 除了启动solr以外，在启动atlas过程中还会自动创建索引，也需要采用上面的方法来解决，具体方法就是在atlas_config.py中找到`create_solr_collection`方法，在cmd后面追加`-force` 参数。

#### 插曲2：无法启动内置的Hbase

　　除了无法启动solr服务外，我也遇到了无法启动内置的Hbase的问题，症状就是启动之后找不到HMaster进程。好在有了上面的经验之后，解决起来就轻车熟路了，从Debug日志信息中找到启动HBase的命令，粘贴到命令行中手动执行。执行后发现是由于没有设置JAVA_HOME导致HBase无法启动， 设置了JAVA_HOME后就可以正常启动了。

## 3.总结：

　　经过上述2个问题，不难发现都是因为atlas的启动脚本是通过python封装的shell导致无法打印出依赖组件的错误信息所导致的，好在作者在代码中加入的DEBUG模式，打开之后可以查看更为详细的日志信息。

　　最后，我们在使用一些开源框架工具的时候遇到问题的时候，在网上找不到解决方案的情况下，可以尝试从源码中下手，往往一般都能找到解决方案。