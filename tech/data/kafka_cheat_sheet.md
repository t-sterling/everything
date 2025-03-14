# 🦄 Kafka Cheat Sheet

## **1. Installation & Version**
Apache Kafka is a distributed event streaming platform used for high-performance data pipelines, streaming analytics, and event-driven applications.

```sh
kafka-topics.sh --version         # Check Kafka version
zookeeper-server-start.sh config/zookeeper.properties  # Start Zookeeper (if using it)
kafka-server-start.sh config/server.properties        # Start Kafka broker
```

---

## **2. Working with Topics**
Kafka **topics** store messages in a distributed and durable manner.

```sh
kafka-topics.sh --create --topic my-topic --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1  # Create a topic
kafka-topics.sh --list --bootstrap-server localhost:9092                       # List all topics
kafka-topics.sh --describe --topic my-topic --bootstrap-server localhost:9092  # Get topic details
kafka-topics.sh --delete --topic my-topic --bootstrap-server localhost:9092    # Delete a topic
```

---

## **3. Producing & Consuming Messages**
Kafka **producers** publish messages, and **consumers** subscribe to topics.

```sh
kafka-console-producer.sh --topic my-topic --bootstrap-server localhost:9092   # Start a producer
> Hello Kafka!  # (Type a message and hit enter)

kafka-console-consumer.sh --topic my-topic --from-beginning --bootstrap-server localhost:9092  # Start a consumer
```

---

## **4. Consumer Groups**
A **consumer group** allows multiple consumers to share a topic.

```sh
kafka-consumer-groups.sh --list --bootstrap-server localhost:9092              # List consumer groups
kafka-consumer-groups.sh --describe --group my-group --bootstrap-server localhost:9092  # Describe a consumer group
kafka-consumer-groups.sh --delete --group my-group --bootstrap-server localhost:9092  # Delete a consumer group
```

---

## **5. Monitoring Kafka**
Monitor Kafka cluster health and message flow.

```sh
kafka-run-class.sh kafka.tools.ConsumerOffsetChecker --group my-group --bootstrap-server localhost:9092  # Check consumer offsets
kafka-run-class.sh kafka.admin.TopicCommand --describe --topic my-topic --bootstrap-server localhost:9092  # Describe topic partitions
kafka-run-class.sh kafka.tools.GetOffsetShell --topic my-topic --bootstrap-server localhost:9092  # Get topic offsets
```

---

## **6. Kafka Partitions & Replication**
Kafka **partitions** enable parallel processing, and **replication** provides fault tolerance.

```sh
kafka-topics.sh --alter --topic my-topic --partitions 5 --bootstrap-server localhost:9092  # Increase partitions
```

---

## **7. Retention Policies**
Kafka can automatically delete or compact old messages.

```sh
kafka-configs.sh --alter --entity-type topics --entity-name my-topic --add-config retention.ms=86400000 --bootstrap-server localhost:9092  # Set retention to 1 day
```

---

## **8. Kafka Connect (Data Integration)**
Kafka Connect simplifies integrating Kafka with databases and cloud services.

```sh
connect-standalone.sh config/connect-standalone.properties config/my-sink-config.properties  # Start a Kafka Connect standalone process
```

---

## **9. Schema Registry & Avro (Serialization)**
Schema Registry manages schemas for Avro, JSON, and Protobuf messages.

```sh
curl -X GET http://localhost:8081/subjects  # List registered schemas
```

---

## **10. Security & Authentication**
Kafka supports **SSL, SASL, and ACLs** for security.

```sh
kafka-acls.sh --add --allow-principal User:alice --operation Read --topic my-topic --bootstrap-server localhost:9092  # Grant read access to a user
```

---

## **11. Cleanup & Debugging**
```sh
kafka-delete-records.sh --offset-json-file offsets.json --bootstrap-server localhost:9092  # Delete records up to specific offsets
kafka-log-dirs.sh --describe --bootstrap-server localhost:9092  # Inspect log directories
```

---

### **Happy Streaming with Kafka! 🚀**
