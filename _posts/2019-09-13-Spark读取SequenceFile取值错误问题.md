---
layout:     post
title:      Spark读取SequenceFile取值错误问题
subtitle:   
date:       2019-09-13
author:     gary
header-img: 
catalog: true
tags:
    - Spark
---

# 现象
读取SequenceFile并转成Tuple，在后续使用时，取值都是文件里最后一条记录的值，而不是实际文件内容。

例如：
SequenceFile内容为k1,v1和k2,v2两行。rdd foreach输出，两条都是k2。

读取SequenceFile，最终生成Tuple3(文件名，key，val)的rdd，则错误示例代码如下：
```
String path = "/your-path/file.data";
try (JavaSparkContext spark = SparkUtils.getJavaSparkContext()) {
    JavaNewHadoopRDD<Text, BytesWritable> recordsRdd = (JavaNewHadoopRDD<Text, BytesWritable>) spark
            .newAPIHadoopFile(path, SequenceFileInputFormat.class,
                    Text.class, BytesWritable.class, spark.hadoopConfiguration());

    JavaRDD<Tuple3<String, Text, String>> tuple3JavaRDD = recordsRdd.mapPartitionsWithInputSplit((Function2<InputSplit,
            Iterator<Tuple2<Text, BytesWritable>>,
            Iterator<Tuple3<String, Text, String>>>) (is, dataIterator) -> {
        String filePath = ((FileSplit) is).getPath().toString();
        List<Tuple3<String, Text, String>> list = new LinkedList<>();
        while (dataIterator.hasNext()) {
            Tuple2<Text, BytesWritable> data = dataIterator.next();
            BytesWritable value = data._2();
            value.setCapacity(value.getLength());
            String valueContent = new String(value.getBytes(), StandardCharsets.UTF_8);
            // 下面的第二个值（即key）错误
            list.add(new Tuple3<>(filePath, data._1(), valueContent));
        }
        return list.iterator();
    }, true);

    tuple3JavaRDD.foreach((VoidFunction<Tuple3<String, Text, String>>) v -> {
        // v._2始终是最后一行的k2
        logger.debug("fileName:{},key:{},val length:{}", v._1(), v._2(), v._3().length());
    });
}
```

# 原因
一句话总结：
**同一实例，只读新值。**

其实上例中，BytesWritable的value也会遇到同样问题，只是碰巧想转成byte[]方便后续使用而避开了。
此外BytesWritable取值需注意capacity和length的关系。

源码分析(以key为例，value同样存在问题)：
当spark程序中，iterator.next()时，deserializeKey会把key传进去
hadoop-common-2.7.3.2.6.3.0-235-sources.jar!/org/apache/hadoop/io/SequenceFile.java:2577：
```
2562:public synchronized Object next(Object key) throws IOException {
2563:      if (key != null && key.getClass() != getKeyClass()) {
2564:        throw new IOException("wrong key class: "+key.getClass().getName()
2565:                              +" is not "+keyClass);
2566:      }
2567:
2568:      if (!blockCompressed) {
2569:        outBuf.reset();
2570:        
2571:        keyLength = next(outBuf);
2572:        if (keyLength < 0)
2573:          return null;
2574:        
2575:        valBuffer.reset(outBuf.getData(), outBuf.getLength());
2576:        
2577:        key = deserializeKey(key);
        ...
```

下一步是hadoop-common-2.7.3.2.6.3.0-235-sources.jar!/org/apache/hadoop/io/serializer/WritableSerialization.java:63
```
public Writable deserialize(Writable w) throws IOException {
  Writable writable;
  if (w == null) {
    writable 
      = (Writable) ReflectionUtils.newInstance(writableClass, getConf());
  } else {
    writable = w;
  }
  writable.readFields(dataIn);
  return writable;
}
```

因为是Text类型，所以readFields方法会进到hadoop-common-2.7.3.2.6.3.0-235-sources.jar!/org/apache/hadoop/io/Text.java:289
```
public void readFields(DataInput in) throws IOException {
  int newLength = WritableUtils.readVInt(in);
  readWithKnownLength(in, newLength);
}
```

先从in里获取长度，然后317行的readWithKnownLength方法如下：
```
public void readWithKnownLength(DataInput in, int len) throws IOException {
  setCapacity(len, false);
  in.readFully(bytes, 0, len);
  length = len;
}
```

可以看到，上面全程没有创建Text对象（其它Writeable同理），只是通过setCapacity和in.readFully读取新值，所以如果程序中像上面示例一样，直接把key对象（例如Text类型的）加入到集合中，随着iterator遍历，加入到集合里的key对象其实是同一个引用，只是值在不停的变。

# 解决
直接取内容。

# 正确示例
```
String path = "/your-path/file.data";
try (JavaSparkContext spark = SparkUtils.getJavaSparkContext()) {
    JavaNewHadoopRDD<Text, BytesWritable> recordsRdd = (JavaNewHadoopRDD<Text, BytesWritable>) spark
            .newAPIHadoopFile(path, SequenceFileInputFormat.class,
                    Text.class, BytesWritable.class, spark.hadoopConfiguration());

    JavaRDD<Tuple3<String, String, String>> tuple3JavaRDD = recordsRdd.mapPartitionsWithInputSplit((Function2<InputSplit,
            Iterator<Tuple2<Text, BytesWritable>>,
            Iterator<Tuple3<String, String, String>>>) (is, dataIterator) -> {
        String filePath = ((FileSplit) is).getPath().toString();
        List<Tuple3<String, String, String>> list = new LinkedList<>();
        while (dataIterator.hasNext()) {
            Tuple2<Text, BytesWritable> data = dataIterator.next();
            BytesWritable value = data._2();
            value.setCapacity(value.getLength());
            String valueContent = new String(value.getBytes(), StandardCharsets.UTF_8);
            // 只改了这一行，可以新建个Text对象，或像下面一样(注意Tuple类型改变需改动相关几处代码)
            list.add(new Tuple3<>(filePath, data._1().toString(), valueContent));
        }
        return list.iterator();
    }, true);

    tuple3JavaRDD.foreach((VoidFunction<Tuple3<String, String, String>>) v -> {
        // 此时v._2()输出就是正常的每行数据实际值了
        logger.debug("fileName:{},key:{},val length:{}", v._1(), v._2(), v._3().length());
    });
}
```

# 拓展
hadoop其它Writeable类型其实也是同理，都是复用对象，只设置值，而不是每次都new。

# spark debug时的注意事项
例如：连续两行日志输出，断点在第二行输出上。当进入断点后，F8继续，会看到输出结果不一致，原因同上。

JAVA程序debug时无此问题，Spark程序设置单线程（cpu数、并行等）也无法避免，同时观察到，第二行输出的错误值始终是同样的，留坑待填。

目前对spark理解还不是很深，只尝试了spark.driver.cores=1，spark.executor.cores=1，spark.master=local[1]等参数，分区数是1。
在SparkUI上观察，确实也只有一个Executor，执行唯一的1个Task，1核执行中。程序里的日志输出，线程也都是Executor task launch worker for task 0，以上均说明没有并行执行。
