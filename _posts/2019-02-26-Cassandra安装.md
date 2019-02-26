---
layout:     post
title:      Cassandra安装
subtitle:   
date:       2019-02-26
author:     gary
header-img: 
catalog: true
tags:
    - Cassandra
---

# 安装
```
echo "deb http://debian.datastax.com/community stable main" | sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
curl https://www.apache.org/dist/cassandra/KEYS | sudo apt-key add -
# 如果报错哪个key(例如350200F2B999A372)找不到,则用下面命令修复
# apt-key adv --keyserver pool.sks-keyservers.net --recv-key 350200F2B999A372
apt-get update
apt-get install dsc22=2.2.8-1 cassandra=2.2.8 cassandra-tools=2.2.8

# 允许外部访问
vi /etc/cassandra/cassandra.yaml
# 找到rpc_address: localhost,改为实际IP:
rpc_address: 实际IP

service cassandra restart

export CQLSH_NO_BUNDLED=true
export CQLSH_HOST=之前设置的IP
virtualenv --no-site-packages --python=python2.7 cassandra-venv
source cassandra-venv/bin/activate
pip install cqlsh
```

# 验证
nodetool status或service cassandra status

# 连接
cqlsh,可以看到[cqlsh 5.0.1 | Cassandra 2.2.8 | CQL spec 3.3.1 | Native protocol v4]

# 常用

## 启动
```
service cassandra start
```
However, normally the service will start automatically. For this reason be sure to stop it if you need to make any configuration changes.

## 关闭
```
service cassandra stop
```

## 文件位置
- configuration: /etc/cassandra.
- log and data: /var/log/cassandra/ and /var/lib/cassandra
- Start-up options (heap size, etc): /etc/default/cassandra
