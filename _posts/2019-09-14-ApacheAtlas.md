---
layout:     post
title:      Apache Atlas
subtitle:   
date:       2019-09-14
author:     gary
header-img: 
catalog: true
tags:
    - Atlas
---

# Apache Atlas
## HBase新建命名空间
```
su - hbase
hbase shell

create_namespace 'atlas'
grant 'atlas', 'RWXCA','@atlas'
```

## 安装Atlas
ambari上直接添加服务。
配置修改：
```
atlas.audit.hbase.tablename        atlas:ATLAS_ENTITY_AUDIT_EVENTS
atlas.graph.storage.hbase.table    atlas:atlas_titan
atlas.server.bind.address          0.0.0.0
```
日志修改
```
<appender name="AUDIT" class="org.apache.log4j.RollingFileAppender">
    <param name="File" value="{{log_dir}}/audit.log"/>
    <param name="Append" value="true"/>
    <param name="Threshold" value="info"/>
    <param name="MaxFileSize" value="128MB" />
    <param name="MaxBackupIndex" value="10" />
    <layout class="org.apache.log4j.PatternLayout">
      <param name="ConversionPattern" value="%d %x %m%n"/>
    </layout>
</appender>
```

metadata_classpath可设置三方jar。避免java.lang.NoClassDefFoundError: org/apache/hadoop/hbase/util/Bytes等错误。因为Hive表的STORED BY可指定handler，实现查询HBase、JDBC、ES等。

复制elasticsearch-hadoop-hive-6.5.4.jar，hive-jdbc-handler-2.3.5.jar到每个Atlas节点的/usr/local/common-jar。
最终metadata_classpath如下(注意前后不能有空格，冒号分割):
```
/usr/local/common-jar/elasticsearch-hadoop-hive-6.5.4.jar:/usr/local/common-jar/hive-jdbc-handler-2.3.5.jar:/usr/hdp/current/hbase-client/lib/hbase-common-1.1.2.2.6.3.0-235.jar
```

## 用户
- 文件验证，路径在/etc/atlas/conf/users-credentials.properties，密码为SHA256
- LDAP需配置角色

## 访问
http://节点IP:21000/
默认用户admin/admin

## 导入hive
. /usr/hdp/current/atlas-server/conf/atlas-env.sh && bash -x /usr/hdp/current/atlas-server/hook-bin/import-hive.sh
然后输入hive配置的superuser用户(hive权限大的).导入日志在/var/log/atlas/import-hive.log

## Reference
- [Hadoop的元数据治理--Apache Atlas](https://www.jianshu.com/p/4eee91bc926c)
- [使用 Apache Atlas 进行数据治理](https://zhuanlan.zhihu.com/p/26843722)
