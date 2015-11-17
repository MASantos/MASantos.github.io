---
layout: post
category : lessons
tagline: "First steps with Haddop"
tags : [intro, beginner, Hadoop, OSX 10.6.8]
---

# First tests with Hadoop 2.6.5 on my MacBook Pro OSX 10.6.8

## Index
* Overview
* Hadoop install & config
* Examples
* Hadoop standalone: Java Mapper and Reducer
*  Conclusions
* Why Should I Care?

## Overview

Recording what happened while getting a hadoop version running and testing the first java, python and 
bash scripts as hadoop standalone and streaming runs.

## Hadoop install & config

Installing & running Hadoop in OSX 10.6.8

1. Downloaded hadoop 2.7.1 binary. I have JAVA 1.6.0_65. when running the basic first test I get error :
```
$bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.1.jar grep input test/output 'dfs[a-z.]+'
Exception in thread "main" java.lang.UnsupportedClassVersionError: org/apache/hadoop/util/RunJar : Unsupported major.minor version 51.0
        at java.lang.ClassLoader.defineClass1(Native Method)
        at java.lang.ClassLoader.defineClassCond(ClassLoader.java:637)
        at java.lang.ClassLoader.defineClass(ClassLoader.java:621)
        at java.security.SecureClassLoader.defineClass(SecureClassLoader.java:141)
        at java.net.URLClassLoader.defineClass(URLClassLoader.java:283)
        at java.net.URLClassLoader.access$000(URLClassLoader.java:58)
        at java.net.URLClassLoader$1.run(URLClassLoader.java:197)
        at java.security.AccessController.doPrivileged(Native Method)
        at java.net.URLClassLoader.findClass(URLClassLoader.java:190)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:306)
        at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:301)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:247)
```
 Latest (as of Thu Nov. 12 2015) JDK jdk-8u65-nb-8_1-macosx-x64.dmg requires OSX 8 or higher. 
 I downloaded  JDK 7_u80, but it requires OS X 10.7.3 or higher

2. Downloaded hadoop 2.5.2 binary, installed and ...it worked. At least so far the basic test
```
bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.5.2.jar grep test/input test/output 'dfs[a-z.]+'
```

3. Downloaded hadoop 2.6.2 binary, installed and it worked as well -again, so far tested just the basic test above
 This is the latest version of the 2.6 branch released Oct 2015. The 2.7.1 version was released on Jul 2015.

## Examples

### data

HCN: NOOA's US historical climate network data. 

### Reference: The plain bash script (no hadoop)

This will be the reference to compare to simply because it's the **fastest**. Hadoop as standalone
can't beat this.
```
$time ./max_temp_hcn.sh > hcn-max

real	0m48.068s
user	0m35.809s
sys	0m8.038s

$sort -n hcn-max | head
1853 348 USC00381310185308TMAX  348
1868 161 USC00144559186810TMAX  161
1869 306 USC00144559186908TMAX  306
1874 256 USW00013724187406TMAX  256
1875 306 USW00013724187507TMAX  306
1876 294 USW00013724187607TMAX  294
1876 306 USW00094728187609TMAX  306
1877
1877
1877 289 USW00013724187709TMAX  289
```
Getting now the max per year for all stations: ./max_temp_hcn-all.sh
```bash
#!/bin/bash

cat input/data/NCDC/ghcnd_hcn/*.dly|  awk /TMAX/'{ year=substr($0,12,4); temp=substr($0,22,5)+0;
        q=substr($0,28,1);
        if( temp!=9999 && q ~ /[[:space:]]/ && temp > max[year]) { max[year] = temp ; l[year]=substr($0,1,28)} }
      END {for( yr in max) print yr,max[yr],l[yr]}'

$time ./max_temp_hcn-all.sh > hcn-max-all

real	0m29.214s
user	0m27.698s
sys	0m3.862s
```

## Hadoop standalone: Java Mapper and Reducer
```
$hadoop MaxTemperature input/data/NCDC/ghcnd_hcn/USC00011084.dly output_hcn
Exception in thread "main" java.lang.NoClassDefFoundError: MaxTemperature
Caused by: java.lang.ClassNotFoundException: MaxTemperature
	at java.net.URLClassLoader$1.run(URLClassLoader.java:202)
	at java.security.AccessController.doPrivileged(Native Method)
	at java.net.URLClassLoader.findClass(URLClassLoader.java:190)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:306)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:301)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:247)
```

Needed to compile: See example HelloWorld
```
$cat HelloWorld.java
public class HelloWorld {
    public static void main(String args[]) {
        System.out.println("Hello World!"); 
    }
}
$java -classpath . HelloWorld.java
$hadoop HelloWorld . out
Hello World!
```
So let's compile MaxTemperature and proceed:
```
msantos@MBP-2[19:49]:~/System/HADOOP/current/test/classes$javac -classpath `hadoop classpath` MaxTemperature.java
Note: MaxTemperature.java uses or overrides a deprecated API.
Note: Recompile with -Xlint:deprecation for details.

msantos@MBP-2[19:53]:~/System/HADOOP/current/test$hadoop MaxTemperature input/data/NCDC/ghcnd_hcn/USC00011084.dly output_hcn
15/11/12 19:53:15 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
15/11/12 19:53:15 INFO Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
15/11/12 19:53:15 INFO jvm.JvmMetrics: Initializing JVM Metrics with processName=JobTracker, sessionId=
15/11/12 19:53:16 WARN mapreduce.JobResourceUploader: Hadoop command-line option parsing not performed. Implement the Tool interface and execute your application with ToolRunner to remedy this.
15/11/12 19:53:16 WARN mapreduce.JobResourceUploader: No job jar file set.  User classes may not be found. See Job or Job#setJar(String).
15/11/12 19:53:16 INFO input.FileInputFormat: Total input paths to process : 1
15/11/12 19:53:16 INFO mapreduce.JobSubmitter: number of splits:1
15/11/12 19:53:16 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_local1442023825_0001
15/11/12 19:53:17 INFO mapred.LocalJobRunner: OutputCommitter set in config null
15/11/12 19:53:17 INFO mapred.LocalJobRunner: OutputCommitter is org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter
15/11/12 19:53:17 INFO mapreduce.Job: The url to track the job: http://localhost:8080/
15/11/12 19:53:17 INFO mapreduce.Job: Running job: job_local1442023825_0001
15/11/12 19:53:17 INFO mapred.LocalJobRunner: Starting task: attempt_local1442023825_0001_m_000000_0
15/11/12 19:53:17 INFO mapred.LocalJobRunner: Waiting for map tasks
15/11/12 19:53:17 INFO util.ProcfsBasedProcessTree: ProcfsBasedProcessTree currently is supported only on Linux.
15/11/12 19:53:17 INFO mapred.Task:  Using ResourceCalculatorProcessTree : null
15/11/12 19:53:17 INFO mapred.MapTask: Processing split: file:/Users/msantos/System/HADOOP/hadoop-2.6.2/test/input/data/NCDC/ghcnd_hcn/USC00011084.dly:0+1700190
15/11/12 19:53:17 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
15/11/12 19:53:17 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
15/11/12 19:53:17 INFO mapred.MapTask: soft limit at 83886080
15/11/12 19:53:17 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
15/11/12 19:53:17 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
15/11/12 19:53:17 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
15/11/12 19:53:18 INFO mapred.LocalJobRunner:
15/11/12 19:53:18 INFO mapred.MapTask: Starting flush of map output
15/11/12 19:53:18 INFO mapred.Task: Task:attempt_local1442023825_0001_m_000000_0 is done. And is in the process of committing
15/11/12 19:53:18 INFO mapred.LocalJobRunner: map
15/11/12 19:53:18 INFO mapred.Task: Task 'attempt_local1442023825_0001_m_000000_0' done.
15/11/12 19:53:18 INFO mapred.LocalJobRunner: Finishing task: attempt_local1442023825_0001_m_000000_0
15/11/12 19:53:18 INFO mapred.LocalJobRunner: map task executor complete.
15/11/12 19:53:18 INFO mapred.LocalJobRunner: Waiting for reduce tasks
15/11/12 19:53:18 INFO mapred.LocalJobRunner: Starting task: attempt_local1442023825_0001_r_000000_0
15/11/12 19:53:18 INFO util.ProcfsBasedProcessTree: ProcfsBasedProcessTree currently is supported only on Linux.
15/11/12 19:53:18 INFO mapred.Task:  Using ResourceCalculatorProcessTree : null
15/11/12 19:53:18 INFO mapred.ReduceTask: Using ShuffleConsumerPlugin: org.apache.hadoop.mapreduce.task.reduce.Shuffle@669a4cb
15/11/12 19:53:18 INFO reduce.MergeManagerImpl: MergerManager: memoryLimit=372781856, maxSingleShuffleLimit=93195464, mergeThreshold=246036032, ioSortFactor=10, memToMemMergeOutputsThreshold=10
15/11/12 19:53:18 INFO reduce.EventFetcher: attempt_local1442023825_0001_r_000000_0 Thread started: EventFetcher for fetching Map Completion Events
15/11/12 19:53:18 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local1442023825_0001_m_000000_0 decomp: 2 len: 6 to MEMORY
15/11/12 19:53:18 INFO reduce.InMemoryMapOutput: Read 2 bytes from map-output for attempt_local1442023825_0001_m_000000_0
15/11/12 19:53:18 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 2, inMemoryMapOutputs.size() -> 1, commitMemory -> 0, usedMemory ->2
15/11/12 19:53:18 INFO reduce.EventFetcher: EventFetcher is interrupted.. Returning
15/11/12 19:53:18 INFO mapred.LocalJobRunner: 1 / 1 copied.
15/11/12 19:53:18 INFO reduce.MergeManagerImpl: finalMerge called with 1 in-memory map-outputs and 0 on-disk map-outputs
15/11/12 19:53:18 INFO mapred.Merger: Merging 1 sorted segments
15/11/12 19:53:18 INFO mapred.Merger: Down to the last merge-pass, with 0 segments left of total size: 0 bytes
15/11/12 19:53:18 INFO reduce.MergeManagerImpl: Merged 1 segments, 2 bytes to disk to satisfy reduce memory limit
15/11/12 19:53:18 INFO reduce.MergeManagerImpl: Merging 1 files, 6 bytes from disk
15/11/12 19:53:18 INFO reduce.MergeManagerImpl: Merging 0 segments, 0 bytes from memory into reduce
15/11/12 19:53:18 INFO mapred.Merger: Merging 1 sorted segments
15/11/12 19:53:18 INFO mapred.Merger: Down to the last merge-pass, with 0 segments left of total size: 0 bytes
15/11/12 19:53:18 INFO mapred.LocalJobRunner: 1 / 1 copied.
15/11/12 19:53:18 INFO Configuration.deprecation: mapred.skip.on is deprecated. Instead, use mapreduce.job.skiprecords
15/11/12 19:53:18 INFO mapred.Task: Task:attempt_local1442023825_0001_r_000000_0 is done. And is in the process of committing
15/11/12 19:53:18 INFO mapred.LocalJobRunner: 1 / 1 copied.
15/11/12 19:53:18 INFO mapred.Task: Task attempt_local1442023825_0001_r_000000_0 is allowed to commit now
15/11/12 19:53:18 INFO mapreduce.Job: Job job_local1442023825_0001 running in uber mode : false
15/11/12 19:53:18 INFO mapreduce.Job:  map 100% reduce 0%
15/11/12 19:53:18 INFO output.FileOutputCommitter: Saved output of task 'attempt_local1442023825_0001_r_000000_0' to file:/Users/msantos/System/HADOOP/hadoop-2.6.2/test/output_hcn/_temporary/0/task_local1442023825_0001_r_000000
15/11/12 19:53:18 INFO mapred.LocalJobRunner: reduce > reduce
15/11/12 19:53:18 INFO mapred.Task: Task 'attempt_local1442023825_0001_r_000000_0' done.
15/11/12 19:53:18 INFO mapred.LocalJobRunner: Finishing task: attempt_local1442023825_0001_r_000000_0
15/11/12 19:53:18 INFO mapred.LocalJobRunner: reduce task executor complete.
15/11/12 19:53:19 INFO mapreduce.Job:  map 100% reduce 100%
15/11/12 19:53:19 INFO mapreduce.Job: Job job_local1442023825_0001 completed successfully
15/11/12 19:53:19 INFO mapreduce.Job: Counters: 30
	File System Counters
		FILE: Number of bytes read=3400854
		FILE: Number of bytes written=508728
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
	Map-Reduce Framework
		Map input records=6297
		Map output records=0
		Map output bytes=0
		Map output materialized bytes=6
		Input split bytes=158
		Combine input records=0
		Combine output records=0
		Reduce input groups=0
		Reduce shuffle bytes=6
		Reduce input records=0
		Reduce output records=0
		Spilled Records=0
		Shuffled Maps =1
		Failed Shuffles=0
		Merged Map outputs=1
		GC time elapsed (ms)=11
		Total committed heap usage (bytes)=395960320
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters
		Bytes Read=1700190
	File Output Format Counters
		Bytes Written=8
msantos@MBP-2[19:53]:~/System/HADOOP/current/test$
```
However there is NO OUTPUT from the reduce task: "Reduce output records=0". Something is wrong with the java code obvously.
What is it really doing? Surely it's not reading the records correctly, i.e., not finding the temperature and TMAX strings.
Debugging goal: Lets just read a couple of strings from each record and have the reducer task just print this out. No processing.
This changes the type of the map & reduce classes. Their template have 4 types. My guess is first 2 are for their respective input,
and the other 2 are for their outputs. However, it turns out this is not enough for hadoop to be able to run it. The java compilation
works but hadoop API complains that map is passing an unexpected type: The main interface must declare the right type for the reducer as well!!
Here the error message:
```
15/11/13 12:18:23 WARN mapred.LocalJobRunner: job_local858928967_0001
java.lang.Exception: java.io.IOException: Type mismatch in value from map: expected org.apache.hadoop.io.IntWritable, received org.apache.hadoop.io.Text
	at org.apache.hadoop.mapred.LocalJobRunner$Job.runTasks(LocalJobRunner.java:462)
	at org.apache.hadoop.mapred.LocalJobRunner$Job.run(LocalJobRunner.java:522)
Caused by: java.io.IOException: Type mismatch in value from map: expected org.apache.hadoop.io.IntWritable, received org.apache.hadoop.io.Text
	at org.apache.hadoop.mapred.MapTask$MapOutputBuffer.collect(MapTask.java:1074)
	at org.apache.hadoop.mapred.MapTask$NewOutputCollector.write(MapTask.java:712)
	at org.apache.hadoop.mapreduce.task.TaskInputOutputContextImpl.write(TaskInputOutputContextImpl.java:89)
	at org.apache.hadoop.mapreduce.lib.map.WrappedMapper$Context.write(WrappedMapper.java:112)
	at MaxTemperatureMapper.map(MaxTemperatureMapper.java:33)
```
Once corrected, compiled and run we get:
```
msantos@MBP-2[12:21]:~/System/HADOOP/current/test$!hadoop
hadoop MaxTemperature input/data/NCDC/ghcnd_hcn/USC00011084.dly output_hcn
15/11/13 12:21:46 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
15/11/13 12:21:46 INFO Configuration.deprecation: session.id is deprecated. Instead, use dfs.metrics.session-id
15/11/13 12:21:46 INFO jvm.JvmMetrics: Initializing JVM Metrics with processName=JobTracker, sessionId=
15/11/13 12:21:47 WARN mapreduce.JobResourceUploader: Hadoop command-line option parsing not performed. Implement the Tool interface and execute your application with ToolRunner to remedy this.
15/11/13 12:21:47 WARN mapreduce.JobResourceUploader: No job jar file set.  User classes may not be found. See Job or Job#setJar(String).
15/11/13 12:21:47 INFO input.FileInputFormat: Total input paths to process : 1
15/11/13 12:21:47 INFO mapreduce.JobSubmitter: number of splits:1
15/11/13 12:21:48 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_local1636203858_0001
15/11/13 12:21:48 INFO mapred.LocalJobRunner: OutputCommitter set in config null
15/11/13 12:21:48 INFO mapred.LocalJobRunner: OutputCommitter is org.apache.hadoop.mapreduce.lib.output.FileOutputCommitter
15/11/13 12:21:48 INFO mapreduce.Job: The url to track the job: http://localhost:8080/
15/11/13 12:21:48 INFO mapreduce.Job: Running job: job_local1636203858_0001
15/11/13 12:21:48 INFO mapred.LocalJobRunner: Starting task: attempt_local1636203858_0001_m_000000_0
15/11/13 12:21:48 INFO mapred.LocalJobRunner: Waiting for map tasks
15/11/13 12:21:49 INFO util.ProcfsBasedProcessTree: ProcfsBasedProcessTree currently is supported only on Linux.
15/11/13 12:21:49 INFO mapred.Task:  Using ResourceCalculatorProcessTree : null
15/11/13 12:21:49 INFO mapred.MapTask: Processing split: file:/Users/msantos/System/HADOOP/hadoop-2.6.2/test/input/data/NCDC/ghcnd_hcn/USC00011084.dly:0+1700190
15/11/13 12:21:49 INFO mapred.MapTask: (EQUATOR) 0 kvi 26214396(104857584)
15/11/13 12:21:49 INFO mapred.MapTask: mapreduce.task.io.sort.mb: 100
15/11/13 12:21:49 INFO mapred.MapTask: soft limit at 83886080
15/11/13 12:21:49 INFO mapred.MapTask: bufstart = 0; bufvoid = 104857600
15/11/13 12:21:49 INFO mapred.MapTask: kvstart = 26214396; length = 6553600
15/11/13 12:21:49 INFO mapred.MapTask: Map output collector class = org.apache.hadoop.mapred.MapTask$MapOutputBuffer
15/11/13 12:21:49 INFO mapreduce.Job: Job job_local1636203858_0001 running in uber mode : false
15/11/13 12:21:49 INFO mapreduce.Job:  map 0% reduce 0%
15/11/13 12:21:49 INFO mapred.LocalJobRunner:
15/11/13 12:21:50 INFO mapred.MapTask: Starting flush of map output
15/11/13 12:21:50 INFO mapred.MapTask: Spilling map output
15/11/13 12:21:50 INFO mapred.MapTask: bufstart = 0; bufend = 69267; bufvoid = 104857600
15/11/13 12:21:50 INFO mapred.MapTask: kvstart = 26214396(104857584); kvend = 26189212(104756848); length = 25185/6553600
15/11/13 12:21:50 INFO mapred.MapTask: Finished spill 0
15/11/13 12:21:50 INFO mapred.Task: Task:attempt_local1636203858_0001_m_000000_0 is done. And is in the process of committing
15/11/13 12:21:50 INFO mapred.LocalJobRunner: map
15/11/13 12:21:50 INFO mapred.Task: Task 'attempt_local1636203858_0001_m_000000_0' done.
15/11/13 12:21:50 INFO mapred.LocalJobRunner: Finishing task: attempt_local1636203858_0001_m_000000_0
15/11/13 12:21:50 INFO mapred.LocalJobRunner: map task executor complete.
15/11/13 12:21:50 INFO mapred.LocalJobRunner: Waiting for reduce tasks
15/11/13 12:21:50 INFO mapred.LocalJobRunner: Starting task: attempt_local1636203858_0001_r_000000_0
15/11/13 12:21:50 INFO util.ProcfsBasedProcessTree: ProcfsBasedProcessTree currently is supported only on Linux.
15/11/13 12:21:50 INFO mapred.Task:  Using ResourceCalculatorProcessTree : null
15/11/13 12:21:50 INFO mapred.ReduceTask: Using ShuffleConsumerPlugin: org.apache.hadoop.mapreduce.task.reduce.Shuffle@3e68cd79
15/11/13 12:21:50 INFO reduce.MergeManagerImpl: MergerManager: memoryLimit=372781856, maxSingleShuffleLimit=93195464, mergeThreshold=246036032, ioSortFactor=10, memToMemMergeOutputsThreshold=10
15/11/13 12:21:50 INFO reduce.EventFetcher: attempt_local1636203858_0001_r_000000_0 Thread started: EventFetcher for fetching Map Completion Events
15/11/13 12:21:50 INFO reduce.LocalFetcher: localfetcher#1 about to shuffle output of map attempt_local1636203858_0001_m_000000_0 decomp: 81863 len: 81867 to MEMORY
15/11/13 12:21:50 INFO reduce.InMemoryMapOutput: Read 81863 bytes from map-output for attempt_local1636203858_0001_m_000000_0
15/11/13 12:21:50 INFO reduce.MergeManagerImpl: closeInMemoryFile -> map-output of size: 81863, inMemoryMapOutputs.size() -> 1, commitMemory -> 0, usedMemory ->81863
15/11/13 12:21:50 INFO reduce.EventFetcher: EventFetcher is interrupted.. Returning
15/11/13 12:21:50 INFO mapred.LocalJobRunner: 1 / 1 copied.
15/11/13 12:21:50 INFO reduce.MergeManagerImpl: finalMerge called with 1 in-memory map-outputs and 0 on-disk map-outputs
15/11/13 12:21:50 INFO mapred.Merger: Merging 1 sorted segments
15/11/13 12:21:50 INFO mapred.Merger: Down to the last merge-pass, with 1 segments left of total size: 81856 bytes
15/11/13 12:21:50 INFO reduce.MergeManagerImpl: Merged 1 segments, 81863 bytes to disk to satisfy reduce memory limit
15/11/13 12:21:50 INFO reduce.MergeManagerImpl: Merging 1 files, 81867 bytes from disk
15/11/13 12:21:50 INFO reduce.MergeManagerImpl: Merging 0 segments, 0 bytes from memory into reduce
15/11/13 12:21:50 INFO mapred.Merger: Merging 1 sorted segments
15/11/13 12:21:50 INFO mapred.Merger: Down to the last merge-pass, with 1 segments left of total size: 81856 bytes
15/11/13 12:21:50 INFO mapred.LocalJobRunner: 1 / 1 copied.
15/11/13 12:21:50 INFO Configuration.deprecation: mapred.skip.on is deprecated. Instead, use mapreduce.job.skiprecords
15/11/13 12:21:50 INFO mapreduce.Job:  map 100% reduce 0%
15/11/13 12:21:51 INFO mapred.Task: Task:attempt_local1636203858_0001_r_000000_0 is done. And is in the process of committing
15/11/13 12:21:51 INFO mapred.LocalJobRunner: 1 / 1 copied.
15/11/13 12:21:51 INFO mapred.Task: Task attempt_local1636203858_0001_r_000000_0 is allowed to commit now
15/11/13 12:21:51 INFO output.FileOutputCommitter: Saved output of task 'attempt_local1636203858_0001_r_000000_0' to file:/Users/msantos/System/HADOOP/hadoop-2.6.2/test/output_hcn/_temporary/0/task_local1636203858_0001_r_000000
15/11/13 12:21:51 INFO mapred.LocalJobRunner: reduce > reduce
15/11/13 12:21:51 INFO mapred.Task: Task 'attempt_local1636203858_0001_r_000000_0' done.
15/11/13 12:21:51 INFO mapred.LocalJobRunner: Finishing task: attempt_local1636203858_0001_r_000000_0
15/11/13 12:21:51 INFO mapred.LocalJobRunner: reduce task executor complete.
15/11/13 12:21:51 INFO mapreduce.Job:  map 100% reduce 100%
15/11/13 12:21:51 INFO mapreduce.Job: Job job_local1636203858_0001 completed successfully
15/11/13 12:21:51 INFO mapreduce.Job: Counters: 30
	File System Counters
		FILE: Number of bytes read=3564576
		FILE: Number of bytes written=824090
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
	Map-Reduce Framework
		Map input records=6297
		Map output records=6297
		Map output bytes=69267
		Map output materialized bytes=81867
		Input split bytes=158
		Combine input records=0
		Combine output records=0
		Reduce input groups=176
		Reduce shuffle bytes=81867
		Reduce input records=6297
		Reduce output records=6297
		Spilled Records=12594
		Shuffled Maps =1
		Failed Shuffles=0
		Merged Map outputs=1
		GC time elapsed (ms)=13
		Total committed heap usage (bytes)=395960320
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters
		Bytes Read=1700190
	File Output Format Counters
		Bytes Written=69819
msantos@MBP-2[12:21]:~/System/HADOOP/current/test$l output_hcn/
total 136
-rw-r--r--  1 msantos  staff      0 13 Nov 12:21 _SUCCESS
-rw-r--r--  1 msantos  staff  69267 13 Nov 12:21 part-r-00000
```
Now there is an output! "Reduce output records=6297" and so it shows in the output directory where part-r-00000 is no longer empty.

POSTPONED

### Hadoop Streaming I : Python Mapper & Reducer
Let's check the values of TMAX for 1926 as measured by station NCDC/ghcnd_hcn/USC00011084.dly
```
$less ghcnd_hcn/USC00011084.dly|grep "1926.*TMAX" | cut -c 1-26| less
USC00011084192601TMAX-9999
USC00011084192602TMAX  194
USC00011084192603TMAX  217
USC00011084192604TMAX  222
USC00011084192605TMAX  306
USC00011084192606TMAX  317
USC00011084192607TMAX  367
USC00011084192608TMAX  322
USC00011084192609TMAX  356
USC00011084192610TMAX  367
USC00011084192611TMAX  222
USC00011084192612TMAX  222
```
and we see the max temp is 36.7C
With python:
```
$cat ../input/data/NCDC/ghcnd_hcn/USC00011084.dly | ../classes/max_temperature_map.py | ../classes/max_temperature_reduce.py | less
1926    367
1927    367
1928    367
1930    361
1931    367
1932    339
1933    367
1934    350
1935    350
1936    339
1937    383
1938    344
1939    339
1940    350
1941    328
1942    333
1943    356
1944    350
1945    356
```
Here the mapper and reducer are:
The mapper
```python
#!/usr/bin/env python

import re
import sys

for line in sys.stdin:
        val = line.strip()
        element = val[17:21]
        #print "element:>%s<" % element
        if( element == "TMAX"):
                (year, temp, q) = (val[11:15],val[21:26],val[27:28])
                if temp != "-9999" and  q == " ":
                        print "%s\t%s" % (year,temp)
```
and reducer
```python
#!/usr/bin/env python


import sys

(last_key,max_val) = (None, 0)

for line in sys.stdin:
        (key,val) = line.strip().split("\t")
        if ( last_key and last_key != key ):
                print "%s\t%s" % (last_key,max_val)
                (last_key,max_val) = (key,int(val))
        else:
                (last_key,max_val) = (key,max(max_val,int(val)))
if last_key:
        print "%s\t%s" % (last_key,max_val)
```
now Using hadoop streaming with a pythong script:
```
OUTDIR=output-py hadoop jar $HADOOP_INSTALL/share/hadoop/tools/lib/hadoop-streaming-*.jar \
        -input input/data/NCDC/ghcnd_hcn/USC00011084.dly \
        -output $OUTDIR \
        -mapper classes/max_temperature_map.py \
        -reducer classes/max_temperature_reduce.py \
        >& log-py
```
we get
```
msantos@MBP-2[15:20]:~/System/HADOOP/current/test$head output-py/part-00000
1926	367
1927	367
1928	367
1930	361
1931	367
1932	339
1933	367
1934	350
1935	350
1936	339
```
Python stream on ALL station files (same command as before with input the whole directory input/data/NCDC/ghcnd_hcn):
```
msantos@MBP-2[15:33]:~/System/HADOOP/current/test$time ./run-MaxTemperature-py
		Reduce output records=145

real	12m25.303s
user	8m36.415s
sys	4m41.120s
```
By not checking the quality of the measurement I included a value like this one for 1967
```
$cat ghcnd_hcn/USC00244364.dly |grep USC00244364196701TMAX
USC00244364196701TMAX 1544 X0-9999   -9999   -9999   -9999   -9999   -9999   -9999   -9999   -9999   -9999   -9999   -9999   -9999   -9999   -9999   -9999   -9999   -9999   -9999   -9999   -9999   -9999   -9999   -9999   -9999   -9999   -9999   -9999   -9999   -9999
```
That is, a max temp of 154.4C !! Obviously this is a failed measurement and the quality indicator correspondingly shows a non-blank character namely 'X'.
Filtering for quality we get the final result set.
```
$time ./run-MaxTemperature-py
		Reduce output records=145

real	11m44.270s
user	8m34.876s
sys	4m45.707s
```
### Hadoop Streaming II : Python single script & /usr/bin/cat as trivial reducer
Streaming single python map reading all files and calculating the max temp for each year
The python map:
```python
#!/usr/bin/env python

import re
import sys

for line in sys.stdin:
        val = line.strip()
        element = val[17:21]
        #print "element:>%s<" % element
        max = {}
        if( element == "TMAX"):
                (year, temp, q) = (val[11:15],val[21:26],val[27:28])
                if temp != "-9999" and  q == " ":
                        #print "%s\t%s" % (year,temp)
                        if not max.has_key(year) or temp>max[year]:
                                max[year] = temp
        for yr in max.keys():
                print "%s\t%s" % (yr,max[yr])
```
The hadoop command \& execution
```
cat ./run-MaxTemperature-py-single
#!/bin/bash
# all/single-file mode: -input input/data/NCDC/ghcnd_hcn \ #/USC00011084.dly \
OUTDIR=output-py-all-single
[ -d $OUTDIR ] && rm -Rf $OUTDIR
hadoop jar $HADOOP_INSTALL/share/hadoop/tools/lib/hadoop-streaming-*.jar \
        -input input/data/NCDC/ghcnd_hcn \
        -output $OUTDIR \
        -mapper classes/max_temperature_single.py \
        -reducer /bin/cat \
        >& log-py-all-single
grep "Reduce output" log-py-all-single
$time ./run-MaxTemperature-py-single

real	22m22.339s
user	22m2.756s
sys	0m39.069s
```
Made the mistake of not testing the script first directly on a few sample files. After 22m it turns out it just loaded everything into memory (+600MBi)
and crash with error
```
Traceback (most recent call last):
  File "/Users/msantos/System/HADOOP/hadoop-2.6.2/test/./classes/max_temperature_single.py", line 15, in <module>
    if temp>max[year]:
KeyError: '1889'
```
Once corrected we get
```
msantos@MBP-2[18:55]:~/System/HADOOP/current/test$time ./run-MaxTemperature-py-single
		Reduce output records=1413236

real	11m57.894s
user	7m58.080s
sys	4m52.835s
```
However, directly from the command line we get
```
msantos@MBP-2[19:25]:~/System/HADOOP/current/test$time cat input/data/NCDC/ghcnd_hcn/US*.dly | classes/max_temperature_single.py >& o-single

real	0m30.382s
user	0m25.770s
sys	0m3.815s
```
##  Conclusions

Basically, the python map/reduce and single step process both under Hadoop take the same time in processing all 1218 station files, 
namely around 12min.

If we run the python directly from the command line on all station files it takes, however, way less: ~30s. 
This is more in sync with the awk script we tried at the beginning. 

Hadoop incurrs in an overhead that doesn't pay off undless we use many processors. We need to confirm this explicitly.

## Why Should I Care?
