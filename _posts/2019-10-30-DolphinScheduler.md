---
layout:     post
title:      DolphinScheduler
subtitle:   
date:       2019-10-30
author:     gary
header-img: 
catalog: true
tags:
    - DolphinScheduler
---


# 版本
本文基于1.1.0的tag: https://github.com/apache/incubator-dolphinscheduler/releases/tag/1.1.0

大数据环境基于Ambari 2.6.3.0-235。

# 运行
## 单docker快速体验
```
docker run --name dolphinscheduler -d -p 8888:8888 gary0416/dolphinscheduler:1.1.0
```
docker包含了zk、mysql及DS的组件。

## 多docker支持多节点高可用
依赖：本机需要在ambari上安装过client，需要其中的spark2-client。

docker构建基于：https://github.com/gary0416/incubator-dolphinscheduler/tree/test-docker

目前限制：同一台服务器上不支持起两个worker，因为grpc端口冲突，代码里写死的，不可配置。
其他组件理论上支持同节点起多个相同组件，未详细验证，但还是建议HA时分布在不同节点上，避免单点故障。

此版本docker只是满足使用，没有最优（例如:1.基础镜像是否可以不用ubuntu18。2.通过环境变量前缀约定，实现修改对应配置文件，类似https://github.com/gary0416/docker-kafka/blob/master/start-kafka.sh#L102。3.一些其他细节），所以未提交PR，待DS版本继续完善再跟进。

worker-extra-init.sh可以做额外初始化工作，例如：
```
#!/bin/bash
# support spark,mount hdp spark-client dir with ro mode
# to prevent changing JAVA_HOME var in spark-env.sh,manually setting env.
# load-spark-env.sh need SPARK_ENV_LOADED
# .escheduler_env.sh content:
#export HADOOP_HOME=${HADOOP_HOME:-/usr/hdp/2.6.3.0-235/hadoop}
#export HADOOP_CONF_DIR=/opt/escheduler/conf
#export SPARK_HOME1=/opt/soft/spark1
#export SPARK_HOME2=/opt/spark2-client
#export PYTHON_HOME=/opt/soft/python
#export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
#export HIVE_HOME=/opt/soft/hive
#export FLINK_HOME=/opt/soft/flink
#export PATH=$HADOOP_HOME/bin:$SPARK_HOME1/bin:$SPARK_HOME2/bin:$PYTHON_HOME:$JAVA_HOME/bin:$HIVE_HOME/bin:$PATH:$FLINK_HOME/bin:$PATH
## new:
#export HDP_VERSION=2.6.3.0-235
#export SPARK_CONF_DIR=${SPARK_HOME}/conf
#export SPARK_ENV_LOADED=1
#export SPARK_LOG_DIR=/var/log/spark2
#export SPARK_PID_DIR=/var/run/spark2
#export SPARK_DAEMON_MEMORY=1024m
#export SPARK_IDENT_STRING=$USER
#export SPARK_NICENESS=0
ln -fs /opt/spark2-client/bin/spark-submit /usr/bin/spark-submit && mkdir -p /usr/hdp/2.6.3.0-235/hadoop/lib

# 自动创建测试租户用户
useradd gary && mkdir -p /opt/test/gary && chown gary /opt/test/gary
```

使用时需替换下面环境变量中的IP，指向实际的节点IP。Zookeeper和MySQL可根据实际情况修改。
```
version: '3'

networks:
  dolphinscheduler:
    driver: bridge

services:
  mysql:
    container_name: dolphinscheduler-mysql
    image: mysql:5.7.25
    restart: always
    networks:
      - dolphinscheduler
    environment:
      MYSQL_DATABASE: escheduler
      MYSQL_ROOT_PASSWORD: root@123
    command: [
      '--bind-address=0.0.0.0',
      '--character-set-server=utf8mb4',
      '--collation-server=utf8mb4_unicode_ci',
      '--default-time-zone=+8:00'
    ]
    ports:
      - 3306:3306
    volumes:
      - ./data/mysql:/var/lib/mysql
  zoo1:
    container_name: dolphinscheduler-zoo1
    image: zookeeper:3.4.14
    restart: always
    hostname: zoo1
    networks:
      - dolphinscheduler
    ports:
      - 2181:2181
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=0.0.0.0:2888:3888
    volumes:
      - ./data/zoo1/data:/data
      - ./data/zoo1/datalog:/datalog
  dolphinscheduler-ui:
    container_name: dolphinscheduler-ui
    image: gary0416/dolphinscheduler-ui:test-20191011-1831
    restart: always
    networks:
      - dolphinscheduler
    environment:
      DS_PROXY: 192.168.xx.xx:12345
    ports:
      - 8888:8888
  dolphinscheduler-api:
    container_name: dolphinscheduler-api
    image: gary0416/dolphinscheduler-api:test-20191011-1831
    restart: always
    networks:
      - dolphinscheduler
    depends_on:
      - mysql
      - zoo1
    environment:
      ZK_HOST: 192.168.xx.xx
      ZK_PORT: 2181
      MYSQL_HOST: 192.168.xx.xx
      MYSQL_PORT: 3306
      MYSQL_ROOT_PWD: root@123
      ESZ_DB: escheduler
    command: api
    ports:
      - 12345:12345
    volumes:
      - ./data/logs:/opt/escheduler/logs
      - ../../conf/escheduler/conf:/opt/escheduler/conf:ro
  dolphinscheduler-master:
    container_name: dolphinscheduler-master
    image: gary0416/dolphinscheduler-common:test-20191011-1831
    restart: always
    networks:
      - dolphinscheduler
    depends_on:
      - dolphinscheduler-api
    environment:
      ZK_HOST: 192.168.xx.xx
      ZK_PORT: 2181
    command: master
    ports:
      - 5566:5566
    volumes:
      - ./data/logs:/opt/escheduler/logs
      - ../../conf/escheduler/conf:/opt/escheduler/conf:ro
  dolphinscheduler-worker:
    container_name: dolphinscheduler-worker
    image: gary0416/dolphinscheduler-common:test-20191011-1831
    restart: always
    network_mode: host
    depends_on:
      - dolphinscheduler-api
    environment:
      ZK_HOST: 192.168.xx.xx
      ZK_PORT: 2181
    command: worker
    ports:
      - 7788:7788
      - 50051:50051
    volumes:
      - ./data/logs:/opt/escheduler/logs
      - ../../conf/escheduler/conf:/opt/escheduler/conf:ro
      - ../../multi/docker-compose/worker/worker-extra-init.sh:/opt/escheduler/script/worker-extra-init.sh
      - /usr/hdp/current/spark2-client:/opt/spark2-client:ro
      - /etc/spark2/2.6.3.0-235/0:/etc/spark2/2.6.3.0-235/0:ro
  dolphinscheduler-alert:
    container_name: dolphinscheduler-alert
    image: gary0416/dolphinscheduler-common:test-20191011-1831
    restart: always
    networks:
      - dolphinscheduler
    environment:
      ALERT_SERVER_STARTUP_DELAY: 180
    command: alert
    ports:
      - 7789:7789
    volumes:
      - ./data/logs:/opt/escheduler/logs
      - ../../conf/escheduler/conf:/opt/escheduler/conf:ro
```

# 初始化
HDFS:
```
su - hdfs
hdfs dfs -mkdir /escheduler
hdfs dfs -chown escheduler:escheduler /escheduler
```
common.properties:
- res.upload.startup.type需设置成HDFS（默认NONE不支持文件上传，fs没有初始化，而ResourcesService.verifyResourceName调用HadoopUtils.exists，会NPE）

nginx - default.conf:
- client_max_body_size 1024m;否则上传大的fat jar时报错413 Request Entity Too Large

# 使用备忘
- 架构图https://user-images.githubusercontent.com/48329107/62609545-8f973480-b934-11e9-9a58-d8133222f14d.png
- 本机测试可以不起logger(界面点击查看日志,api服务器通过gRPC访问worker节点取日志)和alert(邮件通知)服务
- 新建队列(对应yarn队列)、worker分组、租户、项目、给用户授权项目
- 安全性：项目、UDF、数据源，默认可见自己租户的，用admin授权给其它租户(新版已删除)，授权后可删改，无只读。
- 执行SQL一般用于修改数据，如果是select，则会邮件形式发送查询结果
- hive数据源，zk方式，端口需2181。官方文档截图的10000有误
- shell执行用户为租户code，会自动新建租户同名用户(组为root)。如果有写文件，需确保有权限
- worker分组Default表示可以在任一worker上执行
- hdfs HA，俩xml放conf下
- 租户即大数据的用户
- 使用时间参数需要先在全局里定义，不能直接使用。其实allParamMap是有默认3个时间参数的，cn/escheduler/common/utils/ParameterUtils.java:140判断如果是$开头的，才会放到resolveMap里，最后作为实际参数。所以需要重命名或转换下，让值是$开头，才能在任务中获取到参数。
- 同一父节点的两个子流程不是并行的
- $[MMdd-5]这种依赖于system.datetime，否则是当前时间（ParameterUtils.java:60），可通过自定义参数system.datetime=${system.biz.date}0000000赋值成业务时间。参数这里做了些易用性的修改，见https://github.com/apache/incubator-dolphinscheduler/pull/913，暂未merge
- Hive SQL里不支持$[]的替换，需要写在自定义参数中，例如param.dd in varchar $[dd]
- Spark目前只有程序参数支持时间参数，--conf等其他参数不支持，修改方法在分支spark-other-params-missing-local-param，使用场景较少，留做备用，未提交PR
- 任务是否可以执行，会检查系统资源(OSUtils.java:277)，master和worker目前相同：load不超过cpu核数乘以2(master.max.cpuload.avg=10)、可用内存大于总内存十分之一(master.reserved.memory=1)。否则等下一个轮寻间隔(1s)
- 对单个节点补数，会依次执行后续子节点
- DAG图太大(几十M时，一般几K)，浏览器会打不开。用子流程拆开，缺点是流程实例里看不到每个环节的甘特图
- 流程依赖只能算是一项额外检查，不满足条件时不运行，用子流程才能实现当A运行后，立即执行B。需要AB都上线，在A上设定时

# 补数
目前只是按天补，对于小时级的任务，无法通过cron表达式推算出运行时间

cn/escheduler/server/master/runner/MasterExecThread.java:202
目前是scheduleDate = DateUtils.getSomeDay(scheduleDate, 1)

cn/escheduler/common/utils/placeholder/BusinessTimeUtils.java:43
获取下一次运行时间schedulerService.previewSchedule(loginUser, projectName, schedule)

https://github.com/apache/incubator-dolphinscheduler/issues/245

对其中一个环节补2019-09-23到25号的过程:
```
curl 'http://localhost:8888/escheduler/projects/dev-test/executors/start-process-instance' -H 'Sec-Fetch-Mode: cors' -H 'Sec-Fetch-Site: same-origin' -H 'Origin: http://localhost:8888' -H 'Accept-Encoding: gzip, deflate, br' -H 'Accept-Language: zh-CN,zh;q=0.9,en;q=0.8' -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.90 Safari/537.36' -H 'Content-Type: application/x-www-form-urlencoded' -H 'Accept: application/json, text/plain, */*' -H 'Referer: http://localhost:8888/' -H 'Cookie: sessionId=aff85882-0331-4e1f-b64d-409f267cbf1d; grafana_session=76d33dde4c53c0e6350724c0b6f7aed1; language=zh_CN' -H 'Connection: keep-alive' -H 'DNT: 1' --data 'processDefinitionId=2&scheduleTime=2019-09-23+00%3A00%3A00%2C2019-09-25+00%3A00%3A00&failureStrategy=CONTINUE&warningType=NONE&warningGroupId=0&execType=COMPLEMENT_DATA&startNodeList=echo+date+param&taskDependType=TASK_POST&runMode=RUN_MODE_SERIAL&processInstancePriority=MEDIUM&receivers=&receiversCc=&workerGroupId=-1' --compressed

会返回{"code":0,"msg":"success","data":null}

实际
INSERT INTO `escheduler`.`t_escheduler_command`(`id`, `command_type`, `process_definition_id`, `command_param`, `task_depend_type`, `failure_strategy`, `warning_type`, `warning_group_id`, `schedule_time`, `start_time`, `executor_id`, `dependence`, `update_time`, `process_instance_priority`, `worker_group_id`) VALUES (1, 5, 2, '{\"StartNodeNameList\":\"echo date param\",\"complementEndDate\":\"2019-09-25 00:00:00\",\"complementStartDate\":\"2019-09-23 00:00:00\"}', 2, 1, 0, 0, NULL, '2019-09-25 11:59:53', 2, NULL, '2019-09-25 11:59:53', 2, -1);

然后master每1秒轮寻，创建流程实例
cn/escheduler/server/master/runner/MasterSchedulerThread.java:87
插入t_escheduler_process_instance表

master查到后拆成task，前面只选了一个环节，所以是1个task，同时删除t_escheduler_command，错误的会转存到t_escheduler_error_command表，message列有错误原因，如变量循环引用。
插入t_escheduler_task_instance表

task少量信息放queue(存在zk里/escheduler/tasks_queue/2_1_2_1_-1，无数据，文档写压测过百万数据)

worker cn/escheduler/server/worker/runner/FetchTaskThread.java:146从队列取，加锁
执行cn/escheduler/server/worker/runner/TaskScheduleThread.java:92
```

# 常见错误
## 无法上传UDF
后台空指针cn.escheduler.api.service.ResourcesService.verifyResourceName(ResourcesService.java:445)，因为tenantCode为空。

需先新建租户tenant-dev，然后在用户管理里，修改租户、队列，或直接新建用户。

## 无法查看执行日志
其实在/opt/escheduler/logs/3/1下，新版已修复(注意1.1.0时，dev和1.1.0tag的配置不一致，需修改Logback）

## 日志时区错误
已修改dockerfile，注意同时修改timezone和localtime。

# 遗留问题
## 权限不严格
- 在保存dag时，可选其它租户。
- 选流程依赖时，可选其他用户的项目。

## 不能重复
- 不同租户，UDF文件名不能重复，尽管在不同的HDFS目录下。
- 不同租户，UDF函数名不能重复，尽管是临时函数。

即，测试环境的SQL，与正式环境的SQL不一致，需修改函数名，除非两套环境。

## 任务kill无效
kill后仍在运行。
