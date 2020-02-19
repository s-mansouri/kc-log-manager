##clickhouse:
connect to it from a native client

```bash
docker run -it --network clickhouse_clickhouse-net --rm yandex/clickhouse-client --host clickhouse-1
```
create table for kafka engine

```mysql
CREATE TABLE accessLog (
    timestamp UInt64,
    level String,
    message String
  ) ENGINE = Kafka SETTINGS kafka_broker_list = '172.18.0.3:9092',
                            kafka_topic_list = 'accessLog',
                            kafka_group_name = 'nginx',
                            kafka_format = 'JSONEachRow',
                            kafka_row_delimiter = '\n',
                            kafka_skip_broken_messages = 1,
                            kafka_max_block_size = 1000;
                            kafka_num_consumers = 4;

```

##Kafka

- create topic:
```bash
kafka-topics --zookeeper zookeeper:2181 --topic myfirst --partitions 2 --replication-factor 1 --create
```

- Listing the number of Topics

```bash
kafka-topics --zookeeper zookeeper:2181 --list
```

- Describing a topic

```bash
kafka-topics --zookeeper zookeeper:2181 --describe --topic myfirst
```

- Deleting a topic

```bash
kafka-topics --zookeeper zookeeper:2181 --delete --topic mysecond
```

- Sending data to Kafka Topics
```bash
kafka-console-producer --broker-list 127.0.0.1:9092 --topic myfirst
```

- Reading whole messages
```bash
kafka-console-consumer --bootstrap-server 127.0.0.1:9092 --topic myfirst -from-beginning
```