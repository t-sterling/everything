# 🔥 Redis Cheat Sheet: Commands & Java API

This cheat sheet provides **essential Redis commands** and **Java API usage** for interacting with Redis efficiently.

📌 **Redis Commands Reference**: [Redis Commands](https://redis.io/commands)  
📌 **Lettuce Java Client**: [Lettuce Redis Docs](https://lettuce.io/)  
📌 **Jedis Java Client**: [Jedis GitHub](https://github.com/redis/jedis)  

---

## **1. Redis CLI Commands**

### **1.1 Basic Key Operations**
```sh
SET mykey "Hello"       # Set a key-value pair
GET mykey               # Get value of a key
DEL mykey               # Delete a key
EXISTS mykey            # Check if key exists
EXPIRE mykey 60         # Set key expiry (TTL in seconds)
TTL mykey               # Get time-to-live for a key
```

### **1.2 Data Structures**  

#### **Strings**  
```sh
INCR counter            # Increment integer value
DECR counter            # Decrement integer value
APPEND mykey " World"   # Append value to existing key
```

#### **Lists**  
```sh
LPUSH mylist "A"        # Push to left
RPUSH mylist "B"        # Push to right
LPOP mylist             # Pop from left
RPOP mylist             # Pop from right
LRANGE mylist 0 -1      # Get all elements
```

#### **Sets**  
```sh
SADD myset "apple"      # Add to set
SISMEMBER myset "apple" # Check membership
SMEMBERS myset          # Get all members
```

#### **Sorted Sets (ZSet)**  
```sh
ZADD leaderboard 100 "Alice"  # Add element with score
ZRANGE leaderboard 0 -1       # Get elements sorted by score
```

#### **Hashes**  
```sh
HSET user:1 name "Alice" age "30"  # Set fields in hash
HGET user:1 name                  # Get a field
HGETALL user:1                    # Get all fields
```

#### **Streams**  
```sh
XADD mystream * temperature 25 humidity 70  # Add to stream
XRANGE mystream - +                          # Get all events
```

---

## **2. Java Redis API (Jedis & Lettuce)**  

### **2.1 Jedis (Synchronous Client)**
```java
import redis.clients.jedis.Jedis;

public class RedisExample {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("localhost");
        jedis.set("mykey", "Hello Redis");
        System.out.println(jedis.get("mykey"));
        jedis.close();
    }
}
```

### **2.2 Lettuce (Asynchronous Client)**
```java
import io.lettuce.core.RedisClient;
import io.lettuce.core.api.sync.RedisCommands;

public class RedisExample {
    public static void main(String[] args) {
        RedisClient client = RedisClient.create("redis://localhost:6379");
        RedisCommands<String, String> commands = client.connect().sync();
        commands.set("mykey", "Hello Lettuce");
        System.out.println(commands.get("mykey"));
        client.shutdown();
    }
}
```

---

## **3. Redis Pub/Sub Messaging**  

### **3.1 Redis CLI**  
```sh
SUBSCRIBE mychannel      # Subscribe to a channel
PUBLISH mychannel "Hello" # Publish message to channel
```

### **3.2 Java API (Jedis)**  
```java
jedis.subscribe((channel, message) -> System.out.println("Received: " + message), "mychannel");
```

---

## **4. Redis Transactions & Pipelining**  

### **4.1 Transactions (Atomic Execution)**  
```sh
MULTI
SET balance 100
DECR balance
EXEC
```

### **4.2 Java Transactions (Jedis)**  
```java
Transaction transaction = jedis.multi();
transaction.set("balance", "100");
transaction.decr("balance");
transaction.exec();
```

### **4.3 Pipelining (Batch Processing)**  
```sh
redis-cli --pipe < commands.txt  # Execute batched commands
```

### **4.4 Java Pipelining (Jedis)**  
```java
Pipeline pipeline = jedis.pipelined();
pipeline.set("key1", "value1");
pipeline.set("key2", "value2");
pipeline.sync();
```

---

## **5. Redis Clustering & High Availability**  

### **5.1 Enable Redis Sentinel for High Availability**
```sh
redis-sentinel /etc/redis/sentinel.conf
```

### **5.2 Java Client for Redis Sentinel (Jedis)**  
```java
JedisSentinelPool sentinelPool = new JedisSentinelPool("mymaster", Set.of("127.0.0.1:26379"));
Jedis jedis = sentinelPool.getResource();
```

### **5.3 Redis Cluster Setup**
```sh
redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 --cluster-replicas 1
```

### **5.4 Java Redis Cluster Connection (Jedis)**
```java
JedisCluster cluster = new JedisCluster(new HostAndPort("127.0.0.1", 7000));
cluster.set("mykey", "Hello Cluster");
System.out.println(cluster.get("mykey"));
```

---

## **6. Redis Streams (Event Processing)**  

### **6.1 Add & Read Stream Events**
```sh
XADD mystream * temperature 25 humidity 70
XRANGE mystream - +
```

### **6.2 Java Redis Streams (Lettuce)**
```java
StreamMessage<String, String> message = StreamMessage.create("mystream", "temperature", "25");
commands.xadd("mystream", message);
```

---

### **Final Thoughts**  
Redis is a **powerful in-memory data store** that supports **various data structures, transactions, clustering, and event streaming**. Whether using the **CLI, Jedis, or Lettuce**, Redis provides **blazing-fast performance for real-time applications**.

### **Happy Scaling with Redis! 🔥🚀**  
