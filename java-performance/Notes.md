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

#### Minor GC Log:
```
17.806: [GC [PSYoungGen: 227983K->14463K(264128K)]
280122K->66610K(613696K), 0.0169320 secs]
[Times: user=0.05 sys=0.00, real=0.02 secs]
```

This GC occurred *17.806* seconds after the program began. Objects in the young generation
now occupy *14463* KB (14 MB, in the survivor space); before the GC, they
occupied *227983* KB (227 MB). (Actually, 227893 KB is only *222* MB. The total size of the young generation at this point is *264* MB.

Meanwhile the overall occupancy of the heap (both young and old generations) decreased from *280* MB to *66* MB, and the size of the entire heap at this point in time was
613 MB. The operation took less than 0.02 seconds (the 0.02 seconds of real time at the
end of the output is 0.0169320 seconds—the actual time—rounded). **The program was
charged for more CPU time than real time because the young collection was done by multiple threads (in this configuration, four threads).**

#### Full GC Log
```
64.546: [Full GC [PSYoungGen: 15808K->0K(339456K)]
[ParOldGen: 457753K->392528K(554432K)] 473561K->392528K(893888K)
[PSPermGen: 56728K->56728K(115392K)], 1.3367080 secs]
[Times: user=4.44 sys=0.01, real=1.34 secs]
```

#### Adaptive and static heap size
Adaptive sizing in the throughput collector will resize the heap (and the generations) in order to meet its pause time goals. Those goals are set with these flags:

```-XX:MaxGCPauseMillis=N and -XX:GCTimeRatio=N```

$$GCTimeRatio = \frac{Throughput}{1-Throughput}$$

For a throughput goal of 95% (0.95), this equation yields a GCTimeRatio of 19. (which means application spend 5% of time doing GC)

### CMS Collector
* Stop all application threads to collect the young generation
* Clean old generation concurrently(in a concurrent cycle)
* If necessary, CMS performs a full GC.(Data fragmentation)

#### Minor Gc Log
```
89.853: [GC 89.853: [ParNew: 629120K->69888K(629120K), 0.1218970 secs]
1303940K->772142K(2027264K), 0.1220090 secs]
[Times: user=0.42 sys=0.02, real=0.12 secs]
```

#### Concurrent cycle

* Initial mark phase is responsible for finding all the GC root objects in the heap.
```
89.976: [GC [1 CMS-initial-mark: 702254K(1398144K)]
772530K(2027264K), 0.0830120 secs]
[Times: user=0.08 sys=0.00, real=0.08 secs]
```

* Mark phase does not stop the application threads. 
```
90.059: [CMS-concurrent-mark-start]
90.887: [CMS-concurrent-mark: 0.823/0.828 secs]
[Times: user=1.11 sys=0.00, real=0.83 secs]
```

* Preclean phase
```
90.887: [CMS-concurrent-preclean-start]
90.892: [CMS-concurrent-preclean: 0.005/0.005 secs]
[Times: user=0.01 sys=0.00, real=0.01 secs]
```
* Remark phase (NOT Concurrent, use preclean-phase to wait for young generation to be 50% full to avoid back-to-back phases)
```
90.892: [CMS-concurrent-abortable-preclean-start]
92.392: [GC 92.393: [ParNew: 629120K->69888K(629120K), 0.1289040 secs]
1331374K->803967K(2027264K), 0.1290200 secs]
[Times: user=0.44 sys=0.01, real=0.12 secs]
94.473: [CMS-concurrent-abortable-preclean: 3.451/3.581 secs]

[Times: user=5.03 sys=0.03, real=3.58 secs]
94.474: [GC[YG occupancy: 466937 K (629120 K)]
94.474: [Rescan (parallel) , 0.1850000 secs]
94.659: [weak refs processing, 0.0000370 secs]
94.659: [scrub string table, 0.0011530 secs]
[1 CMS-remark: 734079K(1398144K)]
1201017K(2027264K), 0.1863430 secs]
[Times: user=0.60 sys=0.01, real=0.18 secs]
```
* Sweep phase
```
94.661: [CMS-concurrent-sweep-start]
95.223: [GC 95.223: [ParNew: 629120K->69888K(629120K), 0.1322530 secs]
999428K->472094K(2027264K), 0.1323690 secs]
[Times: user=0.43 sys=0.00, real=0.13 secs]
95.474: [CMS-concurrent-sweep: 0.680/0.813 secs]
[Times: user=1.45 sys=0.00, real=0.82 secs]
```
* Reset phase
```
95.474: [CMS-concurrent-reset-start]
95.479: [CMS-concurrent-reset: 0.005/0.005 secs]
[Times: user=0.00 sys=0.00, real=0.00 secs]
```
Problems after cycle
* Concurrent mode failure -> Full GC
```
267.006: [GC 267.006: [ParNew: 629120K->629120K(629120K), 0.0000200 secs]
267.006: [CMS267.350: [CMS-concurrent-mark: 2.683/2.804 secs]
[Times: user=4.81 sys=0.02, real=2.80 secs]
(concurrent mode failure):
1378132K->1366755K(1398144K), 5.6213320 secs]
2007252K->1366755K(2027264K),
[CMS Perm : 57231K->57222K(95548K)], 5.6215150 secs]
[Times: user=5.63 sys=0.00, real=5.62 secs]
```
* Promotion failed - fragmented heap
```
6043.903: [GC 6043.903:
[ParNew (promotion failed): 614254K->629120K(629120K), 0.1619839 secs]
6044.217: [CMS: 1342523K->1336533K(2027264K), 30.7884210 secs]
2004251K->1336533K(1398144K),
[CMS Perm : 57231K->57231K(95548K)], 28.1361340 secs]
[Times: user=28.13 sys=0.38, real=28.13 secs]
```
* Permernant generation has filled up -> full GC
```
279.803: [Full GC 279.803:
[CMS: 88569K->68870K(1398144K), 0.6714090 secs]
558070K->68870K(2027264K),
[CMS Perm : 81919K->77654K(81920K)],
0.6716570 secs]
```
