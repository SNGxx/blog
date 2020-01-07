# Java垃圾收集

[TOC]



## JVM选项

下文使用的JDK version信息：

```powershell
C:\Users\Administrator>java -version
java version "1.8.0_60"
Java(TM) SE Runtime Environment (build 1.8.0_60-b27)
Java HotSpot(TM) 64-Bit Server VM (build 25.60-b23, mixed mode)
```

Jdk8查看各选项含义：[官方文档][https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html]。



### 标准选项（Standard Options）

所有JVM都支持的标准选项，且向后兼容。

使用`java -?`或`java -help`来查看JVM标准选项列表：

```powershell
C:\Users\Administrator>java -?
用法: java [-options] class [args...]
           (执行类)
   或  java [-options] -jar jarfile [args...]
           (执行 jar 文件)
其中选项包括:
    -d32          使用 32 位数据模型 (如果可用)
    -d64          使用 64 位数据模型 (如果可用)
    -server       选择 "server" VM
                  默认 VM 是 server.

    -cp <目录和 zip/jar 文件的类搜索路径>
    -classpath <目录和 zip/jar 文件的类搜索路径>
                  用 ; 分隔的目录, JAR 档案
                  和 ZIP 档案列表, 用于搜索类文件。
    -D<名称>=<值>
                  设置系统属性
    -verbose:[class|gc|jni]
                  启用详细输出
    -version      输出产品版本并退出
    -version:<值>
                  警告: 此功能已过时, 将在
                  未来发行版中删除。
                  需要指定的版本才能运行
    -showversion  输出产品版本并继续
    -jre-restrict-search | -no-jre-restrict-search
                  警告: 此功能已过时, 将在
                  未来发行版中删除。
                  在版本搜索中包括/排除用户专用 JRE
    -? -help      输出此帮助消息
    -X            输出非标准选项的帮助
    -ea[:<packagename>...|:<classname>]
    -enableassertions[:<packagename>...|:<classname>]
                  按指定的粒度启用断言
    -da[:<packagename>...|:<classname>]
    -disableassertions[:<packagename>...|:<classname>]
                  禁用具有指定粒度的断言
    -esa | -enablesystemassertions
                  启用系统断言
    -dsa | -disablesystemassertions
                  禁用系统断言
    -agentlib:<libname>[=<选项>]
                  加载本机代理库 <libname>, 例如 -agentlib:hprof
                  另请参阅 -agentlib:jdwp=help 和 -agentlib:hprof=help
    -agentpath:<pathname>[=<选项>]
                  按完整路径名加载本机代理库
    -javaagent:<jarpath>[=<选项>]
                  加载 Java 编程语言代理, 请参阅 java.lang.instrument
    -splash:<imagepath>
                  使用指定的图像显示启动屏幕
有关详细信息, 请参阅 http://www.oracle.com/technetwork/java/javase/documentation/index.html。
```



- -server

  设置jvm使用server模式。64位jdk下默认启用该模式，忽略-client选项。

- -D<名称>=<值>

  设置系统属性，在此JVM之上的应用程序可用System.getProperty("名称")得到对应的值。如果值中有空格，需要使用双引号括起来，如-Dapp.home="C:/Program Files"。

  该选项通常用于设置系统级全局变量值，如配置文件路径，以便该属性在程序中任何地方都可以访问。



### 非标准选项（Non-Standard Options）

HotSpot虚拟机的通用选项，不保证所有JVM都支持，并且可能会更改。非标准选项以-X开头。

使用`java -X`来查看非标准选项：

```powershell
C:\Users\Administrator>java -X
    -Xmixed           混合模式执行 (默认)
    -Xint             仅解释模式执行
    -Xbootclasspath:<用 ; 分隔的目录和 zip/jar 文件>
                      设置搜索路径以引导类和资源
    -Xbootclasspath/a:<用 ; 分隔的目录和 zip/jar 文件>
                      附加在引导类路径末尾
    -Xbootclasspath/p:<用 ; 分隔的目录和 zip/jar 文件>
                      置于引导类路径之前
    -Xdiag            显示附加诊断消息
    -Xnoclassgc       禁用类垃圾收集
    -Xincgc           启用增量垃圾收集
    -Xloggc:<file>    将 GC 状态记录在文件中 (带时间戳)
    -Xbatch           禁用后台编译
    -Xms<size>        设置初始 Java 堆大小
    -Xmx<size>        设置最大 Java 堆大小
    -Xss<size>        设置 Java 线程堆栈大小
    -Xprof            输出 cpu 配置文件数据
    -Xfuture          启用最严格的检查, 预期将来的默认值
    -Xrs              减少 Java/VM 对操作系统信号的使用 (请参阅文档)
    -Xcheck:jni       对 JNI 函数执行其他检查
    -Xshare:off       不尝试使用共享类数据
    -Xshare:auto      在可能的情况下使用共享类数据 (默认)
    -Xshare:on        要求使用共享类数据, 否则将失败。
    -XshowSettings    显示所有设置并继续
    -XshowSettings:all
                      显示所有设置并继续
    -XshowSettings:vm 显示所有与 vm 相关的设置并继续
    -XshowSettings:properties
                      显示所有属性设置并继续
    -XshowSettings:locale
                      显示所有与区域设置相关的设置并继续

-X 选项是非标准选项, 如有更改, 恕不另行通知。
```



- -Xms 设置初始堆大小
- -Xmx 设置最大堆大小
- -Xloggc:\<file\> 指定gc日志文件
- -Xint 解释执行
- -Xcomp 第一次使用就编译成本地代码
- -Xmixed 混合模式、JVM自己来决定是否编译成本地代码



### 高级选项（Advanced Options）

高级选项属于开发者用来调整HotSpot虚拟机特定行为的选项。和非标准选项一样，它不保证其它JVM的支持，并且很有可能更改。高级选项以-XX开头。

布尔类型用于启用默认关闭的功能或禁用默认开启的功能，此类型不需要参数，使用加号或减号表示启用或禁用。例如：`-XX:+UseParallelGC`。

对于需要参数的选项，参数可以使用空格、冒号或等号与选项名称分隔。例如：`-XX:MetaspaceSize=64M`。**size类选项不写单位时，单位为字节，支持的单位有k、m、g，大小写均可。**

高级选项分为高级运行时选项、高级JIT编译器选项、高级服务能力选项、高级垃圾回收选项。

- 高级运行时选项（Advanced Runtime Options）

  该类选项控制虚拟机在运行时的行为。如：

  - -XX:ErrorFile=\<fileName\> 当发生不可恢复的错误时，错误信息及崩溃时刻jvm的相关信息记录到哪个文件。默认是当前目录下hs_err_pid.log文件。

- 高级JIT编译器选项（Advanced JIT Compiler Options）

  该类选项主要在动态just in time编译时用到。如：

  - -XX:+BackgroundCompilation 后台编译，默认开启。

- 高级服务能力选项（Advanced ServiceAbility Options）

  该类选项可以做系统信息收集和扩展性的debug。如：

  - -XX:+HeapDumpOnOutOfMemory 设置当oom时，将heap内存dump到当前目录下的一个文件，默认是关闭的。
  - -XX:HeapDumpPath=\<path\> 设置dump heap时，将文件dump到哪里。例如：-XX:HeapDumpPath=/var/log/java/java_heapdump.hprof。
  - -XX:LogFile=\<path\> 指定日志被记录在哪里，默认时当前目录的hotspot.log下。

- 高级垃圾回收选项（Advanced Garbage Collection Options）

  该类选项控制JVM如何进行垃圾回收。如：

  - -XX:MetaspaceSize=\<size\> 设置元数据区大小，第一次超出该值后会触发GC。默认值依赖于平台。

  - -XX:MaxMetaspaceSize=\<size\> 设置元数据区的上限，默认无限制。
  - -XX:MaxDirectMemorySize=\<size\> 为NIO的direct-buffer分配时指定最大的内存大小。默认是0，意思是JVM自动选择direct-buffer的大小。



## 垃圾收集的区域和对象

Java内存运行时区域中，程序计数器、虚拟机栈、本地方法栈3个区域随线程而生，随线程而灭；栈中的栈帧随着方法的进入和退出而有条不紊地执行着出栈和入栈操作。每一个栈帧中分配多少内存基本上是在类结构确定下来就已知的。因此这几个区域的内存分配和回收都具备确定性，所以不需要多考虑内存回收的问题。

而Java堆和方法区则不一样，一个接口中的多个实现类需要的内存可能不一样，一个方法中的多个分支需要的内存可能不一样，我们只有在程序处于运行时期才知道会创建哪些对象，这部分内存的分配和回收都是动态的，垃圾收集器所关注的是这部分内存。

![jvm-runtime](src\jvm-runtime.jpg)

### 回收堆内存

在堆里面存放着几乎所有的对象实例，回收之前首先要确定对象是否“死亡”，即不可能再被任何途径使用。

判断对象是否“死亡”的方法有两种：一是引用计数法，存在相互引用的问题；二是可达性分析算法。

可达性分析算法的基本思路就是通过一系列的称为“GC Roots“的对象作为起始点，从这些节点开始向下搜索，搜索走过的路径称为引用链，当一个对象到GC Roots不可达时，则证明此对象是不可用的。

在Java中，可以作为GC Roots的对象包括下面几种：

- 虚拟机栈（栈帧中的本地变量表）中引用的对象。
- 方法区中类静态属性引用的对象。
- 方法区中常量引用的对象。
- 本地方法栈中JNI引用的对象。

#### 引用类型

JDK 1.2之后，Java对引用的概念进行了扩充，将引用分为强引用（Strong Reference）、软引用（Soft Reference）、弱引用（Weak Reference）、虚引用（Phantom Reference）。

- 强引用：在程序代码之中不变存在的，类似Object object=new Object()这类引用，只要强引用还在，就不会回收被引用的对象。
- 软引用：用来描述一些有用但非必须的对象。对于软引用关联着的对象，在系统将要发生内存溢出之前，将会把这些对象列入回收范围之中进行第二次回收。使用SoftReference类来实现软引用。
- 弱引用：也是用来描述非必须对象的，但是它的强度比软引用更弱一些，被弱引用关联的对象只能够生存到下一次垃圾回收发生之前。当垃圾回收器工作时，无论内存是否足够，都会回收掉只被弱引用关联的对象。使用WeakReference来实现弱引用。
- 虚引用：它是最弱的一种引用关系。一个对象是否有虚引用，完全不会对其生存时间构成影响，也无法通过虚引用来取得一个对象实例。为一个对象设置虚引用的唯一目的就是能在这个对象被收集器回收时收到一个系统通知。



### 回收方法区

方法区中主要回收两部分：废弃常量和无用的类。

回收废弃常量与回收堆中的对象非常类似。

判定一个类是无用的类要满足以下三个条件：

- 该类所有的实例都已经被回收，也就是Java堆中不存在该类的任何实例。
- 加载该类的ClassLoader已经被回收。
- 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。



## 垃圾收集算法

### 标记-清除（Mark-Sweep）算法

分为标记和清除两个阶段：首先标记出所有需要回收的对象，在标记完成后同一回收所有被标记的对象。

### 复制（Copying）算法

将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另一块上面，然后再把已使用过的内存空间一次性清理掉。

由于新生代中大部分对象都是朝生夕死的，所以不需要按照1:1来划分内存空间。HotSpot将内存分为一块较大的Eden空间和两块较小的Survivor空间（比例为8:1），每次使用Eden和其中一块Survivor。当回收时，将Eden（80%）和Survivor（10%）中还存活的对象一次性地复制到另外一块Survivor（10%）上，最后清理掉Eden和刚才用过的Survivor空间。由于不能保证每次回收都只有10%的对象存活，因此还需要依赖其它内存（老年代）进行分配担保。

### 标记-整理（Mark-Compact）算法

首先标记出所有需要回收的对象，标记完成后让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存。

### 分代收集（Generational Collection）算法

根据对象存活周期的不同将内存划分为几块，一般是分为新生代和老年代，在不同的年代采用适合的收集算法。

在新生代中，每次垃圾收集时都有大量的对象死去，只有少量存活，选用复制算法。

老年代中对象存活率高、没有额外的空间对它进行分配担保，就必须使用”标记-清除“或”标记-整理“算法来进行回收。

根据当前商业虚拟机都采用分代收集算法。



## 垃圾收集器

当GC只发生在新生代中时称为Minor GC；当GC发生在老年代时，称为Major GC，经常会伴随着至少一次的Minor GC。有个说法是Major GC就是Full GC，也有说法是Full GC = Minor GC + Major GC。这里采取后面这种说法，听起来靠谱点。

> G1收集器在JAVA8甚至JAVA9中都是不建议使用的，这里不对G1收集器进行介绍。

![gc-collector](src\gc-collector.jpg)

### 新生代垃圾收集器

所有新生代垃圾收集器，都使用复制算法，都会发生stop-the-world。

#### Serial收集器

最简单的单线程垃圾收集器，在进行垃圾收集时，必须暂停其它所有的工作线程，直到它收集结束。

下图为Serial配合Serial Old运行示意图。

![serial-collector](src\serial-collector.jpg)

#### ParNew收集器

ParNew收集器其实是Serial收集器的多线程版本，与Serial不同的地方就是在垃圾收集过程中使用多个线程。

下图为ParNew配合Serial Old运行示意图。

![parnew-collector](src\parnew-collector.jpg)

#### Parallel Scavenge收集器

Parallel Scavenge收集器是一个吞吐量优先的收集器。高吞吐量可以高效率地利用CPU时间，尽快完成程序的运算任务。

在JDK 1.6之前，如果新生代选择了Parallel Scavenge收集器，老年代除了PS MarkSweep（与Serial Old非常接近）之外，别无选择。

### 老年代垃圾收集器

#### Serial Old收集器

Serial Old是Serial的老年版本，它也是一个单线程收集器，使用标记-整理算法。

#### Parallel Old收集器

Parallel Old收集器是Parallel Scavenge的老年版本，它使用多线程和标记-整理算法。

JDK 1.6中提供了Parallel Old收集器，使得Parallel Scavenge可以配合Parallel Old作为吞吐量优先的收集器组合了。

下图为Parallel Scavenge配合Parallel Old运行示意图。

![parallel-old-collector](src\parallel-old-collector.jpg)

#### CMS收集器

CMS（Concurrent Mark Sweep）收集器是基于标记-清除算法实现的，它的目标是尽可能降低STW时间。

CMS运行过程分为4个步骤：

- 初始标记（CMS initial mark）：仅标记直接与GC Roots关联的对象，速度很快。
- 并发标记（CMS concurrent mark）：进行GC Roots Tracing的过程。
- 重新标记（CMS remark）：修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，停顿时间比初始标记稍长，远比并发标记时间短。
- 并发清除（CMS concurrent sweep）：清除被标记的对象。

其中，初始标记和重新标记这两个步骤仍然需要STW。

下图为CMS收集器的运行示意图。

![cms-collector](src\cms-collector.jpg)

### 垃圾收集器组合

目前，在设定JVM选项的时候，由于不是所有的年轻代收集器和老年代收集器都能配合工作，所以不能分别指定年轻代和老年代的收集器，而是提供了几套可选的组合，如图：

![gc-collector-pair](src\gc-collector-pair.jpg)

> 注：CMS和Serial Old的虚线连接表示，在CMS运行期间预留的内存无法满足程序需要，就会出现一次”Concurrent Mode Failure“失败，这时虚拟机将启动后备预案：临时启用Serial Old收集器来进行老年代的垃圾收集。

下表是JDK8中除G1收集器外，所有可选的新生代和老年代垃圾收集组合：

| JVM选项                               | 关联选项          | 新生代收集器      | 老年代收集器 | 备注                           |
| ------------------------------------- | ----------------- | ----------------- | ------------ | ------------------------------ |
| +UseSerialGC                          | /                 | Serial            | Serial Old   | 不推荐                         |
| +UseParallelGC                        | +UseParallelOldGC | Parallel Scavenge | Parallel Old | JDK 1.8默认值                  |
| **+UseParallelOldGC**                 | +UseParallelGC    | Parallel Scavenge | Parallel Old | 推荐，吞吐量优先               |
| +UseParNewGC                          | /                 | ParNew            | Serial Old   | JDK 1.8已过时，应配合CMS使用。 |
| **+UseConcMarkSweepGC**               | +UseParNewGC      | ParNew            | CMS          | 推荐，响应时间优先             |
| +UseConcMarkSweepGC<br />-UseParNewGC | /                 | Serial            | CMS          | 不推荐                         |



> 1. 查看默认使用的垃圾收集器：
>
>    ```powershell
>    > java -XX:+PrintCommandLineFlags -version
>    -XX:InitialHeapSize=132619072 -XX:MaxHeapSize=2121905152 -XX:+PrintCommandLineFlags -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelGC
>    java version "1.8.0_60"
>    Java(TM) SE Runtime Environment (build 1.8.0_60-b27)
>    Java HotSpot(TM) 64-Bit Server VM (build 25.60-b23, mixed mode)
>    ```
>
> 2. 在jconsole连接JVM进程查看垃圾收集器的时候发现+UseParallelGC-UseParallelOldGC和+UseParallelOldGC时显示的垃圾收集器相同。
>
>    实际上只是那个MXBean的名字叫做PS MarkSweep而已。背后的collector实际不一样，前者在HotSpot内是PS MarkSweep，后者是PS ParallelCompact。
>
>    [参考这里][https://hllvm-group.iteye.com/group/topic/37945]
>
> 3. 垃圾收集器在jconsole里显示（MXBean）名称和常用名称对应表：
>
>    | 显示（MXBean）名称  | 常用名称                   |
>    | ------------------- | -------------------------- |
>    | Copy                | Serial                     |
>    | MarkSweepCompact    | Serial Old                 |
>    | PS Scavenge         | Parallel Scavenge          |
>    | PS MarkSweep        | PS MarkSweep或Parallel Old |
>    | ParNew              | ParNew                     |
>    | ConcurrentMarkSweep | CMS                        |

关于垃圾收集器的选择，参考[文档][https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/collectors.html#sthref28]。



## GC日志

阅读GC日志是处理Java虚拟机内存问题的基础技能，它只是一些人为确定的规则，没有太多技术含量。

例如以下两段使用了默认并行收集器和默认格式的典型GC日志：

```shell
4.315: [GC (Allocation Failure)  524800K->16347K(2010112K), 0.0186193 secs]
5.512: [GC (Allocation Failure)  541147K->26748K(2010112K), 0.0252458 secs]
```

它的格式是：

```shell
虚拟机启动以来经过的秒数: [停顿类型 (触发原因) GC前堆使用容量->GC后堆使用容量(堆区总容量), GC时间]
```



### GC日志选项

| 选项                                  | 含义                           | 备注     |
| ------------------------------------- | ------------------------------ | -------- |
| -XX:+PrintGC                          | 输出GC日志                     | 默认开启 |
| -XX:+PrintGCDetails                   | 输出GC的详细信息               | 默认关闭 |
| -XX:+PrintGCTimeStamps                | 输出GC的时间戳（基准时间形式） | 默认开启 |
| -XX:+PrintGCDateStamps                | 输出GC的日期                   | 默认关闭 |
| -XX:+PrintHeapAtGC                    | 在进行GC的前后打印出堆的信息   | 默认关闭 |
| -XX:+PrintGCApplicationStoppedTime    | 输出GC造成的应用暂停时间       | 默认关闭 |
| -Xloggc:/base_dir/logs/managed_gc.log | 指定日志文件路径名称           | /        |



### Parallel GC日志

JVM选项：`-XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps`

```shell
Java HotSpot(TM) 64-Bit Server VM (25.60-b23) for windows-amd64 JRE (1.8.0_60-b27), built on Aug  4 2015 11:06:27 by "java_re" with MS VC++ 10.0 (VS2010)
Memory: 4k page, physical 8288692k(1869224k free), swap 19298740k(4812852k free)
CommandLine flags: -XX:InitialHeapSize=2097152 -XX:MaxHeapSize=2097152 -XX:+PrintGC -XX:+PrintGCDateStamps -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelGC 
2020-01-07T10:40:02.399+0800: 0.085: [GC (Allocation Failure) [PSYoungGen: 512K->488K(1024K)] 512K->496K(1536K), 0.0008800 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
2020-01-07T10:40:02.462+0800: 0.148: [GC (Allocation Failure) [PSYoungGen: 999K->488K(1024K)] 1007K->568K(1536K), 0.0018225 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
2020-01-07T10:40:02.492+0800: 0.179: [GC (Allocation Failure) [PSYoungGen: 1000K->488K(1024K)] 1080K->668K(1536K), 0.0009332 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
2020-01-07T10:40:03.137+0800: 0.823: [GC (Allocation Failure) [PSYoungGen: 1000K->504K(1024K)] 1180K->900K(1536K), 0.0010037 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
2020-01-07T10:40:03.138+0800: 0.824: [Full GC (Ergonomics) [PSYoungGen: 504K->491K(1024K)] [ParOldGen: 396K->346K(512K)] 900K->838K(1536K), [Metaspace: 3423K->3423K(1056768K)], 0.0052635 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
2020-01-07T10:40:03.163+0800: 0.850: [Full GC (Ergonomics) [PSYoungGen: 1003K->668K(1024K)] [ParOldGen: 346K->337K(512K)] 1350K->1005K(1536K), [Metaspace: 3781K->3781K(1056768K)], 0.0070294 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
```

1. PSYoungGen，即年轻代使用了Parallel Scavenge收集器，后面是年轻代收集情况；ParOldGen即老年代使用了Parallel Old收集器，后面是老年代收集情况；Metaspace后面是元数据区收集情况。

2. [Times: user=0.00 sys=0.00, real=0.01 secs]：这里user，sys和real与linux的time命令所输出的时间含义一致，分别代表用户态消耗的cpu时间、内核态消耗的cpu时间和操作从开始到结束经历的墙钟时间（Wall Clock Time），墙钟时间包括了非运算时间，例如IO等待、线程阻塞。多核cpu时，多线程操作会叠加这些cpu时间（墙钟时间不会叠加），所以user或sys时间大于real时间是完全正常的。
3. 并行收集器注重吞吐量，Ergonomics负责自动的调解gc暂停时间和吞吐量之间的平衡，在某个generation被过度使用之前，GC ergonomics就会启动一次GC。



### CMS GC日志

JVM选项：`-XX:+UseConcMarkSweepGC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps`

```shell
Java HotSpot(TM) 64-Bit Server VM (25.60-b23) for windows-amd64 JRE (1.8.0_60-b27), built on Aug  4 2015 11:06:27 by "java_re" with MS VC++ 10.0 (VS2010)
Memory: 4k page, physical 8288692k(2000460k free), swap 19298740k(4849080k free)
CommandLine flags: -XX:InitialHeapSize=2097152 -XX:MaxHeapSize=2097152 -XX:MaxNewSize=700416 -XX:MaxTenuringThreshold=6 -XX:NewSize=700416 -XX:OldPLABSize=16 -XX:OldSize=1396736 -XX:+PrintGC -XX:+PrintGCDateStamps -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseConcMarkSweepGC -XX:-UseLargePagesIndividualAllocation -XX:+UseParNewGC 
2020-01-07T10:43:47.469+0800: 0.145: [GC (Allocation Failure) 2020-01-07T10:43:47.469+0800: 0.145: [ParNew: 512K->64K(576K), 0.0015440 secs] 512K->441K(1984K), 0.0017642 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
2020-01-07T10:43:47.553+0800: 0.228: [GC (Allocation Failure) 2020-01-07T10:43:47.553+0800: 0.228: [ParNew: 576K->63K(576K), 0.0006414 secs] 953K->553K(1984K), 0.0007105 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
2020-01-07T10:43:47.589+0800: 0.265: [GC (Allocation Failure) 2020-01-07T10:43:47.590+0800: 0.265: [ParNew: 575K->62K(576K), 0.0006051 secs] 1065K->627K(1984K), 0.0006889 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
2020-01-07T10:43:48.328+0800: 1.003: [GC (Allocation Failure) 2020-01-07T10:43:48.328+0800: 1.003: [ParNew (promotion failed): 574K->576K(576K), 0.0021027 secs]2020-01-07T10:43:48.330+0800: 1.005: [CMS: 763K->843K(1408K), 0.0042437 secs] 1139K->843K(1984K), [Metaspace: 3422K->3422K(1056768K)], 0.0064693 secs] [Times: user=0.02 sys=0.00, real=0.01 secs] 
2020-01-07T10:43:48.334+0800: 1.010: [GC (CMS Initial Mark) [1 CMS-initial-mark: 843K(1408K)] 848K(1984K), 0.0004684 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
2020-01-07T10:43:48.335+0800: 1.010: [CMS-concurrent-mark-start]
2020-01-07T10:43:48.336+0800: 1.012: [CMS-concurrent-mark: 0.001/0.001 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
2020-01-07T10:43:48.336+0800: 1.012: [CMS-concurrent-preclean-start]
2020-01-07T10:43:48.337+0800: 1.013: [CMS-concurrent-preclean: 0.001/0.001 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
2020-01-07T10:43:48.337+0800: 1.013: [GC (CMS Final Remark) [YG occupancy: 25 K (576 K)]2020-01-07T10:43:48.337+0800: 1.013: [Rescan (parallel) , 0.0004624 secs]2020-01-07T10:43:48.338+0800: 1.014: [weak refs processing, 0.0000380 secs]2020-01-07T10:43:48.338+0800: 1.014: [class unloading, 0.0004634 secs]2020-01-07T10:43:48.339+0800: 1.014: [scrub symbol table, 0.0009359 secs]2020-01-07T10:43:48.339+0800: 1.015: [scrub string table, 0.0001673 secs][1 CMS-remark: 843K(1408K)] 868K(1984K), 0.0022749 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
2020-01-07T10:43:48.341+0800: 1.017: [CMS-concurrent-sweep-start]
2020-01-07T10:43:48.342+0800: 1.017: [CMS-concurrent-sweep: 0.001/0.001 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
2020-01-07T10:43:48.342+0800: 1.017: [CMS-concurrent-reset-start]
2020-01-07T10:43:48.342+0800: 1.017: [CMS-concurrent-reset: 0.000/0.000 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
```

1. ParNew，即新生代使用了ParNew收集器，后面是新生代的收集情况。

2. 初始标记阶段

   GC (CMS Initial Mark)表示CMS开始对老年代进行垃圾回收的初始标记阶段，该阶段从GC Roots开始遍历，只标记与GC Roots直接关联的对象，这个阶段需要暂停用户线程（STW），速度很快。

3. 并发标记阶段

   该阶段进行了细分，但都是和用户线程并发进行的。

   CMS-concurrent-mark表示并发标记阶段，会遍历整个老年代并且标记活着的对象，耗时比较长。由于该阶段运行的过程中用户线程也在运行，就可能发生这样的情况，已经被遍历过的对象引用被用户线程改变，如果发生这种情况，JVM就会标记这个区域为Dirty Card。

   CMS-concurrent-preclean阶段会把上一个阶段被标记为Dirty Card的对象以及可达的对象重新遍历标记，完成后清除Dirty Card标记。另外，一些必要的清扫工作也会做，还会做一些final remark阶段需要的准备工作。

4. 重新标记阶段

   该阶段同样被细分为多个子阶段。

   GC (CMS Final Remark)表示这是CMS的重新标记阶段，会STW，该阶段的任务是完成标记整个老年代所有的存活对象，尽管先前pre cliean阶段尽量应对处理了并发运行时用户线程改变的对象应用的标记，但是不可能跟上对象改变的速度，只是为final remark阶段尽量减少了负担。

   YG occupancy表示年轻代当前的内存占用情况。

   Rescan (parallel)这是整个final remark阶段扫描对象的用时总计，该阶段会重新扫描CMS堆中剩余的对象，重新从”根对象“开始扫描，并且也会处理对象关联。

   第一个子阶段weak refs processing，处理弱引用。

   第二个子阶段class unloading，卸载无用类。

   第三个子阶段scrub symbol table，清理符号表。

   第四个子阶段scrub string table，清理字符串表。

   1 CMS-remark表示经历了上面的阶段后老年代的内存使用情况。

5. 并发清除阶段

   该阶段和用户线程并发执行，不会STW。作用是清除之前标记阶段没有被标记的无用对象，并回收内存。整个过程分为两个子阶段。

   第一个子阶段CMS-concurrent-sweep，清除那些没有被标记的无用对象并回收内存。

   第二个子阶段CMS-concurrent-reset，重新设置CMS算法内部的数据结构，准备下一个CMS声明周期的使用。



### G1 GC日志

JVM选项：`-XX:+UseG1GC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps`

G1收集器上面未作介绍，这里眼熟一下它的日志格式就可以了。

```shell
Java HotSpot(TM) 64-Bit Server VM (25.60-b23) for windows-amd64 JRE (1.8.0_60-b27), built on Aug  4 2015 11:06:27 by "java_re" with MS VC++ 10.0 (VS2010)
Memory: 4k page, physical 8288692k(2074068k free), swap 19298740k(4790652k free)
CommandLine flags: -XX:InitialHeapSize=2097152 -XX:MaxHeapSize=2097152 -XX:+PrintGC -XX:+PrintGCDateStamps -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseG1GC -XX:-UseLargePagesIndividualAllocation 
2020-01-07T10:48:16.680+0800: 0.274: [GC pause (G1 Evacuation Pause) (young), 0.0010579 secs]
   [Parallel Time: 0.7 ms, GC Workers: 4]
      [GC Worker Start (ms): Min: 273.9, Avg: 274.0, Max: 274.0, Diff: 0.0]
      [Ext Root Scanning (ms): Min: 0.1, Avg: 0.2, Max: 0.3, Diff: 0.2, Sum: 0.8]
      [Update RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
         [Processed Buffers: Min: 0, Avg: 0.0, Max: 0, Diff: 0, Sum: 0]
      [Scan RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Code Root Scanning (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Object Copy (ms): Min: 0.3, Avg: 0.4, Max: 0.5, Diff: 0.2, Sum: 1.6]
      [Termination (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
         [Termination Attempts: Min: 1, Avg: 1.3, Max: 2, Diff: 1, Sum: 5]
      [GC Worker Other (ms): Min: 0.0, Avg: 0.0, Max: 0.1, Diff: 0.0, Sum: 0.1]
      [GC Worker Total (ms): Min: 0.6, Avg: 0.6, Max: 0.7, Diff: 0.0, Sum: 2.6]
      [GC Worker End (ms): Min: 274.6, Avg: 274.6, Max: 274.6, Diff: 0.0]
   [Code Root Fixup: 0.0 ms]
   [Code Root Purge: 0.0 ms]
   [Clear CT: 0.1 ms]
   [Other: 0.3 ms]
      [Choose CSet: 0.0 ms]
      [Ref Proc: 0.2 ms]
      [Ref Enq: 0.0 ms]
      [Redirty Cards: 0.1 ms]
      [Humongous Register: 0.0 ms]
      [Humongous Reclaim: 0.0 ms]
      [Free CSet: 0.0 ms]
   [Eden: 1024.0K(1024.0K)->0.0B(1024.0K) Survivors: 0.0B->1024.0K Heap: 1024.0K(2048.0K)->648.1K(2048.0K)]
 [Times: user=0.00 sys=0.00, real=0.00 secs] 
2020-01-07T10:48:17.406+0800: 0.999: [GC pause (G1 Evacuation Pause) (young) (to-space exhausted), 0.0060943 secs]
   [Parallel Time: 5.0 ms, GC Workers: 4]
      [GC Worker Start (ms): Min: 999.5, Avg: 999.6, Max: 999.9, Diff: 0.4]
      [Ext Root Scanning (ms): Min: 0.0, Avg: 1.4, Max: 5.0, Diff: 4.9, Sum: 5.7]
      [Update RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
         [Processed Buffers: Min: 0, Avg: 0.0, Max: 0, Diff: 0, Sum: 0]
      [Scan RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Code Root Scanning (ms): Min: 0.0, Avg: 0.1, Max: 0.3, Diff: 0.3, Sum: 0.3]
      [Object Copy (ms): Min: 0.0, Avg: 1.5, Max: 3.1, Diff: 3.1, Sum: 6.1]
      [Termination (ms): Min: 0.0, Avg: 1.8, Max: 4.3, Diff: 4.3, Sum: 7.4]
         [Termination Attempts: Min: 1, Avg: 1.0, Max: 1, Diff: 0, Sum: 4]
      [GC Worker Other (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [GC Worker Total (ms): Min: 4.6, Avg: 4.9, Max: 5.0, Diff: 0.4, Sum: 19.5]
      [GC Worker End (ms): Min: 1004.5, Avg: 1004.5, Max: 1004.5, Diff: 0.0]
   [Code Root Fixup: 0.0 ms]
   [Code Root Purge: 0.0 ms]
   [Clear CT: 0.1 ms]
   [Other: 1.0 ms]
      [Evacuation Failure: 0.7 ms]
      [Choose CSet: 0.0 ms]
      [Ref Proc: 0.1 ms]
      [Ref Enq: 0.0 ms]
      [Redirty Cards: 0.1 ms]
      [Humongous Register: 0.0 ms]
      [Humongous Reclaim: 0.0 ms]
      [Free CSet: 0.0 ms]
   [Eden: 1024.0K(1024.0K)->0.0B(1024.0K) Survivors: 1024.0K->0.0B Heap: 1672.1K(2048.0K)->1672.1K(2048.0K)]
 [Times: user=0.02 sys=0.02, real=0.01 secs] 
2020-01-07T10:48:17.412+0800: 1.006: [GC pause (G1 Evacuation Pause) (young) (initial-mark), 0.0008040 secs]
   [Parallel Time: 0.5 ms, GC Workers: 4]
      [GC Worker Start (ms): Min: 1005.9, Avg: 1005.9, Max: 1006.1, Diff: 0.2]
      [Ext Root Scanning (ms): Min: 0.0, Avg: 0.2, Max: 0.4, Diff: 0.4, Sum: 0.8]
      [Update RS (ms): Min: 0.0, Avg: 0.2, Max: 0.2, Diff: 0.2, Sum: 0.6]
         [Processed Buffers: Min: 0, Avg: 1.3, Max: 2, Diff: 2, Sum: 5]
      [Scan RS (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Code Root Scanning (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Object Copy (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [Termination (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.1]
         [Termination Attempts: Min: 1, Avg: 1.0, Max: 1, Diff: 0, Sum: 4]
      [GC Worker Other (ms): Min: 0.0, Avg: 0.0, Max: 0.0, Diff: 0.0, Sum: 0.0]
      [GC Worker Total (ms): Min: 0.3, Avg: 0.4, Max: 0.5, Diff: 0.2, Sum: 1.6]
      [GC Worker End (ms): Min: 1006.3, Avg: 1006.3, Max: 1006.3, Diff: 0.0]
   [Code Root Fixup: 0.0 ms]
   [Code Root Purge: 0.0 ms]
   [Clear CT: 0.1 ms]
   [Other: 0.2 ms]
      [Choose CSet: 0.0 ms]
      [Ref Proc: 0.1 ms]
      [Ref Enq: 0.0 ms]
      [Redirty Cards: 0.1 ms]
      [Humongous Register: 0.0 ms]
      [Humongous Reclaim: 0.0 ms]
      [Free CSet: 0.0 ms]
   [Eden: 0.0B(1024.0K)->0.0B(1024.0K) Survivors: 0.0B->0.0B Heap: 1672.1K(2048.0K)->1672.1K(2048.0K)]
 [Times: user=0.00 sys=0.00, real=0.00 secs] 
2020-01-07T10:48:17.413+0800: 1.007: [GC concurrent-root-region-scan-start]
2020-01-07T10:48:17.413+0800: 1.007: [GC concurrent-root-region-scan-end, 0.0000118 secs]
2020-01-07T10:48:17.413+0800: 1.007: [GC concurrent-mark-start]
2020-01-07T10:48:17.413+0800: 1.007: [Full GC (Allocation Failure)  1672K->805K(2048K), 0.0038517 secs]
   [Eden: 0.0B(1024.0K)->0.0B(1024.0K) Survivors: 0.0B->0.0B Heap: 1672.1K(2048.0K)->805.5K(2048.0K)], [Metaspace: 3422K->3422K(1056768K)]
 [Times: user=0.00 sys=0.00, real=0.00 secs] 
```



### 日志分析工具

[在线日志文件分析][https://gceasy.io/]。