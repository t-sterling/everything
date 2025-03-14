# 🔥 Kafka Configuration Parameters Cheat Sheet

This cheat sheet provides **essential Kafka configuration parameters** for **brokers, producers, consumers, and Kafka Streams**, including performance tuning options.

📌 **Official Kafka Configuration Docs**: [Kafka Configs](https://kafka.apache.org/documentation/#configuration)  

---

## **1. Kafka Broker Configuration**  

| Parameter | Description | Default |
|-----------|------------|---------|
| `broker.id` | Unique ID for the broker. | None |
| `log.dirs` | Directory where logs (topics) are stored. | `/tmp/kafka-logs` |
| `num.partitions` | Default number of partitions for a topic. | 1 |
| `zookeeper.connect` | Zookeeper connection string. | None |
| `default.replication.factor` | Default number of replicas per partition. | 1 |
| `log.retention.hours` | Duration (hours) before logs are deleted. | 168 (7 days) |
| `log.segment.bytes` | Size of a log segment before rolling. | 1GB |
| `auto.create.topics.enable` | Allows automatic topic creation. | `true` |
| `delete.topic.enable` | Allows topic deletion. | `false` |
| `log.cleaner.enable` | Enables log compaction. | `false` |

```properties
# Example broker configuration
broker.id=1
log.dirs=/var/lib/kafka/logs
num.partitions=3
default.replication.factor=2
zookeeper.connect=localhost:2181
```

🔗 **More on Broker Configs**: [Kafka Broker Configs](https://kafka.apache.org/documentation/#brokerconfigs)  

---

## **2. Kafka Producer Configuration**  

| Parameter | Description | Default |
|-----------|------------|---------|
| `bootstrap.servers` | List of Kafka brokers to connect to. | None |
| `acks` | Message acknowledgment mode (`all`, `1`, `0`). | `1` |
| `retries` | Number of retries before failing. | `2147483647` (infinite) |
| `batch.size` | Max batch size before sending messages. | `16384` |
| `linger.ms` | Time to wait before sending a batch. | `0` |
| `compression.type` | Compression algorithm (`none`, `gzip`, `snappy`, `lz4`). | `none` |
| `enable.idempotence` | Enables exactly-once semantics. | `false` |
| `transactional.id` | Enables transactional messaging. | None |

```properties
# Example producer configuration
bootstrap.servers=localhost:9092
acks=all
retries=3
batch.size=32768
linger.ms=5
compression.type=lz4
enable.idempotence=true
transactional.id=my-transaction-id
```

🔗 **More on Producer Configs**: [Kafka Producer Configs](https://kafka.apache.org/documentation/#producerconfigs)  

---

## **3. Kafka Consumer Configuration**  

| Parameter | Description | Default |
|-----------|------------|---------|
| `bootstrap.servers` | Kafka brokers to connect to. | None |
| `group.id` | Consumer group ID. | None |
| `enable.auto.commit` | Automatically commit offsets. | `true` |
| `auto.commit.interval.ms` | Offset commit interval. | `5000` |
| `auto.offset.reset` | Behavior when no offset is found (`earliest`, `latest`). | `latest` |
| `fetch.min.bytes` | Minimum data fetched per request. | `1` |
| `max.poll.records` | Max records returned per fetch. | `500` |

```properties
# Example consumer configuration
bootstrap.servers=localhost:9092
group.id=my-consumer-group
enable.auto.commit=false
auto.offset.reset=earliest
max.poll.records=1000
fetch.min.bytes=1024
```

🔗 **More on Consumer Configs**: [Kafka Consumer Configs](https://kafka.apache.org/documentation/#consumerconfigs)  

---

## **4. Kafka Streams Configuration**  

| Parameter | Description | Default |
|-----------|------------|---------|
| `application.id` | Unique ID for the Kafka Streams application. | None |
| `bootstrap.servers` | Kafka brokers to connect to. | None |
| `num.stream.threads` | Number of processing threads. | `1` |
| `commit.interval.ms` | Interval for committing processed records. | `30000` |
| `cache.max.bytes.buffering` | Cache size for stateful operations. | `10485760` |
| `processing.guarantee` | Defines processing mode (`at_least_once`, `exactly_once`). | `at_least_once` |

```properties
# Example Kafka Streams configuration
application.id=my-streams-app
bootstrap.servers=localhost:9092
num.stream.threads=2
commit.interval.ms=10000
cache.max.bytes.buffering=0
processing.guarantee=exactly_once
```

🔗 **More on Kafka Streams Configs**: [Kafka Streams Configs](https://kafka.apache.org/documentation/#streamsconfigs)  

---

## **5. Kafka Security Configuration**  

| Parameter | Description | Default |
|-----------|------------|---------|
| `ssl.keystore.location` | Path to SSL keystore file. | None |
| `ssl.keystore.password` | Keystore password. | None |
| `ssl.truststore.location` | Path to truststore file. | None |
| `ssl.truststore.password` | Truststore password. | None |
| `sasl.mechanism` | SASL mechanism (`PLAIN`, `SCRAM-SHA-256`, `GSSAPI`). | None |
| `security.protocol` | Security protocol (`SSL`, `SASL_PLAINTEXT`). | `PLAINTEXT` |

```properties
# Example SSL configuration
security.protocol=SSL
ssl.keystore.location=/etc/kafka/client.keystore.jks
ssl.keystore.password=my-secret-password
ssl.truststore.location=/etc/kafka/client.truststore.jks
ssl.truststore.password=my-secret-password
```

🔗 **More on Kafka Security**: [Kafka Security Guide](https://kafka.apache.org/documentation/#security)  

---

## **6. Kafka Performance Tuning**  

| Optimization | Recommended Settings |
|-------------|----------------------|
| **Batch Size** | Increase `batch.size` for higher throughput. |
| **Compression** | Use `lz4` or `snappy` for efficiency. |
| **Message Acks** | Set `acks=all` for durability. |
| **Consumer Polling** | Increase `max.poll.records` for better performance. |

```properties
# Optimized Kafka Producer Config
batch.size=65536
compression.type=snappy
acks=all
linger.ms=10

# Optimized Kafka Consumer Config
max.poll.records=5000
fetch.min.bytes=4096
```

🔗 **Kafka Performance Tuning**: [Kafka Optimization Guide](https://kafka.apache.org/documentation/#design)  

---

### **Final Thoughts**  
Kafka configuration **determines reliability, performance, and scalability**. By **tuning producer, consumer, and broker parameters**, you can **optimize Kafka for high-throughput, exactly-once processing, and secure event streaming**.

### **Happy Streaming with Kafka! 🔥🚀**  
