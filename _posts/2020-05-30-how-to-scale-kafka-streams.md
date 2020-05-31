---
layout: post
title: How to Effectively Scale a Kafka Streams Application
subtitle: A 
tags: ["kstreams", "kafka"]
keywords: kstreams, kafka 
comments: true
#gh-repo: daattali/beautiful-jekyll
#gh-badge: [star, fork, follow]
#cover-img: /assets/img/path.jpg
---

Great news, kstream applications are very easy to scale out of the box! That is of course until things turn sour and you're left scurrying to figure out why one partition is continuously lagging. When that happens you'll be thankful for understanding what is happening behind the scenes of this elegant api. That is the knowledge I'm going to attempt to give you.

The first step in understanding kstreams, is understanding kafka itself. I'm going to assume you already have basic kafka knowledge. If not, I'd recommend you first read [Kafka In A Nutshell](https://sookocheff.com/post/kafka/kafka-in-a-nutshell)

Let's start by defining what a topology is.

### The Streams Topology

A topology is the architectural schema of your kstreams application in which each message flows through to be processed. The composition of the most basic topology is one source topic, one processor and one output topic. That is a boring topology though and doesn't describe how the stream can scale, so let's look at a slightly more complex topology. Let's add a `groupBy { .. }` clause that will cause a repartition where we can see the parallelism potentials of kstreams.

During initialisation of your kstreams application, you will be able to call a describe function that will output your topology like so:

```kotlin
val sBuilder = StreamsBuilder()

sBuilder.stream("sourceTopic", Consumed.with(StringSerde(), ValueSerde()))
                .groupBy { key, value ->  KeyValue(value.newKey, value) }
                .reduce { _, value2 -> value2 }
                .toStream()
                .to("targetTopic")

val topology = sBuilder.build()
logger.info(topology.describe())
```

Which will log this:
```
Sub-topology: 0
    Source: KSTREAM-SOURCE-0000000000 (topics: [sourceTopic])
      --> KSTREAM-KEY-SELECT-0000000001
    Processor: KSTREAM-KEY-SELECT-0000000001 (stores: [])
      --> KSTREAM-FILTER-0000000005
      <-- KSTREAM-SOURCE-0000000000
    Processor: KSTREAM-FILTER-0000000005 (stores: [])
      --> KSTREAM-SINK-0000000004
      <-- KSTREAM-KEY-SELECT-0000000001
    Sink: KSTREAM-SINK-0000000004 (topic: KSTREAM-AGGREGATE-STATE-STORE-0000000002-repartition)
      <-- KSTREAM-FILTER-0000000005

  Sub-topology: 1
    Source: KSTREAM-SOURCE-0000000006 (topics: [KSTREAM-AGGREGATE-STATE-STORE-0000000002-repartition])
      --> KSTREAM-AGGREGATE-0000000003
    Processor: KSTREAM-AGGREGATE-0000000003 (stores: [KSTREAM-AGGREGATE-STATE-STORE-0000000002])
      --> KTABLE-TOSTREAM-0000000007
      <-- KSTREAM-SOURCE-0000000006
    Processor: KTABLE-TOSTREAM-0000000007 (stores: [])
      --> KSTREAM-SINK-0000000008
      <-- KSTREAM-AGGREGATE-0000000003
    Sink: KSTREAM-SINK-0000000008 (topic: targetTopic)
      <-- KTABLE-TOSTREAM-0000000007
```

You can copy that into this really useful [topology visualisation tool](https://zz85.github.io/kafka-streams-viz/) which will output this image:

![Topology](/assets/img/posts/how-to-scale-kafka-streams-effectively/stream-example.png)


### Topology Execution and Parallelism

Let's go over how the topology is executed in your server.

Source topic partitions are what define the maximum level of parallelism that the application will be capable of. One source partition equates to one task that can be run in parallel. A source topic can be from external or it can be the result of a repartition like we've seen in our example. So if we have one partition on our topic `source` we will be required to match that with one partition on the `repartition` topic adding up to two tasks that can be executed in parallel in total.

For each source partition in the topology, there will be one stream task that runs in a [kstream thread](https://docs.confluent.io/current/streams/architecture.html#streams-architecture-threads). It's not required to have one thread per task as one thread can handle multiple, but to get the most performance out of the application, we can dedicate one thread to each task to prevent streams creating contention amongst tasks to be processed. Figuring out how many tasks the topology requires is easy. It is the sum of all source partitions. This get's a little bit more clever when a topology has a repartition from, for example, of a `groupBy { .. }` call. The output of the grouping will be materialised back to the broker and consumed by a task in potentially an independent thread, possibly even a different server. The topology is then comprised of two sub topologies.

You can confirm this by looking at the log output when starting your application

```
[2020-05-04 16:45:26,859] INFO (org.apache.kafka.streams.processor.internals.StreamThread:351) stream-thread [entities-eb9c0arb-ecad-4rg1-b4e8-715dcf2afee3-StreamThread-1] partition assignment took 100 ms.
current active tasks: [0_0, 0_2, 1_2, 2_2]
current standby tasks: []
previous active tasks: []
```
In this example we have 1 thread. The current active tasks array has 4 tasks in it, which are all executed on the same thread. The first number indicates the sub-topology number (we have two in our example) and the partition of each topic (again, there are 2 in this example)


TODO * A kstream thread does not exactly boil down to one java thread, but a few of them. **

### Partitioning

Choosing a good key to partition by is absolutely crucial in being able to operate at scale. This is of course not unique to streams, but it is so fundamental to streaming that we need to cover it now before going further. It is absolutely crucial to get partitioning right or you risk drastically reducing the overall throughput your application can handle.

Partitioning in kafka is a simple (hash of key) modulo (partition count) operation.

### Task Allocation

The [StreamsPartitionAssignor](https://github.com/apache/kafka/blob/trunk/streams/src/main/java/org/apache/kafka/streams/processor/internals/StreamsPartitionAssignor.java) is responsible for divvying up the tasks amongst all the kstream threads in a given consumer group. It is important to understand this class and how this streams specific implementation differs from standard consumer partition allocation algorithms, as it has a direct impact on how you can scale your service. It's fantastic in keeping an even task count across all threads, but it does have some short comings you'll need to keep in mind.

Imagine you have a poorly distributed partition load. This is very possible if you partition by a poor key.

### Repartitioning

### Knowing Kstreams Limits



