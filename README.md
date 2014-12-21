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

