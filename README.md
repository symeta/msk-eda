# msk-eda
## 1. eda context review
### 1.1 traditional sync request-response model pros & cons analysis

<img width="889" alt="image" src="https://github.com/symeta/msk-eda/assets/97269758/462a7805-b182-4223-9008-fb38aac8fbdb">

### 1.2 ASYNC message model:
- point-point model (queue/router)
- pub-sub model

### 1.3 Even Bus:

<img width="284" alt="image" src="https://github.com/symeta/msk-eda/assets/97269758/a0228a03-9423-49c6-b5d4-6b56d78fda70">

## 2. msk overview
### 2.1 kafka application scenario
- 1. Distributed Message Cache

![image](https://github.com/symeta/msk-eda/assets/97269758/9a39c2f5-15da-4161-bab3-74dc9b811216)

- 2. Distributed Event Bus

![image](https://github.com/symeta/msk-eda/assets/97269758/99249337-f634-4514-92fa-241133d4a08f)

### 2.2 msk managed scope

<img width="1083" alt="image" src="https://github.com/symeta/msk-eda/assets/97269758/228006c8-d43c-495b-a00f-e263d09c13e0">

### 2.3 msk sizing
- [1. msk sizing tool]([msk_sizing.xlsx](https://github.com/symeta/msk-eda/files/11449744/msk_sizing.xlsx)
)
- [2. msk sizing blog](https://aws.amazon.com/blogs/big-data/best-practices-for-right-sizing-your-apache-kafka-clusters-to-optimize-performance-and-cost/)

### 2.4 msk configuration
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

### 2.5 msk producer sample code (python)
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


### 2.6 msk consumer sample code (pyspark)
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
- [details for configing managing offsets](https://spark.apache.org/docs/2.2.0/structured-streaming-kafka-integration.html)
- [kafka streams exactly once message consumption](https://www.confluent.io/blog/enabling-exactly-once-kafka-streams/)

### 2.7 msk scaling
- ***vertical scaling***: manually upgrade broker instance type, or simply add storage
- ***horizontal scaling***: manually add broker into cluster
- [msk update broker count](https://docs.aws.amazon.com/msk/latest/developerguide/msk-update-broker-count.html)
- [kafka cluster expansion ops](https://kafka.apache.org/documentation/#basic_ops_cluster_expansion)

### 2.8 Kafka Connect
- [msk managed connectors](https://aws.amazon.com/blogs/aws/introducing-amazon-msk-connect-stream-data-to-and-from-your-apache-kafka-clusters-using-managed-connectors/)

### 2.9 msk monotoring
<img width="1069" alt="image" src="https://github.com/symeta/msk-eda/assets/97269758/d4d8f099-7eac-4d30-a041-d0e61e8cd195">

- [msk developerguide monitoring](https://docs.aws.amazon.com/msk/latest/developerguide/monitoring.html)
- [msk developerguide logging](https://docs.aws.amazon.com/msk/latest/developerguide/msk-logging.html)

### 2.10 msk cli
msk cluster broker list sample:
```sh
<broker1 endpoint>:9092,<broker2 endpoint>:9092,<broker3 endpoint>:9092
```
cli sample:
```sh
./kafka-topics.sh --bootstrap-server <msk cluster broker list> —list

./kafka-console-consumer.sh --bootstrap-server <msk cluster broker list> --topic <topic name> from beginning

./kafka-console-consumer.sh --bootstrap-server <msk cluster broker list> --topic <topic name>

./kafka-topics.sh —bootstrap-server <msk cluster broker list>  —create —topic <topic name> —partitions 3 —replication-factor 2
```
- [cli details](https://docs.confluent.io/kafka/operations-tools/kafka-tools.html)

### 2.11 msk tier storage
this feature is to offer the possibility that all kafka cluster data is persistent with optimized storage. version 2.8.2 and above available.
- [msk tiered storage](https://docs.aws.amazon.com/msk/latest/developerguide/msk-tiered-storage.html)


## 3. useful resources:
- [1. msk best practices](https://docs.aws.amazon.com/msk/latest/developerguide/bestpractices.html)
- [2. best practices of msk provisioning](https://www.youtube.com/watch?v=4C_FT2Ie9E4)
- [3. msk learning resources](https://aws.amazon.com/msk/resources/)
- [4. msk hands on workshop](https://catalog.us-east-1.prod.workshops.aws/workshops/c2b72b6f-666b-4596-b8bc-bafa5dcca741/en-US)

