---
layout: post
title: "Hadoop with Python 遇到的问题"
description: "Hadoop with Python 遇到的问题"
category: Hadoop
tags: [Hadoop,Python]
---


昨天使用了 Python 折腾　Hadoop ，遇到一个神经的问题，纠结了一整天

使用 

    hadoop jar share/hadoop/tools/lib/hadoop-streaming-*.jar -mapper map.py -reducer reduce.py -input /data/*.txt -output /output

命令时，一直提示我如下错误

    java.io.IOException: Cannot run program "/home/liuzheng/map.py": error=2, No such file or directory
            at java.lang.ProcessBuilder.start(ProcessBuilder.java:1047)
            at org.apache.hadoop.streaming.PipeMapRed.configure(PipeMapRed.java:209)
            at org.apache.hadoop.streaming.PipeMapper.configure(PipeMapper.java:66)
            at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
            at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
            at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
            at java.lang.reflect.Method.invoke(Method.java:606)
            at org.apache.hadoop.util.ReflectionUtils.setJobConf(ReflectionUtils.java:106)
            at org.apache.hadoop.util.ReflectionUtils.setConf(ReflectionUtils.java:75)
            at org.apache.hadoop.util.ReflectionUtils.newInstance(ReflectionUtils.java:133)
            at org.apache.hadoop.mapred.MapRunner.configure(MapRunner.java:38)
            at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
            at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
            at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
            at java.lang.reflect.Method.invoke(Method.java:606)
            at org.apache.hadoop.util.ReflectionUtils.setJobConf(ReflectionUtils.java:106)
            at org.apache.hadoop.util.ReflectionUtils.setConf(ReflectionUtils.java:75)
            at org.apache.hadoop.util.ReflectionUtils.newInstance(ReflectionUtils.java:133)
            at org.apache.hadoop.mapred.MapTask.runOldMapper(MapTask.java:426)
            at org.apache.hadoop.mapred.MapTask.run(MapTask.java:342)
            at org.apache.hadoop.mapred.LocalJobRunner$Job$MapTaskRunnable.run(LocalJobRunner.java:243)
            at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:471)
            at java.util.concurrent.FutureTask.run(FutureTask.java:262)
            at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
            at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
            at java.lang.Thread.run(Thread.java:745)
    Caused by: java.io.IOException: error=2, No such file or directory
            at java.lang.UNIXProcess.forkAndExec(Native Method)
            at java.lang.UNIXProcess.<init>(UNIXProcess.java:186)
            at java.lang.ProcessImpl.start(ProcessImpl.java:130)
            at java.lang.ProcessBuilder.start(ProcessBuilder.java:1028)
            ... 25 more
    14/10/27 15:35:52 INFO mapred.LocalJobRunner: map task executor complete.
    14/10/27 15:35:52 WARN mapred.LocalJobRunner: job_local1435167124_0001
    java.lang.Exception: java.lang.RuntimeException: Error in configuring object
            at org.apache.hadoop.mapred.LocalJobRunner$Job.runTasks(LocalJobRunner.java:462)
            at org.apache.hadoop.mapred.LocalJobRunner$Job.run(LocalJobRunner.java:522)
    Caused by: java.lang.RuntimeException: Error in configuring object
            at org.apache.hadoop.util.ReflectionUtils.setJobConf(ReflectionUtils.java:109)
            at org.apache.hadoop.util.ReflectionUtils.setConf(ReflectionUtils.java:75)
            at org.apache.hadoop.util.ReflectionUtils.newInstance(ReflectionUtils.java:133)
            at org.apache.hadoop.mapred.MapTask.runOldMapper(MapTask.java:426)
            at org.apache.hadoop.mapred.MapTask.run(MapTask.java:342)
            at org.apache.hadoop.mapred.LocalJobRunner$Job$MapTaskRunnable.run(LocalJobRunner.java:243)
            at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:471)
            at java.util.concurrent.FutureTask.run(FutureTask.java:262)
            at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
            at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
            at java.lang.Thread.run(Thread.java:745)
    Caused by: java.lang.reflect.InvocationTargetException
            at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
            at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
            at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
            at java.lang.reflect.Method.invoke(Method.java:606)
            at org.apache.hadoop.util.ReflectionUtils.setJobConf(ReflectionUtils.java:106)
            ... 10 more
    Caused by: java.lang.RuntimeException: Error in configuring object
            at org.apache.hadoop.util.ReflectionUtils.setJobConf(ReflectionUtils.java:109)
            at org.apache.hadoop.util.ReflectionUtils.setConf(ReflectionUtils.java:75)
            at org.apache.hadoop.util.ReflectionUtils.newInstance(ReflectionUtils.java:133)
            at org.apache.hadoop.mapred.MapRunner.configure(MapRunner.java:38)
            ... 15 more
    Caused by: java.lang.reflect.InvocationTargetException
            at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
            at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
            at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
            at java.lang.reflect.Method.invoke(Method.java:606)
            at org.apache.hadoop.util.ReflectionUtils.setJobConf(ReflectionUtils.java:106)
            ... 18 more
    Caused by: java.lang.RuntimeException: configuration exception
            at org.apache.hadoop.streaming.PipeMapRed.configure(PipeMapRed.java:222)
            at org.apache.hadoop.streaming.PipeMapper.configure(PipeMapper.java:66)
            ... 23 more
    Caused by: java.io.IOException: Cannot run program "/home/liuzheng/map.py": error=2, No such file or directory
            at java.lang.ProcessBuilder.start(ProcessBuilder.java:1047)
            at org.apache.hadoop.streaming.PipeMapRed.configure(PipeMapRed.java:209)
            ... 24 more
    Caused by: java.io.IOException: error=2, No such file or directory
            at java.lang.UNIXProcess.forkAndExec(Native Method)
            at java.lang.UNIXProcess.<init>(UNIXProcess.java:186)
            at java.lang.ProcessImpl.start(ProcessImpl.java:130)
            at java.lang.ProcessBuilder.start(ProcessBuilder.java:1028)
            ... 25 more

令人十分费解，Google若干也没找到（主要忽略了没有 *star* 的回答）

今早仔细研读各种bug问题，发现在下方若干回复都有曾提及dos2unix之类的行尾转换符的问题。

尝试打开自己的程序，将其运行该命令，再跑之。。。。成功！

仔细分析转换前后文件，发现行尾回车 `\r\n` 被转换成了 `\n`。好吧，下面我就不多说了。。。 
