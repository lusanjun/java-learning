## JVM性能调优



### 1. JVM工具

#### 1.1 JDK工具

##### 1.1.1  jps

jps（JVM Process Status Tool），虚拟机进程状况工具，主要用来输出JVM中运行的进程状态信息。

```shell
jps -q：只输出进程ID
jps -m：输出传递给Java进程的参数
jps -l：输出主类的全名，如果进程执行的是Jar包，则输出Jar路径
jps -v：输出虚拟机进程启动时的JVM参数
```

##### 1.1.2  jstat

jstat（JVM Statistics Moitoring Tool），虚拟机统计信息监视工具， 用于监视虚拟机各种运行状态信息。

```shell
jstat -class：类加载、卸载数量、总空间以及类装载所耗费的时间
jstat -gc：Java堆状况
jstat -gcnew：新生代垃圾收集状况
jstat -gcold：老年代垃圾收集状况
jstat -pcpermcapacity：永久代使用到的最大、最小空间
jstat -compiler：输出即时编译器编译过的方法、耗时等信息
jstat -printcompilation：输出已经被即时编译的方法
```

##### 1.1.3  jmap

jmap（Memory Map for Java），Java内存映像工具， 用于生成堆转储快照。

```shell
jmap -dump：生成Java堆转储快照
jmap -finalizerinfo：显示在F-Queue中等待Finalizer线程执行finalize方法的对象
jmap -heap：显示Java堆详细信息
jmap -histo：显示堆中对象统计信息，包括类、实例数量、合计容量
jmap -permastat：以ClassLoader为统计口径显示永久代内存状态
jmap -F：强制生成dump快照
```

##### 1.1.4  jstack

jstack（Stack Trace for Java），Java堆栈跟踪工具， 用于生成虚拟机当前时刻的线程快照。

```shell
jstack -F：强制输出线程堆栈
jstack -l：显示关于锁的附加信息
jstack -m：可以显示C/C++的堆栈
```

### 2. 调优实践

#### 2.1 什么时候JVM调优

- 系统吞吐量下降，或P90、P99响应增加
- 堆内存持续上涨，到出现OOM
- 频繁FullGC、GC停顿过长
- 堆内存占用过高

#### 2.2 调优原则

- 优先原则：产品、架构、代码、数据库优先，JVM是最后的手段
- 观测性原则：发现问题解决问题，没有问题不创造问题

#### 2.3 调优步骤

1. 监控JVM分析问题：评估重要性，内存使用、GC频率、GC耗时
2. 确定目标：内存占用、响应延迟
3. 制定方案：配置内存及GC相关参数
4. 验证方案：测试环境对比方案前后差异，确定是否生效
5. 结果验收：灰度测试，全量发布

### 3. 调优案例

#### 3.1 CPU过高

1. 使用`top`命令查看CPU占用情况，确定哪个进程cpu过高，例如 4012
2. 使用命令查看线程占用cpu，`ps H -eo pid,tid,%cpu | grep 4012`，如 4022
3. 使用命令查看进程堆栈情况，`jstack 4012`
4. 确定线程十六进制号，`printf "%x\n" 4022`  ，结果 FB6
5. 找到十六进制线程号日志，看到某一行代码造成cpu过高。

