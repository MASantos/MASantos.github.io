# Standalone Mode with multiple cores

## Overview

Let's try running Hadoop in standalone mode but using multiple cores.
I couldn't find anything within the [2.6.2 documentation](file:///Users/msantos/System/HADOOP/hadoop-2.6.2/share/doc/hadoop/hadoop-project-dist/hadoop-common/SingleCluster.html), but found this answered in [StackOverflow](http://stackoverflow.com/questions/3546025/is-it-possible-to-run-hadoop-in-pseudo-distributed-operation-without-hdfs).
I'll be following the this example closely.

# Configuration

Using "file:///" for fs.default.name doesn't work. Even after using mapreduce.jobtracker.address as "localhost:50030" (different from 'local')

Config changes are:
1. In core-site.xml. The only property included is
```
        <property>
                <name>fs.default.name</name>
                <value>file:///</value>
        </property>
```
2. mapred-site.xml contains now (previously non-existent):
```
<configuration>
<property>
        <name>mapreduce.job.tracker</name>
        <value>localhost:9021</value>
</property>
<property>
        <name>mapreduce.jobtracker.address</name>
        <value>localhost:50030</value>
</property>
```
Log file hadoop-msantos-namenode-MBP-2.local.log gives folllowing error :
```
2015-11-19 12:59:42,782 INFO org.apache.hadoop.hdfs.server.namenode.NameNode: registered UNIX signal handlers for [TERM, HUP, INT]
2015-11-19 12:59:42,787 INFO org.apache.hadoop.hdfs.server.namenode.NameNode: createNameNode []
2015-11-19 12:59:43,560 INFO org.apache.hadoop.metrics2.impl.MetricsConfig: loaded properties from hadoop-metrics2.properties
2015-11-19 12:59:44,226 INFO org.apache.hadoop.metrics2.impl.MetricsSystemImpl: Scheduled snapshot period at 10 second(s).
2015-11-19 12:59:44,226 INFO org.apache.hadoop.metrics2.impl.MetricsSystemImpl: NameNode metrics system started
2015-11-19 12:59:44,241 INFO org.apache.hadoop.hdfs.server.namenode.NameNode: fs.defaultFS is file:///
2015-11-19 12:59:44,580 WARN org.apache.hadoop.util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
2015-11-19 12:59:45,015 FATAL org.apache.hadoop.hdfs.server.namenode.NameNode: Failed to start namenode.
java.lang.IllegalArgumentException: Invalid URI for NameNode address (check fs.defaultFS): file:/// has no authority.
```

And log file hadoop-msantos-datanode-MBP-2.local.log gives error:
```
2015-11-19 13:01:51,906 FATAL org.apache.hadoop.hdfs.server.datanode.DataNode: Exception in secureMain
java.io.IOException: Incorrect configuration: namenode address dfs.namenode.servicerpc-address or dfs.namenode.rpc-address is not configured.
```


