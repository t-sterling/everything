# 🔍 OpenSearch & Elasticsearch Cheat Sheet: CLI, API, & Java

This cheat sheet provides **essential OpenSearch/Elasticsearch CLI commands, REST API queries, and Java API usage** for interacting with search engines efficiently.

📌 **OpenSearch API Reference**: [OpenSearch REST API](https://opensearch.org/docs/latest/opensearch/rest-api/)  
📌 **Elasticsearch API Reference**: [Elasticsearch REST API](https://www.elastic.co/guide/en/elasticsearch/reference/current/rest-apis.html)  
📌 **Java OpenSearch Client**: [OpenSearch Java SDK](https://opensearch.org/docs/latest/clients/java/)  

---

## **1. Index & Document Management**  

### **1.1 Create an Index**
```json
PUT my_index
{
  "settings": { "number_of_shards": 3, "number_of_replicas": 2 }
}
```

### **1.2 Insert a Document**
```json
PUT my_index/_doc/1
{
  "name": "Alice",
  "age": 30,
  "city": "New York"
}
```

### **1.3 Retrieve a Document**
```json
GET my_index/_doc/1
```

### **1.4 Delete a Document**
```json
DELETE my_index/_doc/1
```

🔗 **Managing Documents**: [OpenSearch Index API](https://opensearch.org/docs/latest/opensearch/rest-api/index-apis/)  

---

## **2. Searching & Querying**  

### **2.1 Basic Match Query**
```json
GET my_index/_search
{
  "query": {
    "match": { "name": "Alice" }
  }
}
```

### **2.2 Boolean Query (AND, OR, NOT)**
```json
GET my_index/_search
{
  "query": {
    "bool": {
      "must": [ { "match": { "city": "New York" } } ],
      "must_not": [ { "match": { "age": "25" } } ]
    }
  }
}
```

### **2.3 Range Query (Numeric & Date Filtering)**
```json
GET my_index/_search
{
  "query": {
    "range": { "age": { "gte": 30, "lte": 40 } }
  }
}
```

🔗 **Query DSL Guide**: [Elasticsearch Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)  

---

## **3. Aggregations & Analytics**  

### **3.1 Count Documents by City**
```json
GET my_index/_search
{
  "size": 0,
  "aggs": {
    "cities": {
      "terms": { "field": "city.keyword" }
    }
  }
}
```

### **3.2 Average Age Calculation**
```json
GET my_index/_search
{
  "size": 0,
  "aggs": {
    "average_age": {
      "avg": { "field": "age" }
    }
  }
}
```

🔗 **Aggregation API**: [Elasticsearch Aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations.html)  

---

## **4. Performance Optimization & Scaling**  

### **4.1 Increase Index Refresh Interval (Reduce Write Overhead)**
```json
PUT my_index/_settings
{
  "index": {
    "refresh_interval": "30s"
  }
}
```

### **4.2 Use Bulk API for Mass Inserts**
```json
POST _bulk
{ "index": { "_index": "my_index", "_id": "1" } }
{ "name": "Alice", "age": 30, "city": "New York" }
{ "index": { "_index": "my_index", "_id": "2" } }
{ "name": "Bob", "age": 25, "city": "Chicago" }
```

🔗 **Bulk API Guide**: [OpenSearch Bulk API](https://opensearch.org/docs/latest/opensearch/rest-api/document-apis/bulk/)  

---

## **5. Java API (OpenSearch & Elasticsearch Clients)**  

### **5.1 Java API: Creating an Index (OpenSearch Client)**
```java
import org.opensearch.client.opensearch.OpenSearchClient;
import org.opensearch.client.opensearch.indices.CreateIndexRequest;

CreateIndexRequest request = new CreateIndexRequest.Builder()
        .index("my_index")
        .build();

client.indices().create(request);
```

### **5.2 Java API: Indexing a Document**
```java
import org.opensearch.client.opensearch.core.IndexRequest;

IndexRequest<Map<String, Object>> request = new IndexRequest.Builder<Map<String, Object>>()
        .index("my_index")
        .id("1")
        .document(Map.of("name", "Alice", "age", 30, "city", "New York"))
        .build();

client.index(request);
```

### **5.3 Java API: Searching for Documents**
```java
import org.opensearch.client.opensearch.core.SearchRequest;
import org.opensearch.client.opensearch.core.search.Query;

SearchRequest request = new SearchRequest.Builder()
        .index("my_index")
        .query(Query.of(q -> q.match(m -> m.field("name").query("Alice"))))
        .build();

SearchResponse response = client.search(request, Map.class);
```

🔗 **OpenSearch Java SDK**: [OpenSearch Java Client](https://opensearch.org/docs/latest/clients/java/)  

---

## **6. OpenSearch Clustering & High Availability**  

### **6.1 Check Cluster Health**
```json
GET _cluster/health
```

### **6.2 View Cluster Nodes**
```json
GET _cat/nodes?v
```

### **6.3 Enable Replication for High Availability**
```json
PUT my_index/_settings
{
  "index": {
    "number_of_replicas": 2
  }
}
```

🔗 **Cluster Management**: [Elasticsearch Cluster API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster.html)  

---

## **7. Security & Access Control**  

### **7.1 Enable Authentication & Users (OpenSearch Security)**
```json
PUT _security/user/alice
{
  "password" : "securepass",
  "roles" : [ "admin" ]
}
```

🔗 **OpenSearch Security Guide**: [OpenSearch Security Plugin](https://opensearch.org/docs/latest/security-plugin/)  

---

### **Final Thoughts**  
OpenSearch and Elasticsearch provide **high-performance, scalable search capabilities** for **log analytics, full-text search, and monitoring solutions**. Whether using **REST API, Java SDK, or OpenSearch Dashboards**, they **power large-scale data retrieval and analytics efficiently**.

### **Happy Searching with OpenSearch & Elasticsearch! 🔍🚀**  
