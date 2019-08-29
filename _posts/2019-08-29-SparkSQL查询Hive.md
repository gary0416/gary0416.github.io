---
layout:     post
title:      SparkSQL查询Hive
subtitle:   
date:       2019-08-29
author:     gary
header-img: 
catalog: true
tags:
    - Spark
---

# 1. 命令行方式
hive-site.xml放到spark的conf下即可，注意此处非全部配置，配置key如下：

- hive.metastore.uris (主要)
- hive.server2.thrift.port
- hive.server2.enable.doAs
- hive.metastore.client.socket.timeout
- hive.metastore.client.connect.retry.delay

下面的程序方式用到的也是这个精简版的配置文件。

此时使用spark-sql --master yarn已经可以查询Hive。

# 2. Java程序方式
## 2.1. 方法一：配置文件随jar包发布
适用于开发环境。

hive-site.xml（精简版）放resources下即可。

测试集群的配置文件可留在项目里，maven shade打包排除配置xml。这样代码库中同一份代码，可同时适用于本地调试和正式环境运行。

## 2.2. 方法二：配置文件在jar包外
适用于正式环境。

主要代码如下，以HDP路径为例：

```Java
SparkConf sparkConf = new SparkConf();
...
String hiveSitePath = "/etc/spark2/conf/hive-site.xml";
if (new File(hiveSitePath).exists()) {
    Configuration cfg = new Configuration(false);
    cfg.addResource(new Path(hiveSitePath));
    cfg.forEach(consumer -> {
        sparkConf.set(consumer.getKey(), consumer.getValue());
    });
}

SparkSession sparkSession = SparkSession.builder().config(sparkConf).enableHiveSupport().getOrCreate();
sparkSession.sql("show databases").show();
```

生产环境可将不同集群的配置文件，挂载到容器里，供程序加载并运行。

如何加载hive-site.xml：
https://github.com/hortonworks/spark2-release/blob/HDP-2.6.3.0-235-tag/sql/core/src/main/scala/org/apache/spark/sql/internal/SharedState.scala#L47

如果是Scala，不是用Java，也可参考源码的写法直接加载文件。
