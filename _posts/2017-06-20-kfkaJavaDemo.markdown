---
layout:     post
title:      "kfka 2.12 Java 连接Demo"
date:       UTC 2017-06-20 17:30:00
author:     "Pearpai"
header-img: "img/tag-bg.jpg"
catalog: true
tags:
    - kafka
    - Blog
    - Java
    - 中间件
---
## pom.xml
```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka_2.12</artifactId>
    <version>0.10.2.1</version>
</dependency>
```
##  生产者 TestProducer
```java
package com.action.kafkademo;

import java.util.Properties;
import java.util.Random;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.clients.producer.ProducerRecord;


/**
 * partitioner 生产者
 * Created by wuyunfeng on 2017/6/20.
 */
public class TestProducer {
    public static void main(String[] args) {
        long events = Long.parseLong(args[0]);
        Random rnd = new Random();

        Properties props = new Properties();
        props.put("bootstrap.servers", "127.0.0.1:9092,127.0.0.1:9093,127.0.0.1:9094");
        props.put("acks", "all");
        props.put("retries", 0);
        props.put("batch.size", 16384);
        props.put("linger.ms", 1);
        props.put("buffer.memory", 33554432);
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        // 进行partition 控制
        props.put("partitioner.class", "com.action.kafkademo.SimplePartitioner");

        Producer<String, String> producer = new KafkaProducer<>(props);

        for (long nEvents = 0; nEvents < events; nEvents++) {
            String key = String.valueOf(rnd.nextInt(1000));
            String value = "content ..." ;
            String topic = "my-visit";
            ProducerRecord<String, String> data = new ProducerRecord<>(topic, key, value);
            producer.send(data,
                    (metadata, e) -> {
                        if (e != null) {
                            e.printStackTrace();
                        } else {
                            System.out.println(metadata.toString());
                        }
                    });
        }
        producer.close();
    }
}
```
## Partition控制 SimplePartitioner
```java
package com.action.kafkademo;

import java.util.List;
import java.util.Map;

import org.apache.kafka.clients.producer.Partitioner;
import org.apache.kafka.common.Cluster;
import org.apache.kafka.common.PartitionInfo;

/**
 * partitioner 控制
 * Created by wuyunfeng on 2017/6/20.
 */
public class SimplePartitioner implements Partitioner {

    @Override
    public int partition(String topic, Object key, byte[] keyBytes,
                         Object value, byte[] valueBytes, Cluster cluster) {
        int partition = 0;
        List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
        int numPartitions = partitions.size();
        int key1 = Integer.valueOf((String) key);
        if (key1 > 0) {
            partition = key1 % numPartitions;
        }

        return partition;
    }

    @Override
    public void close() {

    }


    @Override
    public void configure(Map<String, ?> configs) {

    }
}
```
## 消费者 TestConsumer
```java
package com.action.kafkademo;

import java.util.Arrays;
import java.util.Properties;

import org.apache.kafka.clients.consumer.Consumer;
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.serialization.StringDeserializer;


/**
 * 消费者
 * Created by wuyunfeng on 2017/6/20.
 */
public class TestConsumer {

    public static void main(String[] args) {
        Properties props = new Properties();

        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "127.0.0.1:9092,127.0.0.1:9093,127.0.0.1:9094");
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "test");
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true");
        props.put(ConsumerConfig.AUTO_COMMIT_INTERVAL_MS_CONFIG, "1000");
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);

        Consumer<String, String> consumer = new KafkaConsumer<>(props);
        consumer.subscribe(Arrays.asList("my-visit"));

        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(100);
            for (ConsumerRecord<String, String> record : records) {
                System.out.println(record.toString());
            }
        }
    }
}
```
