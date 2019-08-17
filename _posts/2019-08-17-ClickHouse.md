---
layout:     post
title:      ClickHouse
subtitle:   
date:       2019-08-17
author:     gary
header-img: 
catalog: true
tags:
    - ClickHouse
---

# 启动clickhouse-server(docker)
## 获取配置文件
```bash
docker run --name tmp-clickhouse-server -d yandex/clickhouse-server
docker cp tmp-clickhouse-server:/etc/clickhouse-server/ .
docker rm -f tmp-clickhouse-server
```

## start-server.sh
```bash
#!/bin/bash -l
set -o nounset
set -o errexit
# https://hub.docker.com/r/yandex/clickhouse-server/
# Container exposes 8123 port for HTTP interface and 9000 port for native client.
docker run -d --name test-clickhouse-server --ulimit nofile=262144:262144 -v /home/zhangtb/soft/docker/clickhouse/data:/var/lib/clickhouse -v /home/zhangtb/soft/docker/clickhouse/clickhouse-server:/etc/clickhouse-server/clickhouse-server -p 8123:8123 -p 9000:9000 yandex/clickhouse-server
```

# 启动clickhouse-client(docker)
## start-client.sh
```bash
docker run -it --rm --link test-clickhouse-server:clickhouse-server yandex/clickhouse-client --host clickhouse-server
```

# 安装clickhouse-client
```bash
sudo echo "deb http://repo.yandex.ru/clickhouse/deb/stable/ main/" > /etc/apt/sources.list.d/clickhouse.list
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv E0C56BD4
sudo apt-get update && sudo apt-get install clickhouse-client
```

# 常用
## 数据库
默认在default库

## 建表
```sql
CREATE TABLE 表名
(
    列1 String,
    列2 Int32,
    列3 DateTime,
    列4 UInt8,
    列5 Nullable(String),
    列6 Date
)
ENGINE=MergeTree(列6, (列3, 列6), 8192);
```
- 也可以ENGINE=MergeTree() PARTITION BY toYYYYMMDD(dt) ORDER BY (CounterID, event_time, intHash32(UserID)) SETTINGS index_granularity=8192，详见官方文档。

# 导入
## 从csv
```bash
xz -v -c -d < data_201908.csv.xz | clickhouse-client --format_csv_delimiter="," --query="INSERT INTO 表名 FORMAT CSVWithNames"
```
- 日期格式关注date_time_input_format配置。

## 从mysql
1. csv导入mysql
```bash
#!/usr/bin/env bash
for f in *.csv
do
    mysql -e "LOAD DATA LOCAL INFILE '"$f"' INTO TABLE 表名 FIELDS TERMINATED BY ',' lines terminated by '\n' ignore 1 lines(列1,列2,...)" -u 用户名 --password=密码 数据库名
done
```

2. 从mysql导入clickhouse
```sql
insert into 表名
SELECT * FROM mysql('非本机的IP:3306', '数据库名', '表名', '用户名', '密码') 
```
mysql的IP不能写localhost或127.0.0.1

3. 导入结果
Elapsed: 414.291 sec. Processed 126.21 million rows, 24.87 GB (304.65 thousand rows/s., 60.04 MB/s.)

此时1亿两千万条记录查询count耗时0.467s。

# 附
## hive导出
```bash
beeline -u 'jdbc:hive2://zk1:2181,zk2:2181,zk3:2181/数据库名;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=hiveserver2' -n 用户名 -p 密码 --showHeader=true --verbose=true --outputformat=csv2 -e "select语句">>data_201908.csv && xz data_201908.csv
```

## hive日期格式转换
```sql
from_unixtime(unix_timestamp(日期列,'yyyyMMdd'),'yyyy-MM-dd')
```
