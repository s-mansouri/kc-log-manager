#KCL
Store Nginx log to Clickhouse, through Kafka
##Clickhouse:
connect to Clickhouse from a native client

```bash
docker run -it --network clickhouse_clickhouse-net --rm yandex/clickhouse-client --host clickhouse-1
```
create table for kafka engine

```mysql
CREATE TABLE accesslog (
    remote_addr String,
    remote_user String,
    time_local String,
    request String,
    status Int32,
    body_bytes_sent Int64,
    http_referer String,
    http_user_agent String
    ) ENGINE = Kafka SETTINGS kafka_broker_list = 'kafka:19092',
                            kafka_topic_list = 'accesslog2',
                            kafka_group_name = 'nginx',
                            kafka_format = 'JSONEachRow',
                            kafka_row_delimiter = '\n',
                            kafka_skip_broken_messages = 1,
                            kafka_max_block_size = 1000,
                            kafka_num_consumers = 4;

CREATE TABLE saved_accesslog (
    remote_addr String,
    remote_user String,
    time_local String,
    date Date,
    request String,
    status Int32,
    body_bytes_sent Int64,
    http_referer String,
    http_user_agent String
    ) ENGINE = MergeTree(date, (status, date), 8192);

CREATE MATERIALIZED VIEW log_consumer TO saved_accesslog
    AS SELECT *
    FROM accesslog;
```

sample data
```json
{"remote_addr": "172.18.0.1",
"remote_user": "-",
"time_local":  "20/Feb/2020:10:45:19 +0000",
"request":     "GET / HTTP/1.1",
"status":      "200",
"body_bytes_sent": "612",
"http_referer": "-",
"http_user_agent": "curl/7.52.1"
}
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

#kafkacat
- read log from file
```bash
docker run --rm -i --volume /srv/nginx/nginx/log:/var/log/nginx --network kcl_kcl-net mirror.imbco.ir:4000/confluentinc/cp-kafkacat:5.4.0 kafkacat -b kafka:19092 -t accesslog2 -P -l /var/log/nginx/access.log
```