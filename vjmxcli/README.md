
# 1. 概述

在cmdline-jmxclient项目上定制，增加功能

* 支持以pid接入JVM，不需要原JVM在启动参数中打开了JMX选项
* 完全模拟`jstat -gcutil`输出的`gcutil`，用于jstat不能使用的情况。

因为每调度一次`java -jar vjmxclient.jar`，其实是创建了一个新的JVM，因此在vjmxcli.sh 加上了一系列JVM参数减少消耗。


## 2. 获取MBean属性值

```
// 以host:port接入
./vjmxcli.sh - 127.0.0.1:8060 java.lang:type=Memory HeapMemoryUsage

// 以pid接入
./vjmxcli.sh - 98583 java.lang:type=Memory HeapMemoryUsage
```

参数解释

* `-` : 无密码
* `127.0.0.1:8060` or `98582` : 应用地址及jmx端口, 或pid
* `java.lang:type=Memory`:MBean名
* `HeapMemoryUsage`:Attribute名


#3. 模拟jstat gcutil

jstat有时候会不可使用，比如目标JVM使用-Djava.tmp.dir 设置了临时目录，或者使用了-XX:+PerfDisableSharedMem禁止了perfdata，此时，可以用vjmxcli代替。

```
//一次性输出
./vjmxcli.sh - 127.0.0.1:7001 gcutil

//间隔5秒连续输出
./vjmxcli.sh - 127.0.0.1:7001 gcutil 5

```
JDK7 示例输出

```
S	      S	      E	     O	     P	   	YGC	YGCT	FGC	FGCT	GCT	
41.25	41.25	2.25	0.00	0.48	2	0.025	0	0.0	   0.025
```

JDK8 示例输出
```
S	      S	      E	     O	      M	   CCS	YGC	YGCT	FGC	FGCT	GCT	
41.25	41.25	2.25	0.00	0.48	0	2	0.025	0	0.0	   0.025
```


# 4. 附录 

## 4.1 常用JMX项

| 项目 | Object Name | Attribute Name|
| -------- | -------- | -------- |
| 堆内存    |  java.lang:type=Memory | HeapMemoryUsage     |
| 非堆内存(不包含堆外内存)    |  java.lang:type=Memory | NonHeapMemoryUsage     |
| 堆外内存(不包含新版Netty申请的堆外内存)    |  java.nio:type=BufferPool,name=direct |MemoryUsed |
| 线程数    |  java.lang:type=Threading | ThreadCount |
| 守护线程数    |  java.lang:type=Threading | DaemonThreadCount |
| 分代内存及GC  |  不同JDK的值不一样 | 不同JDK的值不一样  |

## 4.2 启用JMX的启动参数

```
-Dcom.sun.management.jmxremote
-Dcom.sun.management.jmxremote.port=7001  
-Dcom.sun.management.jmxremote.authenticate=false 
-Dcom.sun.management.jmxremote.ssl=false
-Djava.rmi.server.hostname=127.0.0.1
```
