---
title: "Use It or Lose It Notes"
date: 2025-07-27
draft: false
tags: ["spark", "java", "snippets", "use-it-or-lose-it",]
Categories: [tech, living]
---

> Because I mostly don't use it, and then end up losing it.

This is my living blog of quick, forgotten patterns. Not profound, just practical.

---
## Table of Contents

- [Spark](#spark)

- [Java](#java)
## Spark

### 1. Create a SparkSession (Boilerplate I always forget)

```java
SparkSession spark = SparkSession.builder()
        .appName("UILI")
        .master("local[*]")
        .getOrCreate();
```

---

### 2. Create a Dataset from Strings (not from files)

```java
Dataset<String> ds = spark.createDataset(
        Arrays.asList("Abc", "xyz"),
        Encoders.STRING()
);
```

---

### 3. SparkConf and RDD creation options

```java
SparkConf conf = new SparkConf().setAppName("uili").setMaster("local[*]");
JavaSparkContext sc = new JavaSparkContext(conf);

// Create RDDs from:
sc.parallelize(...);
sc.textFile("path");
sc.wholeTextFiles("path");

// Or convert from DataFrame:
df.javaRDD();

// Or from Kafka:
JavaInputDStream<String> lines = KafkaUtils.createStream(...);
lines.foreachRDD(rdd -> { /* process rdd */ });
```

---

### 4. Word Count with Stopword Filtering

```java
                lines
                .flatMap(s ->  Arrays.asList(SPACE.split(s)).iterator())
                .filter( word -> !word.isEmpty() && !stopWords.contains(word))
                .mapToPair(word -> new Tuple2<>(word, 1))
                .reduceByKey(Integer::sum)
                .mapToPair(Tuple2::swap)
                .sortByKey(false)
                .take(10)
                .forEach( tuple -> { System.out.println(tuple._1() + ": " + tuple._2());});
```

---

### 5. Custom Partitioner


```java
rdd.partitionBy(new UserIdModPartitioner(10));

public static class UserIdModPartitioner extends Partitioner {
    private final int numPartitions;

    public UserIdModPartitioner(int numPartitions) {
        this.numPartitions = numPartitions;
    }

    @Override
    public int numPartitions() {
        return numPartitions;
    }

    @Override
    public int getPartition(Object key) {
        Integer userId = (Integer) key;
        return userId % numPartitions;
    }

    @Override
    public boolean equals(Object obj) {
        if (obj instanceof UserIdModPartitioner) {
            return ((UserIdModPartitioner) obj).numPartitions == this.numPartitions;
        }
        return false;
    }
}
```

---

### 6. groupByKey vs reduceByKey vs mapValues

- `groupByKey()` → Avoid for large data. Shuffles everything.
- `reduceByKey()` → Preferred. Does local aggregation before shuffle.
- `mapValues()` → Lightweight transform on values only (no shuffle).

```java
rdd.mapValues(value -> transform(value));
```

---

## Java

> Random nuggets that fade away from memory faster than they should.


---

