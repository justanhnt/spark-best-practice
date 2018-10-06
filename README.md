# spark-best-practice
My experience for one year of using spark

### Chapter I: From ground up
1. Setting up environment
* For clustering, just search Hortonworks opensource, setting thing up using Ambari.
* For local, testing: just come straight to 1st application.

2. Build your first application
```java
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
