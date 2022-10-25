### Garbage collection

#### GC Algorithms

* Serial garbage collector: single CPU machine default (`-XX:+UseSerialGC`)

* Throughput collector: Other machines default (Young Gen Collector: `-XX:+UseParallelGC` Old Gen Collector:`-XX:+UseParallelOldGC`) 

  * Uses multiple threads, stops **all** application threads during both minor and full GCs. 
  * Fully compact old gen during a full GC.

* CMS collector(`-XX:+UseConcMarkSweepGC -XX:+UseParNewGC`)

  * eliminate the long pauses associated with the full GC cycles of the throughput and serial collectors
  * Use background threads to scan old gen
  * Less paused time, more CPU usage
  * Does NOT compact old gen, needs to stop all application threads to clean and compact old gen(full GC)

* G1 Garbage First collector(`-XX:+UseG1GC`)

  * designed to process large heaps (greater than

    about 4 GB)

  * Divides heap into serveral of regions

  * young generation is still collected by stopping all application threads and moving all objects that are alive into the old generation or the survivor spaces.

并行：（Parallel）描述的是多条垃圾收集器线程之间的关系。

说明同一时间有多条这样的线程协同工作，通常默认此时用户线程是处于等待状态。

并发： （Concurrent）： 并发描述的是垃圾收集器线程与用户线程之间的关系。

说明在同一时间垃圾收集器线程与用户线程都在运行。由于用户线程并非被冻结，所以程序仍然是能够响应服务请求的，但由于垃圾收集器线程占有了一部分的系统资源，此时应用程序的处理的吞吐量将会受到一定的影响。


Causing and Disabling Explicit GC: 

* Causing:
  * `System.gc()` Always trigger a full gc. Bad idea in production. Useful examples:
    * small benchmarks that run a bunch of code to properly warm up the JVM
  * `jcmd <process id> GC.run`
* Disabling:
  * `-XX:+DisableExplicitGC`

#### GC Tuning

* Sizing the heap

  * The size of the heap is controlled by two values: an initial value (specified with `-XmsN`) and a maximum value (`-XmxN`).![Screen Shot 2022-10-21 at 08.21.56](/Users/peter/Desktop/reading notes/java-performance/Screen Shot 2022-10-21 at 08.21.56.png)

  * A good rule of thumb is to size the heap so that it is 30% occupied after a **full GC.**

  * Set both is how much size heap is needed (e.g.,

    -Xms4096m -Xmx4096m).

* Sizing the generation

  * `Initial Young Gen Size = Initial Heap Size / (1 + NewRatio)`

  * `-XX:NewRatio=N`

    `young generation : old generation`

  * `-XX:NewSize=N`

    Set the initial size of the young generation.

  * `-XX:MaxNewSize=N`

    Set the maximum size of the young generation.

  * `-XmnN`

    Shorthand for setting both NewSize and MaxNewSize to the same value.

* Sizing permgen and metaspace
  * For permgen, the sizes are specified via these flags: -XX:PermSize=N and -XX:MaxPermSize=N. 
  * Metaspace is sized with these flags:XX:MetaspaceSize=N and -XX:MaxMetaspaceSize=N.

#### Controlling parallelism

* Set the number of parallel gc threads by adding flag: 

  ```-XX:ParallelGCThreads=N```

* For machine with more than eight CPUS(where N is the number of CPUs): 

  ```ParallelGCThreads = 8 + ((N - 8) * 5 / 8)```

  8 threads with less than or equal to eight CPUS.

* To see how the JVM is resizing the spaces in an application, set the `-XX:+PrintAdaptiveSizePolicy` flag.

#### GCLog

* `-XX:+PrintGCDetails` print detailed gc log
* `-XX:+PrintGCDateStamps` `-XX:+PrintGCTimeStamps` print date stamps / time stamps in gc log
* `-XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=N -XX:GCLogFileSize=N` GC log rotation settings
* `-Xloggc:filename`

### Throughput Collector

Adaptive sizing in the throughput collector will resize the heap (and the generations) in order to meet its pause time goals. Those goals are set with these flags:

```-XX:MaxGCPauseMillis=N and -XX:GCTimeRatio=N```

$$GCTimeRatio = \frac{Throughput}{1-Throughput}$$

For a throughput goal of 95% (0.95), this equation yields a GCTimeRatio of 19. (which means application spend 5% of time doing GC)