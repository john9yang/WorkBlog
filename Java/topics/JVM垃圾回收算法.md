# JVM经典垃圾回收算法

## Serial GC
"Stop the world": 在进行垃圾回收时，必须暂停其他的工作线程，直到它收集完成。

![serial_gc.png](serial_gc.png){width="600"}

## ParNew GC
Serial收集器的多线程并行版本。

![par_new.png](par_new.png){width="600"}

JDK5发布的CMS收集器是第一款真正意义上支持并发的垃圾收集器，它首次实现了让垃圾收集线程与用户线程（基本上）同时工作。但是作为老年代的垃圾回收器，CMS无法与新生代
收集器Parallel Scavenge配合工作，新生代只能选择ParNew或者Serial。ParNew收集器是激活CMS(使用-XX:+UseConcMarkSweepGC)后的默认新生代收集器。

并行：多条垃圾回收线程之间的关系，说明同一时间有多条这样的线程在协同工作，默认此时用户线程是出于等待状态。
并发：垃圾回收线程与用户线程之间的关系，说明同一时间垃圾回收线程与用户线程都在运行。由于垃圾回收线程占用了一定的系统资源，所以应用程序的吞吐量受到一定的影响。

## Parallel Old GC

<p>新生代:Parallel Scavenge <shortcut>+</shortcut> 老年代：Parallel Old</p>

## CMS GC

在并发标记和并发清理的时候，用户线程还是继续的，程序在运行自然就会产生垃圾，但这一部分垃圾产生在标记过程结束以后，CMS无法在当次收集中处理掉它们，只好留在下一次垃圾收集的时候
再清理掉，容易产生“浮动垃圾”。

## Garbage First

面向堆内存任何部分来组成回收集(Collection Set), 衡量标准不再是哪个分代，而是哪块内存中存放的垃圾数量最大，收益最多，这就是G1收集器的Mixed GC模式。
把Java堆分成多个大小相等的独立区域(Region),每个Region根据需要划为eden空间，survivor空间，老年代空间，新生代和老年代不再是固定的，是一系列region的动态集合。
用户可以设定允许的收集停顿时间（使用参数-XX：MaxGCPauseMillis置顶，默认是200ms）。

# 低延迟垃圾收集器

衡量垃圾回收器的三个指标：1.内存占用(Footprint) 2.吞吐量(Throughput) 3.延迟(Latency)

## Shenandoah收集器

也是使用基于Region的布局

## ZGC收集器
染色指针技术(Colored Pointer), 把标记信息记在引用对象的指针上。