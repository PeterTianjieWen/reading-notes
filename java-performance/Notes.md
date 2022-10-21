### Garbage collection

#### GC Algorithms

* Serial garbage collector: single CPU machine default (`-XX:+UseSerialGC`)

* Throughput collector: Other machines default (`-XX:+UseParallelGC -XX:+UseParallelOldGC`) 

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

Causing and Disabling Explicit GC: 

* Causing:
  * `System.gc()` Always trigger a full gc. Bad idea in production. Useful examples:
    * small benchmarks that run a bunch of code to properly warm up the JVM
  * `jcmd <process id> GC.run`
* Disabling:
  * `-XX:+DisableExplicitGC`

#### GC Tuning

* Sizing the heap

  * The size of the heap is controlled by two values: an initial value (specified with `-XmsN`) and a maximum value (`-XmxN`).![Screen Shot 2022-10-21 at 08.21.56](/Users/peter/Desktop/Screen Shot 2022-10-21 at 08.21.56.png)

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