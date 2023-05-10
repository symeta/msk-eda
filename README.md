# msk-eda
## eda context review
### traditional sync request-response model pros & cons analysis
<img width="1505" alt="Screenshot 2023-05-10 at 10 41 40" src="https://github.com/symeta/msk-eda/assets/97269758/5a8b8713-9926-4ef2-ac44-1302c53f34b1">

### ASYNC message model:
- point-point model (queue/router)
- pub-sub model

### Even Bus:

<img width="284" alt="image" src="https://github.com/symeta/msk-eda/assets/97269758/a0228a03-9423-49c6-b5d4-6b56d78fda70">

## msk overview
### kafka application scenario
- 1

![image](https://github.com/symeta/msk-eda/assets/97269758/9a39c2f5-15da-4161-bab3-74dc9b811216)
- 2

![image](https://github.com/symeta/msk-eda/assets/97269758/99249337-f634-4514-92fa-241133d4a08f)

### msk managed scope

<img width="1083" alt="image" src="https://github.com/symeta/msk-eda/assets/97269758/228006c8-d43c-495b-a00f-e263d09c13e0">

### msk sizing
- [1. msk sizing tool](https://tiny.amazon.com/1cquo8z50)
- [2. msk sizing blog](https://aws.amazon.com/blogs/big-data/best-practices-for-right-sizing-your-apache-kafka-clusters-to-optimize-performance-and-cost/)

### msk configuration
#### 1. default.replication.factor 
     same with AZ quantity
#### 2. num.partitions 
<img width="1297" alt="Screenshot 2023-05-10 at 11 56 53" src="https://github.com/symeta/msk-eda/assets/97269758/f3630db5-3a79-451d-9186-8b2e35e599cf">

#### 3. delete.topic.enable
#### 4. retention.ms
This configuration controls the maximum time we will retain a log before we will discard old log segments to free up space if we are using the "delete" retention policy. This represents an SLA on how soon consumers must read their data. If set to -1, no time limit is applied.

- Type:	long
- Default:	604800000 (7 days)
- Valid Values:	[-1,...]
- Server Default Property:	log.retention.ms
- Importance:	medium

#### 5. other config items:
- auto.create.topics.enable=true
- min.insync.replicas=2
- num.io.threads=8
- num.network.threads=5
- num.replica.fetchers=2
- replica.lag.time.max.ms=30000
- socket.receive.buffer.bytes=102400
- socket.request.max.bytes=104857600
- socket.send.buffer.bytes=102400
- zookeeper.session.timeout.ms=18000

### msk producer sample code (python)
```shell
pip install kafka-pythonpip install kafka-python
```
```py
from kafka import KafkaProducer
def send_data(_kafka_topic, _producer):
    while True:
        data = get_random_record()
        partition_key = str(data["rowkey"])
        print(data)
        _producer.send(_kafka_topic, json.dumps(data).encode('utf-8'))

if __name__ == '__main__':
    producer = KafkaProducer(bootstrap_servers="<msk cluster broker list >")
    KAFKA_TOPIC = "<topic name>"
    send_data(KAFKA_TOPIC, producer)
```
#### producer acks
The default value is 1, which means as long as the producer receives an ack from the leader broker of that topic, it would take it as a successful commit and continue with the next message. It’s not recommended to set acks=0, because then you don’t get any guarantee on the commit. acks=all would make sure that the producer gets acks from all the in-sync replicas of this topic. It gives the strongest message durability, but it also takes long time which results in higher latency. So, you need to decide what is more important for you


### msk consumer sample code (pyspark)
```py
from pyspark.sql import SparkSession
spark = SparkSession \
        .builder \
        .appName("<app name>") \
        .config("spark.sql.debug.maxToStringFields", "100") \
        .getOrCreate()
kafka_df = spark.readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "<msk cluster broker list>") \
    .option("kafka.security.protocol", "SSL") \
    .option("failOnDataLoss", "false") \
    .option("subscribe", "topic1") \
    .option("includeHeaders", "true") \
    .option("startingOffsets", "latest") \
    .option("spark.streaming.kafka.maxRatePerPartition", "50") \
    .load()
```

## useful resources:
- [1. msk best practices](https://docs.aws.amazon.com/msk/latest/developerguide/bestpractices.html)
- [2. best practices of msk provisioning](https://www.youtube.com/watch?v=4C_FT2Ie9E4)
- [3. ]
