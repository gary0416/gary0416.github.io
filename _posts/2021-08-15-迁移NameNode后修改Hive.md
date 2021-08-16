---
layout:     post
title:      迁移NameNode后修改Hive
subtitle:   
date:       2021-08-15
author:     gary
header-img: 
catalog: true
tags:
    - Hive
---

# 背景
CDH(6.3.2)集群的NameNode部署位置迁移、namespace不变。
HDFS服务迁移(通过UI迁移角色)并恢复后，Hive查询carbondata等表可能失败，报错：
```
Error: java.net.ConnectException: Call From 主机名/IP to 旧NN的主机名/旧NN的IP:8020 failed on connection exception: java.net.Connection: Connection refused; For more details ses: http://wiki.apache.org/hadoop/ConnectionRefused
```

# 解决
1. Cloudera Manager上，Hive，点击 操作-> 更新 Hive Metastore NameNode
2. 手动修改meatstore(例如MySQL数据库)的SERDE_PARAMS表：
```
update `metastore`.`SERDE_PARAMS` set PARAM_VALUE=replace(PARAM_VALUE,'旧NN主机名','新NN主机名') WHERE PARAM_VALUE like '%旧NN主机名:8020%'
```
如果HDFS的namespace也变更了，还需要修改DBS和SDS表。