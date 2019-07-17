# 实验目的
掌握Java API方式对Kafka实现数据发送和接收
# 实验原理
大数据的流处理分析中实时数据的发送和接收是基本操作，也是进行深入分析的基础，本实验完成Kafka消息生产者和消费者之间的消息发送和接收。Java API方式是更加常用的流数据写入和读取的方式，本实验就是学习通过Java编程实现对
# 实验步骤
在容器中启动Java代码编辑器，创建工程编写和执行下面的代码。

在容器中启动Java代码编辑器，创建工程编写和执行下面的代码。

Producer数据生产者 Java代码
```
package com.inforstack.kafka.demo;

import java.util.Properties;

import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerRecord;

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
		this.properties.put("key.serializer", "org.apache.kafka.common.serialization.IntegerSerializer");
		this.properties.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
		
		this.producer = new KafkaProducer<Integer, String>(this.properties);
		
		this.topic = topic;  
	}

	@Override
	public void run() {
		int messageNo = 1;  
		while (true) {
			String messageStr = new String("Message_" + messageNo);  
			System.out.println("Send:" + messageStr);  
			
			ProducerRecord<Integer, String> record = new ProducerRecord<Integer, String>(this.topic, messageNo, messageStr);
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
Consumer 消费者Java代码
```
package com.inforstack.kafka.demo;

import java.util.Arrays;
import java.util.Properties;
import java.util.UUID;

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;

public class Consumer extends Thread {
	
	private final Properties properties = new Properties();  
	
	private org.apache.kafka.clients.consumer.KafkaConsumer<Integer, String> consumer = null;
	
	private String topic;
	
	public Consumer(String topic) {
		this.properties.put("bootstrap.servers", "localhost:9092");
		this.properties.put("group.id", UUID.randomUUID().toString());
		this.properties.put("enable.auto.commit", "true");
		this.properties.put("auto.commit.interval.ms", "1000");
		this.properties.put("session.timeout.ms", "30000");
		this.properties.put("key.deserializer", "org.apache.kafka.common.serialization.IntegerDeserializer");
		this.properties.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
		this.topic = topic;
		
		this.consumer = new KafkaConsumer<Integer, String>(this.properties);
		this.consumer.subscribe(Arrays.asList(this.topic));	  
	}
	
	@Override
	public void run() {		
		int last = 0;
		while (true) {
			try {
				ConsumerRecords<Integer,String> records = consumer.poll(100);			
				
				if (records.isEmpty()) {
					continue;
				} else {
					for (ConsumerRecord<Integer, String> record : records) {
						System.out.println("receive：" + record.value());  
						last = record.key();
					}
				}
				if (last >= 10) {
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

KafkaConsumerProducerDemo Java代码
```
package com.inforstack.kafka.demo;

public class App 
{
    public static void main( String[] args )
    {
        Producer producer = new Producer("test");
        producer.start();
        
        Consumer consumer = new Consumer("test");
        consumer.start();
    }
}
```

代码整理后目录截图如下:
![](https://kfcoding-static.oss-cn-hangzhou.aliyuncs.com/gitcourse-bigdata/2-5_20171204045636.036.png)

执行方式：
```
cd ~/exercise/exercise_3
java -jar kafkademo-0.0.1-SNAPSHOT.jar
```
执行结果：
![](https://kfcoding-static.oss-cn-hangzhou.aliyuncs.com/gitcourse-bigdata/2-6_20171204045903.003.png)