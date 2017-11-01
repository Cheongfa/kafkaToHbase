# kafkaToHbase

## 系统介绍

本系统由Kafka，spark streaming，hbase组成，这里主要用于日志处理，采用Spark streaming中的Direct方式来消费kafka中的数据。
目前处理的日志量大约在 1亿/天，系统会将kafka流中的数据进行处理，加工格式化，最终序列化到Hbase中。

* **机器环境如下表格：**

host      | zk | Namenode | datenode | resourcemanager | nodemanager |spark | hbase
----------|----|----------|----------|-----------------|-------------|------|------
redhat-01 |  √ |  √       |          |                 |             |      |  
redhat-02 |  √ |  √       |          |                 |             |      |  
redhat-03 |  √ |          |          |       √         |             |      |  
redhat-04 |    |          |    √     |                 |      √      |  √   | √
redhat-05 |    |          |    √     |                 |      √      |  √   | √
redhat-06 |    |          |    √     |                 |      √      |  √   | √

* **版本列表**

zk    | hadoop | jdk       | spark | hbase 
------|--------|-----------|-------|-------
3.4.9 |  2.7.4 | 1.8.0_131 | 2.1.0 | 1.2.6          

## 填坑区

* saveAsHadoopDataset为旧的API，该API存在bug，在连接完ZK后不会释放资源，导致ZK出现最大连接上限或者IO异常
* 建议使用KryoSerializer替换默认的序列化方式，可以大大降低内存使用，节约资源提高效率。（需要注册使用的类）
* 建议使用spark.executor.extraJavaOptions=-XX:+UseConcMarkSweepGC
* 建议设置spark.streaming.kafka.maxRatePerPartition=?，如果发现spark消费的速度过慢时，请检查当前topic的partition数量，并在前面的“？”中设置合理的数值。spark每秒能处理的值为--num-executors * 前面？处的值。
* 若代码逻辑过为复杂，导致DAG过长，或者代码中大量使用了递归函数，此时应该适当的增加JVM的内存上限，典型错误为OOM，StackOverFlowError。设置项为spark.driver.extraJavaOptions: -XX:PermSize=512M -MaxPermSize-1G
