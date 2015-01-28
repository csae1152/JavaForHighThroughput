JavaForHighThroughput
=====================

In this tutorial i will give an overview of how to use the Java language for high volume traffic architectures.

1. Java libraries for high performance and throughput

  - Java Chronicle 
  
    1. Horizontal scalability is valueable for high throughput.
    2. If your goal is low latency, you have to keep your system as simple as possible. The less the system has to do the          less time it takes.
    3. Understand what critical code is doing in your application.
  
  Chronicle vs. Vanilla:

  - Vanilla uses concurrent writers (even across processes) and file rolling. It's overhead of about 200 nano-seconds per message.
  - Index chronicle is simpler and for small messages can do 20+ million messages per second whereas VanillaChronicle can do about 4 million messages per second.
  - IndexChronicle is faster for single writer (if you have about one million messages per second it shouldn't matter)
  
Choosing the right garbage collection mode:

1. Dynamic Garbage Collection Mode 
2. Static Single-Spaced Parallel Garbage Collection
3. 

Manually Tuning the Nursery Size

The nursery or young generation is the area of free chunks in the heap where objects are allocated when running a generational garbage collector (-Xgc:throughput, -Xgc:genpar or -Xgc:gencon). A nursery is valuable because most objects in a Java application die young. Collecting garbage from the young space is preferable to collecting the entire heap, as it is a less expensive process and most objects in the young space would already be dead when the garbage collection is started.

Dynamic Garbage Collection Mode Optimized for Deterministic Pause Times

java -Xgc:deterministic -Xms:1g -Xmx:1g myApplication

Dynamic Garbage Collection Mode Optimized for Short Pauses

Set the pausetime priority as follows:

java -Xgc:pausetime myApplication

Static Generational Concurrent Garbage Collection

To use a generational concurrent garbage collector, enter the following at the command line:

java -Xgc:gencon myApplication

To use a single-spaced concurrent garbage collector, enter the following at the command line:

java -Xgc:singlecon myApplication

We can select a specific garbage collector by using -Xgc:<option>, with option being one of the following:

  - singlecon – Single-space, concurrent garbage collection. This mode reduces application throughput but keeps pause times to a minimum.
  - 
  - gencon – Generational, concurrent garbage collection. The gencon mode is better than the singlecon mode for applications that allocate lots of short-lived objects. This mode reduces application throughput but keeps pause times to a minimum.
  - 
  - singlepar – Single-space, parallel garbage collection. This mode increases pause times when compared with the concurrent mode but maximizes throughput.
  - 
  - genpar – Generational, parallel garbage collection. This mode is generally better than the singlepar mode for applications that allocate lots of short-lived objects. In this mode, a higher number of garbage collections are performed than in the singlepar mode, but the individual pause times are shorter, resulting in lower fragmentation in the old generation space.
  - 
  - genconpar – Generational garbage collection. Sets the garbage collection mode to generational with a concurrent mark algorithm and a parallel sweep algorithm.
  - 
  - genparcon – Generational garbage collection. Sets the garbage collection mode to generational with a parallel mark algorithm and a concurrent sweep algorithm.
  
  - singleconpar – Single-space garbage collection. Sets the garbage collection mode to single-spaced with a concurrent mark algorithm and a parallel sweep algorithm.
  - 
  - singleparcon – Single-space garbage collection. Sets the garbage collection mode to single-spaced with a parallel mark and a concurrent sweep algorithm.
  - 
  - throughput – The garbage collector is optimized for application throughput. This means that the garbage collector works as effectively as possible, giving as much CPU resources to the Java threads as possible. This might, however, cause non-deterministic pauses when the garbage collector stops all Java threads for garbage collection.
  - 
-  pausetime – The garbage collector is optimized for short pauses. This means that the garbage collection works concurrently with the Java application when necessary, in order to avoid pausing the Java threads. This inflicts a slight performance overhead to the application, as the concurrent garbage collector demands more system resources (CPU time and memory) than the parallel garbage collector that is used for optimal throughput. The target pause time is by default 200 milliseconds, which can be changed by using -XpauseTarget.

-  deterministic – Optimizes the garbage collector for very short and deterministic pause times. The garbage collector tries to keep the garbage collection pauses below a given pause target. The performance depends on the application and the hardware. Running on slower hardware, with a different heap size or with a large amount of live data can break the deterministic behavior or cause performance degradation over time; faster hardware or a less amount of live data might allows us to set a lower pause target. The pause target for the deterministic mode is by default 30 milliseconds, which can be changed by using -XpauseTarget



Warm up your JVM. Bytecode starts starts off being interpreted for Hotspot and then gets compiled on the server after 10K observations. Tiered Compilation can be a good stop gap.

Classloading is a sequential process that involves IO to disk. Make sure all the classes for your main transaction flows are loaded upfront and that they never get evicted from the perm generation.

Follow the "Single Writer Principle" to avoid contention and the queueing effect implications of Little's Law, plus study Amdhal's Law for what can be parallel and is it worth it.

Model you business domain and ensure all your algorithms are O(1) or at least O(log n). This is probably the biggest cause of performance issues in my experience. Make sure you have performance tests to cover the main cases.

Low-latency in Java is not just limited to Java. You need to understand the whole stack your code is executing on. This will involve OS tuning, selecting appropriate hardware, tuning systems software and device drivers for that hardware.

Be realistic. If you need low-latency don't run on a hypervisor. Ensure you have sufficient cores for all threads that need to be in the runnable state.

Cache misses are your biggest cost to performance. Use algorithms that are cache friendly and set affinity to processor cores either with taskset or numactl for a JVM or JNI for individual threads.

Consider an alternative JVM like Zing from Azul with a pause-less garbage collector.

Most importantly get someone involved with experience. This will save you so much time in the long run.

Hypervisor mode and low-latency

- What is hypervisor mode












  



