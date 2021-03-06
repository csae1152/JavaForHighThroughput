JavaForHighThroughput
=====================

In this tutorial i will give an overview of how to use the Java language for high volume traffic architectures.

1. Java libraries for high performance and throughput

  - Java Chronicle Usage examples
  
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

Tune the Thread-Local Area Size

Thread Local Areas (TLAs) are chunks of free memory used for object allocation. The TLAs are reserved from the heap and given to the Java threads on demand, so that the Java threads can allocate objects without having to synchronize with the other Java threads for each object allocation.

Increasing the preferred TLA size speeds up allocation of small objects when each Java thread allocates a lot of small objects, as the threads won’t have to synchronize to get a new TLA as often.

In Oracle JRockit JVM R27.3 and later releases the preferred TLA size also determines the size limit for objects allocated in the nursery. Increasing the TLA size will thus also allow larger objects to be allocated in the nursery, which is beneficial for applications that allocate a lot of large objects. In older versions you need to set both the TLA size and the Large Object Limit to allow larger objects to be allocated in the nursery. A JRA recording will show you statistics on the sizes of large objects allocated by your application. For good performance you can try setting the preferred TLA size at least as large as the largest object allocated by your application.
 
Manually Tune Compaction

Compaction is the process of moving chunks of allocated space toward the lower end of the heap, helping to create contiguous free memory at the other end. The JRockit JVM does partial compaction of the heap at each old collection.

The default compaction setting for static garbage collectors (-Xgc or -XXsetGC) use a dynamic compaction scheme that tries to avoid “peaks” in the compaction times. This is a compromise between keeping garbage collection pauses even and maintaining a good throughput, so it doesn't necessarily give the best possible throughput. Tuning the compaction can pay off well, depending on the application's characteristics.

There are two ways to tune the compaction for better throughput; increasing the size of the compaction area and increasing the compact set limit. Increasing the size of the compaction area will help reduce the fragmentation on the heap. Increasing the compact set limit will implicitly allow larger areas to be compacted at each garbage collection. This reduces the garbage collection frequency and makes allocation of large objects faster, thus improving the throughput.

High througput data structures
==============================

If you want to implement fast and correct monetary arithmetic operations in Java, stick to the following rules:
  
  Store monetary values in the smallest currency units (for example, cents) in long variables.
  Avoid working with non-integral values while using double (calculate in the smallest currency units).
  Add/subtract using long.
  Round any multiplication/division results using Math.round/rint/ceil/floor (per your system requirements).
  Your calculations should fit into 52 bits (double precision).
  
 
Comparing for and while loop for high througput.

1. for-loop vs. while-loop

What is Low-Latency Java

• Response Time Requirements
– Typical latency requirement around 5 – 50 ms
– Max latency = transaction time + max pause time
• Typical Applications
– Financial
• Automatic trading
• Matching server

Generational Concurrent Collectors
• Use Case
– Normal heap sizes
– Reasonable amount of live data
• Nursery Collector
– Stop the world
– Parallel
• Old Collection
– Mostly concurrent Mark and Sweep
• Some short parallel pauses
– Partial compaction

When is the right time to optimize GC?

GC behavior can vary with code-level optimizations and workloads. So it is important to tune GC on a codebase that is near completion and includes performance optimizations. But it is also necessary to perform preliminary analysis on an end-to-end basic prototype with stub computation code and synthetic workloads representative of production environments. This will help capture realistic bounds on latency and throughput based on the architecture and guide the decision on whether to scale up or scale out.

Stop The World and gather data: The importance of GC logs

The richest source of data for the state of garbage collection in a system based on a HotSpot JVM are the GC logs. If your JVM is not generating GC logs with timestamps, you’re missing out on a critical source of data to analyze and solve pausing issues. This is true for development environments, staging, load testing and most importantly, in production. You can get data about all GC events in your system, whether they were completed concurrently or caused a stop-the-world pause: how long did they take, how much CPU they consumed, and how much memory was freed. From this data, you’re able to understand the frequency and duration of these pauses, their overhead, and move on to taking actions to reduce them.

With fastutil 6, a new set of classes makes it possible to handle very large collections: in particular, collections whose size exceeds 231. Big arrays are arrays-of-arrays handled by a wealth of static methods that act on them as if they were monodimensional arrays with 64-bit indices, and big lists provide 64-bit list access. The size of a hash big set is limited only by the amount of core memory. The usual methods from java.util.Arrays and similar classes have been extended to big arrays: have a look at the Javadoc documentation of BigArrays and IntBigArrays to get an idea of the generic and type-specific methods available.

fastutil provides in many cases the fastest implementations available. Please have a look at recent benchmarks, as these ones. You can find many other implementations of primitive collections (e.g., HPPC, Koloboke, etc.). Sometimes authors are a little bit quick in defining their implementations the “fastest available“: the truth is, you have to take decisions in any implementation. These decisions make your implementation faster of slower in different scenarios. I suggest to always test speed within your own application, rather than relying on general benchmarks, and ask the authors for suggestions about how to use the libraries in an optimal way. In particular, when testing hash-based data structures you should always set explicitly the load factor, as speed is strongly dependent on the length of collision chains.
 
 Going into the ultra low latency area...
 
 Respond times lower than 5 ms.
 
FastUtil supports that standard Java iteration approaches of using an explicit iterator and using the Java SE 5-introduced for-each loop. FastUtil collections even support the JDK 8 style using .forEach (assuming code is built and run on JDK 8) because the FastUtil collections implement java.lang.Iterable. These are demonstrated in the next code listing.
Iterating FastUtil Collections in Standard Java Style.

Designing high throughput applications with Java 8.

Java 8 Streams and concurrency.

You can pass a lambda to a Stream. You don't have to take care about concurrency.

When is the right time to optimize GC?

GC behavior can vary with code-level optimizations and workloads. So it is important to tune GC on a codebase that is near completion and includes performance optimizations. But it is also necessary to perform preliminary analysis on an end-to-end basic prototype with stub computation code and synthetic workloads representative of production environments. This will help capture realistic bounds on latency and throughput based on the architecture and guide the decision on whether to scale up or scale out.

Steps to optimize GC

Here are some high level steps to optimize GC for high-throughput, low-latency requirements. Also included, are the details of how this was done for the feed data platform prototype. We saw the best GC performance with ParNew/CMS, though we also experimented with the G1 garbage collector.

1. Understand the basics of GC
Understanding how GC works is important because of the large number of variables that need to be tuned. Oracle's whitepaper on Hotspot JVM Memory Management is an excellent starting point to become familiar with GC algorithms in Hotspot JVM. To understand the theoretical aspects of the G1 collector, check out this paper.

2. Scope out GC requirements
There are certain characteristics of GC that you should optimize, to reduce its overhead on application performance. Like throughput and latency, these GC characteristics should be observed over a long-running test to ensure that the application can handle variance in traffic while going through multiple GC cycles.

JMH profilers are a cheap and convenient way to find out the bottlenecks in your microbenchmarks: they try to avoid measuring as much of JMH code as possible and you can use any subset of them simultaneously.
STACK profiler makes a thread dump every 10 ms by default, but you may probably want to decrease this interval a little on powerful boxes.
JIT compiler profilers (COMP / HS_COMP) are recommended for use on most of benchmarks - they will let you know if you have insufficiently warmed up your code.

Performance techniques used in the Hotspot JVM
==============================================

Constants

Use constants when you can.
It's OK to store them in local variables; the SSA representation tends to keep them visible.
It's OK to store them in static final fields, too.
Static final fields can hold non-primitive, non-string constants.
The class of a static final field value is also constant, as is the array length (if it's an array).

Low-level safety checks

Null checks are cheap. They usually fold straight into a related memory access instruction, and use the CPU bus logic to catch nulls. (Deoptimization follows, with regenerated code containing an explicit check.)
User-written null checks are in most cases functionally identical to those inserted by the JVM.
Null checks can be hoisted manually, and suppress implicit null checks in dominated blocks.
Similar points can be made about other simple predicates, like class checks and range checks.
All such checks, whether implicit or manually written, are aggressively folded to constants.

Loops

The server compiler likes a loop with an int counter (int i = 0), a constant stride (i++), and loop-invariant limit (i <= n).
Loops over arrays work especially well when the compiler can relate the counter limit to the length of the array(s).
For long loops over arrays, the majority of iterations are free of individual range checks.
Loops are typically peeled by one iteration, to "shake out" tests which are loop invariant but execute only on a non-zero tripcount. Null checks are the key example.
If a loop contains a call, it is best if that call is inlined, so that loop can be optimized as a whole.
A loop can have multiple exits. Any deoptimization point counts as a loop exit.
If your loop has a rare exceptional condition, consider exiting to another (slower) loop when it happens.

Profiling

Profiling is performed at the bytecode level in the interpreter and tier one compiler. The compiler leans heavily on profile data to motivate optimistic optimizations.
Every null check site has a record of whether a null was ever seen.
Similar points can be made about other low-level checks.
Every call site with a receiver has a record of which types were encountered (up to 2-3 types).
There is also a type profile for every checkcast, instanceof, and aastore. (Helps with generics.)
Every call site and branch point has a record of execution counts.

Deoptimization

Deoptimization is the process of changing an optimized stack frame to an unoptimized one. With respect to compiled methods, it is also the process of throwing away code with invalid optimistic optimizations, and replacing it by less-optimized, more robust code. A method may in principle be deoptimized dozens of times.
The compiler may stub out an untaken branch and deoptimize if it is ever taken.
Similarly for low-level safety checks that have historically never failed.
If a call site or cast encounters an unexpected type, the compiler deoptimizes.
If a class is loaded that invalidates an earlier class hierarchy analysis, any affected method activations, in any thread, are forced to a safepoint and deoptimized.
Such indirect deoptimization is mediated by the dependency system. If the compiler makes an unchecked assumption, it must register a checkable dependency. (E.g., that class Foo has no subclasses, or method Foo.bar is has no overrides.)

Methods

Methods are often inlined. This increases the compiler's "horizon" of optimization.
Static, private, final, and/or "special" invocations are easy to inline.
Virtual (and interface) invocations are often demoted to "special" invocations, if the class hierarchy permits it. A dependency is registered in case further class loading spoils things.
Virtual (and interface) invocations with a lopsided type profile are compiled with an optimistic check in favor of the historically common type (or two types).
Depending on the profile, a failure of the optimistic check will either deoptimize or run through a (slow) vtable/itable call.

On the fast path of an optimistically typed call, inlining is common. The best case is a de facto monomorphic call which is inlined. Such calls, if back-to-back, will perform the receiver type check only once.
In the absence of strong profiling information, a virtual (or interface) call site will be compiled in an agnostic state, waiting for the first execution to provide a provisional monomorphic receiver. (This is called an "inline cache".)
An inline cache will flip to a monomorphic state at the first call, and stay in that state as long as the exact receiver type (not a subtype) is repeated every time.
An inline cache will flip to a "megamorphic" state if a second receiver type is encountered.
Megamorphic calls use assembly-coded vtable and itable stubs, patched in by the JVM. The compiler does not need to manage them.

Intrinsics

There are lots of intrinsic methods. See library_call.cpp and vmSymbols.hpp.
Object.getClass is one or two instructions.
Class.isInstance and Class.isAssignableFrom are as cheap as instanceof bytecodes when the operands are constants, and otherwise no more expensive than aastore type checks.
Most single-bit Class queries are cheap and even constant-foldable.
Reflective array creation is about as cheap as newarray or anewarray instructions.
Object.clone is cheap and shares code with Arrays.copyOf (only recently!).
...Need much more here...

Miscellanous

Use a disassembler if available to inspect the generated code.
Switches are profiled but the profile information is poorly used. For now, consider building an initial decision tree if you know one or two cases are really common.
Exception throwing compiles to a goto, if the thrower and catcher inline together. For such uses, rely on preallocated or cloned exceptions, or override the fillInStackTrace part of exception creation, which is an expensive, reflective native call.

Do not use jsr/ret. Just clone your finally code if you have to.
If you are compiling a non-Java language, consider using standard mangling conventions.
If you are generating almost the same class many times in a row, with small variations, factor out the repeating parts into a superclass or static helper class.
For small variations in the remaining part, consider using a single named class as a template and loading it multiple times as an anonymous class with constant pool edits. Anonymous classes load and unload faster than named ones.

Select a Garbage Collector
The choice of a garbage collection mode or static strategy does not in itself affect memory footprint noticeably, but choosing the right garbage collection strategy may allow you to reduce the heap size without a major performance degradation.

If your application uses a lot of temporary objects you should consider using a generational garbage collection strategy. The use of a nursery reduces fragmentation and thus allows for a smaller heap.

The concurrent garbage collector must start garbage collections before the heap is entirely full, to allow Java threads to continue allocating objects during the garbage collection. This means that the concurrent garbage collector requires a larger heap than the parallel garbage collector, and thus your primary choice for a small memory footprint is a parallel garbage collector.

The default garbage collection mode chooses between a generational parallel garbage collection strategy and a non-generational parallel garbage collection strategy, depending on the sizes of the objects that your application allocate. This means that the default garbage collector is a good choice when you want to minimize the memory footprint.

If you want to use a static garbage collection strategy, you can specify the strategy with the -Xgc command line option; for example:

java -Xgc:genpar myApplication





 










  
 














  



