JavaForHighThroughput
=====================

In this tutorial i will give an overview of how to use the Java language for high volume traffic architectures.

1. Java libraries for high performance and throughput

  - Java Chronicle 
  
    1. Horizontal scalability is valueable for high throughput.
    2. If your goal is low latency, you have to keep your system as simple as possible. The less the system has to do the less time it          takes.
    3. Understand what critical code is doing in your application.
  
  Micro-bursts are critical to understanding your system.

The performance of some systems are characterised in terms of transactions per day.  This implies that if no transactions were completed for the first 23 hours and all of them completed in the last hour, you would still perform this many transactions per day.  Often the transactions per day is quoted because its a higher numbers, but in my case having all day to smooth out the work load sounds like a luxury.

Some systems might be characterised in terms of the number of transactions per second.  This may imply that those transactions can start and complete in one second, but not always.  If you have 1000 transactions and one comes in every milli-seconds, you get an even response time.  What I find more interesting is the number of transactions in the busiest second of a day.  This gives you an indication of the flow rate your system should be able to handle.

How to take the full advantage of multicore systems.

Multi transaction databases:

How we can handle multiple connections without running into a deadlock?

How do use chronicle - a "Hello World" example 
  
  Chronicle vs. Vanilla:

  - Vanilla uses concurrent writers (even across processes) and file rolling. It's overhead of about 200 nano-seconds per message.
  - Index chronicle is simpler and for small messages can do 20+ million messages per second whereas VanillaChronicle can do about 4 million messages per second.
  - IndexChronicle is faster for single writer (if you have about one million messages per second it shouldn't matter)
     
Choosing the right garbage collection mode:

1. Dynamic Garbage Collection Mode 
2. Static Single-Spaced Parallel Garbage Collection

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

 Reduce GC frequency

In generational GC algorithms, collection frequency for a generation can be decreased by (i) reducing the object allocation/promotion rate and (ii) increasing the size of the generation.

In Hotspot JVM, the duration of young GC pause depends on the number of objects that survive a collection and not on the young generation size itself. The impact of increasing the young generation size on the application performance has to be carefully assessed:

    Increasing the young generation size may lead to longer young GC pauses if more data survives and gets copied in the survivor spaces, or if more data gets promoted to the old generation in each young collection. Longer GC pauses may lead to an increased application latency and/or reduced throughput.
    On the other hand, the pause duration may not increase if the number of objects that survive each collection does not increase substantially. In this case, the reduction in GC frequency may lead to a reduction in overall application latency and/or an increase in the throughput.

For applications that mostly create short-lived objects, you will only need to control the aforementioned parameters. For applications that create long-lived objects, there is a caveat; the promoted objects may not be collected in an old generation GC cycle for a long time. If the threshold at which old generation GC is triggered (expressed as a percentage of the old generation that is filled) is low, the application can get stuck in incessant GC cycles. You can avoid this by triggering GC at a higher threshold.

As our application maintains a large cache of long-lived objects in the heap, we increased the threshold of triggering old GC by setting: -XX:CMSInitiatingOccupancyFraction=92 -XX:+UseCMSInitiatingOccupancyOnly. We also tried to increase the young generation size to reduce the young collection frequency, but reverted this change as it increased the 99.9th percentile application latency. 

Reduce GC pause duration

The young GC pause duration can be reduced by decreasing the young generation size as it may lead to less data being copied in survivor spaces or promoted per collection. However, as previously mentioned, we have to observe the impact of reduced young generation size and the resulting increase in GC frequency on the overall application throughput and latency. The young GC pause duration also depends on tenuring thresholds and the old generation size.

Tune your response time into milliseconds range:
================================================

1. Immutable objects.
2. Less synchronized method
3. Locking order should be well documented, and handled carefully
4. Use profiler
5. Use Amdhal's law, and find the sequential execution path
6. Use Java 8 concurrency utilities, and locks
7. Avoid Thread priorities as they are platform dependent
8. JVM warmup can be used
9. Prefer unfair locking strategy
10. Avoid context-switching (many threads lead to counter productive)
11. Avoid boxing, un-boxing
12. Give attention to compiler warnings 
13. Number of threads should be equal or lesser than the number of core


PetaPyte JVM

Starting from the observation that Java responds much faster if you can keep your data in memory rather than going to a database or some other external resource, Lawrey described the kind of problems you hit when you go above the 32GB range that Java is reasonably comfortable in. As you’d expect GC pause times become a major problem, but also memory efficiency drops significantly, and you have the problem of how to recover in the event of a failure.

JVM performance optimization
============================

A two- to four-second pause is not acceptable for most enterprise applications, so Java application instances are stalled out at 2 to 4 GB, despite their need for more memory. On some 64-bit systems, with lots of JVM tuning for scale, it is possible to run 16 GB or even 20 GB heaps and meet typical response-time SLAs. But compared to where Java heap sizes should be today, we're still way off. The limitation lies in the JVM's inability to handle fragmentation without a stop-the-world GC. As a result, Java application developers are stuck doing two tasks that most of us deplore:

Architecting or modeling deployments in chopped-up large instance pools, even though it leads to a horrible operations monitoring and management situation.
Tuning and re-tuning the JVM configuration, or even the application, to "avoid" (meaning postpone) the worst-case scenario of a stop-the-world compaction pause. The most that developers can hope for is that when a pause happens, it won't be during a peak load time. This is what I call a Don Quixote task, of chasing an impossible goal.

JAVA 8 Memory layout:
=====================

Important: PermGem is replaced with Metaspace.

1. Memory layout of classes
2. Memory layout of Subclasses
3. Memory layout of inheritance 

 Turn THP (Transparent Huge Pages) OFF.

2. Set vm.min_free_kbytes to AT LEAST 1GB (8GB on larger systems).

3. Set Swappiness to 0.

4. Set zone_reclaim_mode to 0.
5. Turn HT (Hyper-threading) ON. (double the vcore run queues --> umpteen times lower likelihood of waiting for a cpu).

Tunning factors for high throughput applications:

-> chunk size
-> number of threads

Anatomy of a Java object ( Part 1 )

1. Class: A pointer to the class information, which describes the object type. In the case of an array of int fields, this      is a pointer to the int[] class.

2. Flags: A collection of flags that describe the state of the object, including the hash code for the object if it has one,    and the shape of the object (that is, whether or not the object is an array).

3. Lock: The synchronization information for the object. Whether the object is currently synchronized.
   Size : The size of the array.

ThreadLocale and the Garbage Collector:

Don't forget about the remove() method if using no JDK classes.
Low latency coding (less than 10 microseconds).
High throughput (over 100K request/responses per second).
Using sun.misc.Unsafe
Using Chronicle for low latency persistence.

Find the "right" way between high throughput and low latencey guide.

A Java Garbage collection mini tutorial

1. Overview of different garbage collection strategies.
2. Old space vs. young space.
3. Implementing our own gc strategy.
4. Differences in garbage collection strategy Java 7 vs. Java 8


Tune the Thread-Local Area Size

Thread Local Areas (TLAs) are chunks of free memory used for object allocation. The TLAs are reserved from the heap and given to the Java threads on demand, so that the Java threads can allocate objects without having to synchronize with the other Java threads for each object allocation.

Increasing the preferred TLA size speeds up allocation of small objects when each Java thread allocates a lot of small objects, as the threads won’t have to synchronize to get a new TLA as often.

In Oracle JRockit JVM R27.3 and later releases the preferred TLA size also determines the size limit for objects allocated in the nursery. Increasing the TLA size will thus also allow larger objects to be allocated in the nursery, which is beneficial for applications that allocate a lot of large objects. In older versions you need to set both the TLA size and the Large Object Limit to allow larger objects to be allocated in the nursery. A JRA recording will show you statistics on the sizes of large objects allocated by your application. For good performance you can try setting the preferred TLA size at least as large as the largest object allocated by your application.

Hints for heap space friendly programming :-)
=============================================

- be careful with String, StringBuffer and StringReader

How to create a heap dump ?
===========================

JVM-parameter

You can start the JVM with different options to create heap dumps:

- XX:+HeapDumpOnOutOfMemoryError

A heap dump will be create automatically if a OutOfMemoryError has been thrown:

- XX:+HeapDumpOnCtrlBreak

Creates a heap dump if the correspondending keys are pressed:

- agentlib:hprof=heap=dump,format=b,doe=y

It uses a HPROF-agent.

Comparing Java and C++ for low latency applications
===================================================

The ability to create, destroy and reset objects as and when I want. My biggest convenience with C++ is the ability to call an objects destructor without deallocating the objects memory.

The ability to directly use macros, GCC directives ( for stuff like branch prediction) and in line assembly. 

The possibility of hand coding a latency critical sub routine in assembly and directly integrating it with code is priceless.
Native OS calls( rare) , seamless shared memory and the ability to do certain tweaks like creating NUMA aware algorithms, pinning certain threads to cores, DDIO etc. 

How to write your own sub routine in assembly in integrate it into your java application.

These things definitely help shave anywhere from 2 to 10 micros off your trading stack depending on what you are upto.

How to tune JVM for microseconds latency applications.

The JRockit JVM Runs JIT Compilation
====================================

The first step of code generation is the Just-In-Time (JIT) compilation. This compilation allows your Java application to start and run while the code that is generated is not highly optimized for the platform. Although the JIT is not actually part of the JVM standard, it is, nonetheless, an essential component of Java. In theory, the JIT comes into use whenever a Java method is called, and it compiles the bytecode of that method into native machine code, thereby compiling it “just in time” to execute.

Java data structures for high throughput
========================================

Stream processing is a programming paradigm fit for a class
of data driven applications which must manipulate highvolumes
of data in a timely and responsive fashion. Example
applications include video processing, digital signal processing,
monitoring of business processes and intrusion detection.
While some applications lend themselves naturally to a
distributed implementation, we focus on single node systems
and, in particular, on programming language support for ef-
ficient implementation of systems that require microsecond
latencies and low packet drop rates.

Abstract Syntax Tree and execution speed.
 
 
 










  
 














  



