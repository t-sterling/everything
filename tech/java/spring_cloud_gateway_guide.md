# 🌉 Getting Started with Spring Cloud Gateway

Spring Cloud Gateway is **an API gateway** that provides **intelligent routing, security, and resilience** for microservices.

📌 **Spring Cloud Gateway Docs**: [Spring Cloud Gateway](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/)  
📌 **Spring Cloud Project**: [Spring Cloud](https://spring.io/projects/spring-cloud)  
📌 **GitHub Repository**: [Spring Cloud Gateway GitHub](https://github.com/spring-cloud/spring-cloud-gateway)  

---

## **1. What is Spring Cloud Gateway?**  

Spring Cloud Gateway is **a reactive API gateway** that acts as a reverse proxy **for routing, load balancing, authentication, and monitoring of microservices**.

### **1.1 Key Features of Spring Cloud Gateway**  
✅ **Route Matching** – Routes requests based on URL patterns.  
✅ **Filters & Modifications** – Pre & post-processing of requests/responses.  
✅ **Circuit Breakers** – Handles failures using Resilience4J.  
✅ **Rate Limiting** – Protects APIs from excessive requests.  
✅ **Integrated with Spring Boot** – Works seamlessly with microservices.  

🔗 **More on Spring Cloud Gateway**: [Spring Cloud Gateway Overview](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/)  

---

## **2. Setting Up Spring Cloud Gateway**  

### **2.1 Add Dependencies (`pom.xml`)**  
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

### **2.2 Configure Gateway Routes (`application.yml`)**  
```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: http://localhost:8081
          predicates:
            - Path=/users/**
          filters:
            - AddRequestHeader=X-User-Header, Demo
```

🔗 **More on Gateway Routes**: [Spring Cloud Routing](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#route-matching)  

---

## **3. Creating a Simple Gateway Application**  

### **3.1 Main Application (`GatewayApplication.java`)**  
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```

### **3.2 Testing Gateway Routes**  
Start a mock service on port `8081` and send requests via the gateway:
```sh
curl http://localhost:8080/users/1
```

🔗 **More on Gateway Filters**: [Spring Cloud Filters](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#gatewayfilter-factories)  

---

## **4. Advanced Features in Spring Cloud Gateway**  

### **4.1 Global Filters for Request Modification**  
```java
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.stereotype.Component;
import reactor.core.publisher.Mono;

@Component
public class LoggingFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(org.springframework.cloud.gateway.filter.ServerWebExchange exchange, 
                             org.springframework.cloud.gateway.filter.GatewayFilterChain chain) {
        System.out.println("Incoming request: " + exchange.getRequest().getURI());
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return -1;
    }
}
```

### **4.2 Applying Rate Limiting with Redis**  
```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: http://localhost:8081
          predicates:
            - Path=/users/**
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20
```

🔗 **More on Rate Limiting**: [Rate Limiting Guide](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#rate-limiting)  

---

## **5. Running & Testing the Application**  

### **5.1 Start the Gateway Application**  
```sh
mvn spring-boot:run
```

### **5.2 Send Requests Through the Gateway**  
```sh
curl http://localhost:8080/users/1
```

🔗 **More on Spring Cloud Gateway Debugging**: [Troubleshooting](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#troubleshooting)  

---

## **6. Debugging & Best Practices for Spring Cloud Gateway**  

### **6.1 Enable Debug Logging**  
```yaml
logging:
  level:
    org.springframework.cloud.gateway: DEBUG
```

### **6.2 Best Practices for Spring Cloud Gateway**  
| Best Practice | Why It Matters |
|--------------|---------------|
| **Use Circuit Breakers** | Prevents cascading failures. |
| **Enable Rate Limiting** | Protects against DDoS attacks. |
| **Cache Responses Where Possible** | Improves performance. |
| **Monitor with Metrics** | Enables real-time insights. |

🔗 **More on Gateway Performance**: [Performance Tuning](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/#performance)  

---

### **Final Thoughts**  
Spring Cloud Gateway provides **a robust, scalable API gateway** for **routing, filtering, security, and monitoring** of microservices. By leveraging **filters, rate limiting, and circuit breakers**, teams can **enhance the reliability and security of their API infrastructure**.

### **Happy API Routing with Spring Cloud Gateway! 🌉🚀**  
