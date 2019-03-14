---
layout:     post
title:      SparkStreaming的ActiveBatches出现堆积
subtitle:   
date:       2019-03-13
author:     gary
header-img: 
catalog: true
tags:
    - Spark
---

# 现象
从Spark UI看，Active Batches出现堆积。
![error](/img/in-post/2019-03-13-SparkStreaming的ActiveBatches出现堆积/1.png)

# 原因
event队列满

# 解决
```
// 默认10000,增大
sparkConf.set("spark.scheduler.listenerbus.eventqueue.size", "100000");
```

# 附:排查过程
## 方法一：
根据程序逻辑去查询对应时段是否有数据,如下图:
![data](/img/in-post/2019-03-13-SparkStreaming的ActiveBatches出现堆积/2.png)

## 方法二：
点击任意active batches查看详情,例如16分13秒的,如下图:
![16分13秒的](/img/in-post/2019-03-13-SparkStreaming的ActiveBatches出现堆积/3.png)
再点击红色箭头的Job Id 40120继续查看,如下图:
![40120](/img/in-post/2019-03-13-SparkStreaming的ActiveBatches出现堆积/4.png)
注意红框显示的是0/10, 继续点击红色箭头所示连接,如下图:
![80240](/img/in-post/2019-03-13-SparkStreaming的ActiveBatches出现堆积/5.png)
可以看到状态其实都是已完成,并没有像UI显示的那样完成进度是0/10. 

使用
```
yarn logs -applicationId application_xxx > application_xxx.log
```
导出运行日志(在网页上已无法查看,太大了),查找任意jtask来验证,例如上图第一行的TaskID是1118440,则在log中搜索TID 1188440,如下图:
![1188440](/img/in-post/2019-03-13-SparkStreaming的ActiveBatches出现堆积/6.png)
看红色箭头那一行,确实是已执行完成.

**结论: 只是网页显示不正确，没影响数据**

继续翻看日志发现:
```
19/03/12 13:16:09 ERROR LiveListenerBus: Dropping SparkListenerEvent because no remaining room in event queue. This likely means one of the SparkListeners is too slow and cannot keep up with the rate at which tasks are being started by the scheduler.
```

在Jira上看到相似问题
[https://issues.apache.org/jira/browse/SPARK-18881?focusedCommentId=16030025&page=com.atlassian.jira.plugin.system.issuetabpanels%3Acomment-tabpanel#comment-16030025](https://issues.apache.org/jira/browse/SPARK-18881?focusedCommentId=16030025&page=com.atlassian.jira.plugin.system.issuetabpanels%3Acomment-tabpanel#comment-16030025 "SPARK-18881")

提到:
```
Just to mention a workaround for those experiencing the problem : try increase spark.scheduler.listenerbus.eventqueue.size (default 10000). 
    It may only postpone the problem, if the queue filling is faster than listeners for a long time. In our case, we have bursts of activity and raising this limit helps.
```

增大后问题解决
