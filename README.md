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





  



