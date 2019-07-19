# 实验目的
掌握Java API方式对Kafka实现String类型数据发送和接收。
# 实验原理
大数据的流处理分析中实时数据的发送和接收是基本操作，也是进行深入分析的基础，本实验完成Kafka消息生产者和消费者之间的消息发送和接收。Java API方式是更加常用的流数据写入和读取的方式，本实验就是学习通过Java编程实现对Kafka消息发送与接收。
# 实验步骤
在桌面中启动eclipse，创建工程kafkastringdemo, 编写和执行下面的代码

##### Producer数据生产者 Java代码
```
package com.inforstack.kafka.demo;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;
import java.util.Properties;

public class Producer extends Thread {
    private final Properties properties = new Properties();
    private org.apache.kafka.clients.producer.KafkaProducer<Integer, String> producer = null;
    private String topic;

    public Producer(String topic) {
        this.properties.put("bootstrap.servers", "localhost:9092");
        this.properties.put("acks", "all");
        this.properties.put("retries", 0);
        this.properties.put("batch.size", 16384);
        this.properties.put("linger.ms", 1);
        this.properties.put("buffer.memory", 33554432);
        this.properties.put("key.serializer","org.apache.kafka.common.serialization.IntegerSerializer");
        this.properties.put("value.serializer",
            "org.apache.kafka.common.serialization.StringSerializer");
        this.producer = new KafkaProducer<Integer, String>(this.properties);
        this.topic = topic;
    }

    @Override
    public void run() {
        try {
            //等待consumer
            sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        int messageNo = 1;
        Random r = new Random();
        String str1 = "Hello ShangHai!";
        String str2 = "I love ShangHai!";
        String messageStr = null;
        while (true) {
            Integer num = (r.nextInt(10) + 1);
			if (num % 2 != 0) {
				messageStr = new String(str1);
			} else {
				messageStr = new String(str2);
			}
			System.out.println("Send:" + messageStr);

            ProducerRecord<Integer, Integer> record = new ProducerRecord<Integer, String>(this.topic,
                    messageNo, messageStr);
            this.producer.send(record);
            messageNo++;

            if (messageNo > 10) {
                break;
            } else {
                try {
                    sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }

        this.producer.close();
    }
}

```

##### Consumer 消费者Java代码
```
package com.inforstack.kafka.demo;

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;

import java.util.Arrays;
import java.util.Properties;
import java.util.UUID;


public class Consumer extends Thread {
    private final Properties properties = new Properties();
    private org.apache.kafka.clients.consumer.KafkaConsumer<Integer, String> consumer =
        null;
    private String topic;

    public Consumer(String topic) {
        this.properties.put("bootstrap.servers", "localhost:9092");
        this.properties.put("group.id", UUID.randomUUID().toString());
        this.properties.put("enable.auto.commit", "true");
        this.properties.put("auto.commit.interval.ms", "1000");
        this.properties.put("session.timeout.ms", "30000");
        this.properties.put("key.deserializer",
            "org.apache.kafka.common.serialization.IntegerDeserializer");
        this.properties.put("value.deserializer",
            "org.apache.kafka.common.serialization.StringDeserializer");

        this.topic = topic;
        this.consumer = new KafkaConsumer<Integer, String>(this.properties);
        this.consumer.subscribe(Arrays.asList(this.topic));
    }

    @Override
    public void run() {
        int last = 0;
        String str1 = "Hello ShangHai!";
		String str2 = "I love ShangHai!";
		int countStr1 = 0;
		int countStr2 = 0;

        while (true) {
            try {
                ConsumerRecords<Integer, String> records = consumer.poll(100);

                if (records.isEmpty()) {
                    continue;
                } else {
                    for (ConsumerRecord<Integer, String> record : records) {
                        if (record.value().equals(str1)) {
							countStr1++;
						} else {
							countStr2++;
						}

						System.out.println("receive：" + record.value());
                        last++;
                    }
                }

                if (last >= 10) {
                    System.out.println(str1 + "的记录数为：" + countStr1 + "条");

					System.out.println(str2 + "的记录数为：" + countStr2 + "条");
                    break;
                }

                Thread.sleep(1);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
}

```
##### KafkaConsumerProducerDemo Java代码
```
package com.inforstack.kafka.demo;

public class App {
    public static void main(String[] args) {
        Producer producer = new Producer("test001");

        producer.start();

        Consumer consumer = new Consumer("test001");

        consumer.start();
    }
}
```

代码整理后目录截图如下:

![](/images/1-1_20180404065002.002.png)

# 执行方法
- 在APP.java文件上右键Run as -〉 Java Application，观察实验结果
