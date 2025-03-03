# JVM垃圾回收算法

## Serial GC
"Stop the world": 在进行垃圾回收时，必须暂停其他的工作线程，直到它收集完成。
![serial_gc.png](serial_gc.png)

## ParNew GC
Serial收集器的多线程并行版本。
![par_new.png](par_new.png)

JDK5发布的CMS收集器是第一款真正意义上支持并发的垃圾收集器，它首次实现了让垃圾收集线程与用户线程（基本上）同时工作。但是作为老年代的垃圾回收器，CMS无法与新生代
收集器Parallel Scavenge配合工作，新生代只能选择ParNew或者Serial。ParNew收集器是激活CMS(使用-XX:+UseConcMarkSweepGC)后的默认新生代收集器。

并行：多条垃圾回收线程之间的关系，说明同一时间有多条这样的线程在协同工作，默认此时用户线程是出于等待状态。
并发：垃圾回收线程与用户线程之间的关系，说明同一时间垃圾回收线程与用户线程都在运行。由于垃圾回收线程占用了一定的系统资源，所以应用程序的吞吐量受到一定的影响。