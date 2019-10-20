
# Apache Spark best practice

1+ year of using spark

  

Why Spark for Big Data Computing?

It's ez to use, convenient, large community. v.v. support Stateful Streaming, Batch Processing, Machine Learning enablement.

  

### Chapter I: From ground up

1. Setting up environment

* For clustering, just search Hortonworks opensource, setting thing up using Ambari.

* For local, testing: just come straight to 1st application.

  

2. Build your first application

First application for Word Count, suppose you have data.txt in your classpath, the application, try map every line into Map [words, number_instances], and reduce by words, print answer on console.

  

```java

//used spark 2.3.2 from spark.apache.org

SparkConf conf = new SparkConf().setAppName("your-first-name").setMaster("local[3]");

JavaSparkContext sc = new JavaSparkContext(conf);

JavaRDD<String> txtFile = sc.textFile("data.txt");

Map<String, Long> ans = txtFile

.mapPartitionsToPair((PairFlatMapFunction<Iterator<String>, String, Long>) stringIterator -> {

Map<String, Long> tokenMap = new HashMap<>();

while (stringIterator.hasNext()){

String s = stringIterator.next();

if (s != null && s.length() > 0) {

String tokens[] = s.split(" ");

for (String token : tokens) {

long initial = tokenMap.getOrDefault(token, 0L);

tokenMap.put(token, initial + 1L);

}

}

}

List<Tuple2<String, Long>> ret = new ArrayList<>();

for (Map.Entry<String, Long> entry : tokenMap.entrySet()) {

ret.add(new Tuple2<>(entry.getKey(), entry.getValue()));

}

return ret.iterator();

})

.reduceByKey((Function2<Long, Long, Long>) (aLong, aLong2) -> aLong + aLong2)

.collectAsMap();

  

for (Map.Entry<String, Long> entry : ans.entrySet()) {

System.out.println(entry.getKey() + " " + entry.getValue());

}

  

sc.close();

```

3. Transformation and Action in spark

There are 2 concepts in Apache Spark: transformation and action

* [Transformation] (https://spark.apache.org/docs/latest/rdd-programming-guide.html#transformations)

Actually, RDD is Scala-driven collection, and transformation function of RDD is really similar to Scala collection.

Listing some basic transformation you can use:

  

| Transformation | Meaning |

|---------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------|

| map(func) | Return a new distributed dataset formed by passing each element of the source through a function func. |

| filter(func) | Return a new dataset formed by selecting those elements of the source on which funcreturns true. |

| flatMap(func) | Similar to map, but each input item can be mapped to 0 or more output items (so funcshould return a Seq rather than a single item). |

| mapPartitions(func) | Similar to map, but runs separately on each partition (block) of the RDD, so func must be of type Iterator => Iterator when running on an RDD of type T. |

  

* [Action] (https://spark.apache.org/docs/latest/rdd-programming-guide.html#actions)

  

What cause shuffle, actual computation.

  

4. Spark Batch vs Spark Streaming

  

| Criteria | Batch | Streaming |

|----------|-------|-----------|

|Resouces|Batch claims resource for limited times|Streaming take resource forever|

|Optimization|Batch does not need to worry too much in optimization, because it might be done|Really serious about optimization (computation time, how many resource is OK ...)|

  

* From Batching to Streaming

  

When you are familiar with your batching coding in Spark, move on, try streaming:

- Try first problem: most popular hashtag in newfeeds in past 30 minutes.

Try not using Window and using your own updateStateByKey, so you can manage what you do in State, and not :D

  

```java

final JavaStreamingContext streamingContext = new JavaStreamingContext(sparkConf, Durations.seconds(5));

streamingContext.checkpoint("checkpoint_dir/");

  

// so 2 minutes -> 2 * 60 / 5 = 24 mini-batch

JavaReceiverInputDStream<String> lines = streamingContext.socketTextStream("localhost", 9999);

//...

public class Counter {

// check if batchPass over 24 -> remove

private int batchPass;

// counter in batch

private long count;

}

// when DStream#updateStateByKey -> do something ...

```

  

* Data preparation

  
  

* Spark streaming processing

  

* Visualize your answer in spark master nodes.

  

If you running at localhost -> go to this url:

```

19/01/06 20:02:41 INFO SparkUI: Bound SparkUI to 0.0.0.0, and started at http://127.0.0.1:4040

```

  

* Convert Streaming into Batching

When you do not real-time processing of your data, in that case, you might consider converting streaming into batching by dumping your data real-time into HDFS file in your favourite codec (AVRO, ProtoBuf, JSON, v.v.), and further use your batching technique to get thing done.

  

### Chapter II: Take off

1. Running on cluster
My experience only worked with YARN, so I will write my practice for it.
I'm not that good for deployment, but for performance measurement I knew some. Maybe it's already fixed in current release of Spark.
* Medium cluster
With medium cluster (16 Server, 256GB RAM, Intel Xeon 4? ).
  

* Larger cluster
When running on large cluster, Network latency is crazy, and also why we need larger cluster, we probably deal with very large amount of data
Problem happens with stateful streaming job, and the key is that we need to have least data when checkpoint, cos it will take a lot of time.

2. Checkpoint
Normally one batch will be 5 seconds, so after maybe 6 batches we can have checkpoint. Don't take it too short, cos it will cause extra time on checkpoint task.
Don't take it too long, cos when run in fail state, need to run from checkpoint will make performance worse.
  

3. Fault tolerant
We can have a task similar to checkpoint, do the checkpoint for stateful and use stateful for loading when booting.
  

4. Tip and trick:

* Partition transform is more efficient.

  

### Chapter III: Best practice on:

1. Broadcast
Broadcast the metadata, and shouldn't be changing.

2. Tuning application
Tuning Application is depending on task you working on, if you run a batch job, there's some point you do not need to tune more to run faster, cos it can be time-consuming, and not too much effective.

3. Debugging
Debugging: when debugging should look at the graph in management UI, look at DAG, look at the time running each task.
You should register extra accumulator (actually we always create 1 class, contain all the accumulator, and then register 1 time) for debugging: to see the relation between metrics you use, to make sure that process is producing right value.
4. Testing
Testing is headache for BigData, cos creating environment like production for development is hard, cos data is changing overtime.
