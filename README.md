# Apache Spark best practice
My experience for one year of using spark

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
* Distinguish between map, mapPartition, other transformation
* What cause shuffle, actual computation.

3. Spark Batch vs Spark Streaming

| Criteria | Batch | Streaming |
|----------|-------|-----------|
|Resouces|Batch claims resource for limited times|Streaming take resource forever|
|Optimization|Batch does not need to worry too much in optimization, because it might be done|Really serious about optimization (computation time, how many resource is OK ...)|

* From Batching to Streaming
When you are familiar with your batching coding in Spark, move on, try streaming:
Try first problem: most popular hashtag in newfeeds in past 30 minutes.
* Data preparation
* Spark streaming processing
* Visualize your answer in spark master nodes.


* Convert Streaming into Batching
When you do not real-time processing of your data, in that case, you might consider converting streaming into batching by dumping your data real-time into HDFS file in your favourite codec (AVRO, ProtoBuf, JSON, v.v.), and further use your batching technique to get thing done.

### Chapter II: Take off
1. Running on cluster
* Medium cluster

* Larger cluster

* Comparation between themem

2. Checkpoint

3. Fault tolerant

4. Tip and trick:
* Partition transform is more efficient.

### Chapter III: Best practice on:
1. Broadcast
2. Tuning application
3. Debugging
