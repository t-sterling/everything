# 🚀 Using Cypher with Java & Spring Boot Data (Neo4j)

This guide explains **how to use Cypher queries with Java and Spring Boot** using **Spring Data Neo4j**, covering **setup, querying, repositories, and best practices**.

📌 **Spring Data Neo4j Docs**: [Spring Data Neo4j](https://docs.spring.io/spring-data/neo4j/docs/current/reference/html/)  
📌 **Neo4j Java Driver**: [Neo4j Java SDK](https://neo4j.com/developer/java/)  
📌 **Spring Boot Integration**: [Spring Boot & Neo4j](https://spring.io/projects/spring-data-neo4j)  

---

## **1. What is Spring Data Neo4j?**  

Spring Data Neo4j is **a Spring-based integration** that allows Java applications to interact with **Neo4j graph databases** using **repositories and Cypher queries**.

### **1.1 Key Features of Spring Data Neo4j**  
✅ **Declarative Repositories** – Define graph queries with interfaces.  
✅ **Custom Cypher Queries** – Use Cypher directly in repository methods.  
✅ **Object-Graph Mapping (OGM)** – Maps Java objects to graph nodes.  
✅ **Transaction Management** – Handles database transactions.  
✅ **Integration with Spring Boot** – Works with `spring-boot-starter-data-neo4j`.  

🔗 **More on Spring Data Neo4j**: [Spring Data Neo4j Guide](https://spring.io/projects/spring-data-neo4j)  

---

## **2. Setting Up Neo4j with Spring Boot**  

### **2.1 Add Dependencies (`pom.xml`)**  
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-neo4j</artifactId>
</dependency>
<dependency>
    <groupId>org.neo4j.driver</groupId>
    <artifactId>neo4j-java-driver</artifactId>
    <version>5.11.0</version>
</dependency>
```

### **2.2 Configure Neo4j Connection (`application.yml`)**  
```yaml
spring:
  neo4j:
    uri: bolt://localhost:7687
    authentication:
      username: neo4j
      password: password
```

🔗 **More on Neo4j Configuration**: [Neo4j Config Docs](https://neo4j.com/developer/java/)  

---

## **3. Defining Graph Entities in Java**  

### **3.1 Create a `@Node` Entity**  
```java
import org.springframework.data.neo4j.core.schema.*;

@Node("Person")
public class Person {
    @Id @GeneratedValue private Long id;
    private String name;
    private int age;

    @Relationship(type = "FRIEND", direction = Relationship.Direction.OUTGOING)
    private Set<Person> friends = new HashSet<>();

    // Constructors, Getters, and Setters
}
```

🔗 **More on Graph Entities**: [Spring Data Neo4j Annotations](https://docs.spring.io/spring-data/neo4j/docs/current/reference/html/#mapping.annotations)  

---

## **4. Creating a Repository with Cypher Queries**  

### **4.1 Define a Repository Interface**  
```java
import org.springframework.data.neo4j.repository.Neo4jRepository;
import org.springframework.data.neo4j.repository.query.Query;
import java.util.List;

public interface PersonRepository extends Neo4jRepository<Person, Long> {
    
    // Query using method name convention
    List<Person> findByName(String name);

    // Custom Cypher Query
    @Query("MATCH (p:Person)-[:FRIEND]->(f:Person) WHERE p.name = $name RETURN f")
    List<Person> findFriendsByPersonName(String name);
}
```

🔗 **More on Spring Data Repositories**: [Spring Data Neo4j Queries](https://docs.spring.io/spring-data/neo4j/docs/current/reference/html/#repositories)  

---

## **5. Running Cypher Queries in a Service**  

### **5.1 Create a Service Class**  
```java
import org.springframework.stereotype.Service;
import java.util.List;

@Service
public class PersonService {
    
    private final PersonRepository personRepository;

    public PersonService(PersonRepository personRepository) {
        this.personRepository = personRepository;
    }

    public List<Person> getFriends(String name) {
        return personRepository.findFriendsByPersonName(name);
    }
}
```

🔗 **More on Neo4j Service Layers**: [Spring Boot Services](https://spring.io/guides/gs/service-layer/)  

---

## **6. Exposing Graph Queries via REST API**  

### **6.1 Create a REST Controller**  
```java
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/api/persons")
public class PersonController {

    private final PersonService personService;

    public PersonController(PersonService personService) {
        this.personService = personService;
    }

    @GetMapping("/{name}/friends")
    public List<Person> getFriends(@PathVariable String name) {
        return personService.getFriends(name);
    }
}
```

🔗 **More on Spring Boot REST APIs**: [Spring REST Docs](https://spring.io/guides/gs/rest-service/)  

---

## **7. Running & Testing the Application**  

### **7.1 Start Neo4j Database (Docker)**  
```sh
docker run -d --name neo4j -p 7474:7474 -p 7687:7687 -e NEO4J_AUTH=neo4j/password neo4j
```

### **7.2 Run the Spring Boot App**  
```sh
mvn spring-boot:run
```

### **7.3 Test the API (cURL)**  
```sh
curl http://localhost:8080/api/persons/Alice/friends
```

🔗 **More on Neo4j with Docker**: [Neo4j Docker Setup](https://neo4j.com/developer/docker/)  

---

## **8. Debugging & Best Practices in Neo4j & Cypher**  

### **8.1 Debugging Queries with `EXPLAIN`**  
```cypher
EXPLAIN MATCH (p:Person) RETURN p
```

### **8.2 Using `PROFILE` for Execution Plan**  
```cypher
PROFILE MATCH (p:Person) RETURN p
```

### **8.3 Best Practices for Neo4j with Spring Boot**  
| Best Practice | Why It Matters |
|--------------|---------------|
| **Use Indexes** | Speeds up searches and optimizes performance. |
| **Limit Query Scope** | Avoids unnecessary large graph traversals. |
| **Use Relationships Efficiently** | Avoids redundancy and improves query performance. |
| **Enable Query Logging** | Helps debug Cypher execution plans. |
| **Use Transactions for Bulk Inserts** | Ensures consistency in large operations. |

🔗 **More on Performance Optimization**: [Neo4j Performance Tuning](https://neo4j.com/docs/cypher-manual/current/query-tuning/)  

---

### **Final Thoughts**  
Spring Data Neo4j enables **seamless integration between Java applications and Neo4j**. By leveraging **repositories, Cypher queries, and RESTful APIs**, developers can **build powerful graph-driven applications**.

### **Happy Graph Querying with Spring Boot & Neo4j! 🚀**  
