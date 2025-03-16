I will create a detailed technical guide on scaling a web API on AWS using Auto Scaling and CloudFront, covering REST, GraphQL, WebSockets, and gRPC. The guide will include examples with Terraform and a Spring Boot implementation, along with authentication mechanisms like JWT, OAuth2, and AWS Cognito. It will also explore integrating with Amazon EKS and Kubernetes autoscaling, including sticky routing, stateful vs. stateless workloads, user session management, and security best practices. I will structure it with Markdown, include mermaid diagrams, and provide all relevant links for reference. I'll notify you when it's ready.

# Scaling a Web API on AWS with Auto Scaling and CloudFront

## Introduction

Building a scalable web API on AWS involves careful consideration of API design, authentication, infrastructure provisioning, and automation. This guide provides a step-by-step walkthrough for scaling different types of APIs (REST, GraphQL, WebSockets, gRPC) on AWS. We will cover securing the APIs with JWT, OAuth2, and AWS Cognito, provisioning infrastructure as code using Terraform, and implementing auto-scaling with AWS Auto Scaling Groups and Kubernetes (EKS) with CloudFront as a CDN layer. We include Spring Boot code examples illustrating how to implement these APIs, discuss stateless vs. stateful design (sticky sessions), user session management, and security best practices. Finally, we tie everything together with a complete example architecture, accompanied by **Mermaid** diagrams for clarity. This guide is structured as a Markdown document for easy reading and reference, with code snippets and configuration examples included throughout.

## Types of Web APIs and Scaling Considerations

Modern web APIs come in various styles and protocols. The choice of API type influences how you design for scalability and performance. Here we overview four common API types – **REST**, **GraphQL**, **WebSockets**, and **gRPC** – and discuss their characteristics relevant to scaling.

### REST APIs

**REST (Representational State Transfer)** is a widely-used architectural style for web services. REST APIs communicate over HTTP with a request-response model and are typically **stateless**, meaning each request contains all information needed, and the server does not store client context between requests ([ REST vs. GraphQL vs. gRPC vs. WebSocket](https://www.resolutesoftware.com/blog/rest-vs-graphql-vs-grpc-vs-websocket/#:~:text=1,context%20information%20on%20the%20server)). This statelessness makes REST straightforward to scale horizontally: any instance can handle any request independently, simplifying load balancing and auto-scaling. REST responses can also be marked as cacheable, allowing intermediate caches or CDNs to store responses for repeated requests to reduce load ([ REST vs. GraphQL vs. gRPC vs. WebSocket](https://www.resolutesoftware.com/blog/rest-vs-graphql-vs-grpc-vs-websocket/#:~:text=)). REST’s simplicity and ubiquity make it easy to implement and test, and it works well for **general-purpose APIs** consumed by many clients.

Key scaling considerations for REST APIs:

- **Stateless design** – Because the server does not retain session info by default, you can add or remove instances behind a load balancer without session affinity (we will discuss stateful vs. stateless later).
- **Caching** – REST supports HTTP caching (e.g., using `ETag` or `Cache-Control` headers). Services like CloudFront can cache GET responses to offload traffic.
- **Bandwidth** – Each REST endpoint returns a fixed data format. Over-fetching or under-fetching data is possible (clients may get more or less data than needed), which can impact performance. GraphQL (next section) addresses this issue.

In summary, REST APIs are simple to implement and **scale well** for most use cases ([API Protocols 101: A Guide to Choose the Right One ](https://blog.bytebytego.com/p/api-protocols-101-a-guide-to-choose#:~:text=provides%20more%20control%20over%20data,fetching)), especially when following stateless principles and leveraging caching.

### GraphQL APIs

**GraphQL** is a query language for APIs that allows clients to request exactly the data they need. A GraphQL API typically exposes a single HTTP endpoint (e.g. `/graphql`) that accepts query and mutation requests in the GraphQL format. Like REST, GraphQL usually operates over HTTP and can be designed statelessly (each request is independent), making it similarly amenable to horizontal scaling ([API Protocols 101: A Guide to Choose the Right One ](https://blog.bytebytego.com/p/api-protocols-101-a-guide-to-choose#:~:text=provides%20more%20control%20over%20data,fetching)). The key advantage of GraphQL is efficiency: clients can combine data from multiple sources in one request and avoid over-fetching, which **optimizes bandwidth and performance** ([ REST vs. GraphQL vs. gRPC vs. WebSocket](https://www.resolutesoftware.com/blog/rest-vs-graphql-vs-grpc-vs-websocket/#:~:text=match%20at%20L254%20GraphQL%20gives,solves%20issues%20with%20bandwidth%20efficiency)). This can reduce the overall load on the backend and improve perceived speed for clients, which helps when scaling.

Considerations for GraphQL:

- **Single Endpoint** – All queries go to one endpoint, so load balancing is straightforward (similar to REST) and caching can be done at the level of query results if desired. CloudFront or other caches can cache GraphQL responses, but caching is trickier because request payloads (queries) determine the response.
- **Complexity** – The server must resolve potentially complex queries (joining data from various sources). This can increase backend load per request, so you must design resolvers efficiently and possibly use batching or dataloader patterns to avoid performance issues under scale.
- **Versioning** – GraphQL schemas evolve in a backward-compatible way which can simplify client upgrades, but careful schema design is needed to avoid performance pitfalls.

Overall, GraphQL **scales well** and offers more precise data fetching control ([API Protocols 101: A Guide to Choose the Right One ](https://blog.bytebytego.com/p/api-protocols-101-a-guide-to-choose#:~:text=,more%20control%20over%20data%20fetching)) ([ REST vs. GraphQL vs. gRPC vs. WebSocket](https://www.resolutesoftware.com/blog/rest-vs-graphql-vs-grpc-vs-websocket/#:~:text=match%20at%20L254%20GraphQL%20gives,solves%20issues%20with%20bandwidth%20efficiency)), which can minimize wasted work and improve performance in high-load scenarios.

### WebSocket APIs

**WebSockets** enable full-duplex, persistent connections between clients and server, allowing real-time communication (e.g. chat apps, live notifications). Scaling WebSocket-based APIs presents different challenges compared to stateless request/response protocols:

- **Long-lived Connections**: Once a WebSocket connection is established, it remains open. Load balancers (like AWS Application Load Balancer) support WebSocket by routing the initial handshake and then keeping the TCP connection pinned to one backend server. This effectively creates a stateful interaction (the connection state lives on one server). Auto-scaling WebSocket servers means new servers can be added for new connections, but existing connections won’t move to them. Ensuring you have capacity for peak concurrent connections is critical.
- **Sticky Sessions**: If your WebSocket server instance maintains user-specific state in memory (which it often will, for the open connection), you might require **session stickiness** for any HTTP fallback or related API calls. However, if the WebSocket is purely used for async messages and the rest of the API is stateless, you can often manage with just connection stickiness at the load balancer (which ALB provides inherently for WebSockets).
- **Scaling Broadcasts**: In scenarios like chatrooms or notifications, you may need to broadcast messages to many connected clients across multiple servers. To scale this, you'll introduce a message broker or pub/sub system (e.g., Amazon SNS, Redis Pub/Sub, or AWS IoT PubSub) so that each instance can publish messages and all others can receive and forward to their local clients. This adds complexity but is necessary for horizontal scaling of real-time features.
- **CloudFront**: AWS CloudFront supports WebSocket traffic passthrough ([API & dynamic acceleration](https://aws.amazon.com/developer/application-security-performance/articles/api-dynamic-acceleration/#:~:text=this%20workshop)), meaning you can still use CloudFront as a global entry point for WebSocket connections (it will forward the connection to an origin like an ALB). This can reduce latency by terminating TLS closer to users and keep connections persistent on the AWS backbone.

In summary, WebSockets can scale to large numbers of concurrent connections by load balancing across instances, but design is more stateful. Emphasize externalizing any sharable state and use robust messaging between nodes. Ensure the load balancer and any CDN (CloudFront) are configured to support the WebSocket upgrade and persistent connection.

### gRPC APIs

**gRPC** is a high-performance RPC framework from Google, using HTTP/2 for transport and Protocol Buffers for efficient binary serialization. gRPC is well-suited for microservices or client-server communication where performance is critical. It supports unary request/response as well as server-streaming, client-streaming, and bidirectional streaming RPCs. Key points for scaling gRPC:

- **HTTP/2 and Connections**: gRPC uses HTTP/2, which multiplexes many calls over a single TCP connection. Similar to WebSockets, a client may reuse one connection to make multiple requests to a server. Load balancers (AWS ALB) now support gRPC and can balance HTTP/2 traffic, but typically each client will stick to one server per connection. This means you get effective stickiness on a per-connection basis (though new HTTP/2 connections could be routed to different servers). For high load, clients or client libraries often open multiple connections to improve parallelism.
- **Horizontal Scaling**: Like REST, you can run multiple gRPC server instances and load balance between them. Because gRPC calls are usually stateless (each RPC is independent), scaling horizontally is effective. gRPC is designed to be **very efficient and low-latency**, making it *more efficient for microservices* communications ([API Protocols 101: A Guide to Choose the Right One ](https://blog.bytebytego.com/p/api-protocols-101-a-guide-to-choose#:~:text=provides%20more%20control%20over%20data,fetching)). Many large companies use gRPC for internal APIs due to its performance ([ REST vs. GraphQL vs. gRPC vs. WebSocket](https://www.resolutesoftware.com/blog/rest-vs-graphql-vs-grpc-vs-websocket/#:~:text=gRPC%20is%20a%20robust%20open,API%20security%2C%20performance%2C%20and%20scalability)).
- **Client Libraries**: One consideration is that gRPC requires generated client stubs. Scaling in terms of number of clients (especially web browsers) can be limited because browsers don't natively support gRPC (though there is gRPC-Web which uses an HTTP/1.1 bridge). For mobile or backend-to-backend, gRPC works great. If exposing gRPC to external developers, ensure they can use it (or provide a REST/GraphQL alternative if needed).
- **CloudFront**: AWS CloudFront can also forward gRPC traffic to origins (HTTP/2), as noted by AWS that CloudFront supports gRPC-based APIs ([API & dynamic acceleration](https://aws.amazon.com/developer/application-security-performance/articles/api-dynamic-acceleration/#:~:text=this%20workshop)). This can help distribute and accelerate gRPC services globally, similar to HTTP.

In practice, use gRPC for internal or high-performance services where you control the environment. It provides strong **performance and scalability** characteristics by design ([ REST vs. GraphQL vs. gRPC vs. WebSocket](https://www.resolutesoftware.com/blog/rest-vs-graphql-vs-grpc-vs-websocket/#:~:text=gRPC%20is%20a%20robust%20open,API%20security%2C%20performance%2C%20and%20scalability)), but requires more setup (Proto file definitions, client generation). Combine gRPC with horizontal scaling and possibly Kubernetes/ECS for orchestrating many instances.

### Summary of API Types

Each API type can be scaled on AWS, but the approach differs:

- **REST**: Easiest to scale due to statelessness and caching. Use Auto Scaling Groups or Kubernetes to add more instances behind a Load Balancer.
- **GraphQL**: Also stateless and scalable; be mindful of query complexity. Similar infra as REST.
- **WebSockets**: Requires sticky connection handling. Use ALB (which supports WebSockets) and design for potentially stateful servers or externalized state.
- **gRPC**: Highly performant; ensure load balancers support HTTP/2. Ideal for microservice to microservice calls with Auto Scaling as needed.

By choosing stateless protocols where possible, you simplify scaling. In the next sections, we’ll discuss how to secure these APIs and then how to deploy them on AWS with auto-scaling.

## Authentication and Authorization Mechanisms

Security and scalability go hand-in-hand: a robust authentication mechanism ensures only legitimate traffic reaches your API (preventing overload by malicious requests) ([Securing Your REST API with JWT Authorizers and Amazon Cognito - DEV Community](https://dev.to/aws-builders/securing-your-rest-api-with-jwt-authorizers-and-amazon-cognito-2dp2#:~:text=5,overload%20due%20to%20malicious%20requests)), and a stateless auth design avoids introducing bottlenecks in a distributed environment. We will explore **JWTs**, **OAuth2**, and **AWS Cognito** as authentication solutions for our APIs.

### JWT (JSON Web Tokens)

**JWT** is a compact, URL-safe token format that contains a set of *claims* (user data and metadata) and is cryptographically signed. JWTs are used for stateless authentication: the token itself proves the user's identity (if signed by a trusted authority) without the server storing session data. This is ideal for scalable APIs because each request can be authenticated by verifying the token signature and claims, with no need to query a session store. **Stateless JWT auth reduces server overhead and improves scalability** ([JWT Authentication: A Secure & Scalable Solution for Modern Applications - Authgear](https://www.authgear.com/post/jwt-authentication-a-secure-scalable-solution-for-modern-applications#:~:text=If%20the%20JWT%20is%20valid%2C,reducing%20overhead%20and%20improving%20scalability)) since the server doesn’t need to remember any user state between requests.

Key points for using JWT in your API:

- **Issuance**: Usually, after a user logs in (e.g., via username/password or OAuth2 flow), your auth service (or Cognito, below) issues a JWT to the client. This JWT is then included in the `Authorization: Bearer <token>` header on subsequent API requests.
- **Validation**: Each API instance can validate the JWT by checking the signature (using a secret or public key) and ensuring it’s not expired and perhaps that it has the correct audience/scope. Libraries like Spring Security or jjwt can handle this.
- **No server session**: Because the JWT contains the info, you **don’t store session in memory or DB**. This means any number of API servers can handle the token equally – perfect for auto-scaling. JWTs are self-contained and **ideal for stateless RESTful APIs** ([JWT Authentication: A Secure & Scalable Solution for Modern Applications - Authgear](https://www.authgear.com/post/jwt-authentication-a-secure-scalable-solution-for-modern-applications#:~:text=JWT%20authentication%20is%20widely%20used,side%20sessions)).
- **Expiration and Refresh**: JWTs typically have an expiration time. For long-lived sessions, use refresh tokens (which could be JWT or opaque) to get new JWTs. Refresh tokens can be stored more securely (maybe with Cognito or in a DB) to allow revocation.
- **Security**: Treat JWTs like passwords – don’t leak them. Use HTTPS so they aren’t intercepted, and store them in a secure, httpOnly cookie or secure storage in apps to guard against XSS. Also, consider token revocation strategies (blacklist or short expiry) since stateless tokens by nature can’t be easily revoked server-side.

Using JWTs ensures that adding more API servers doesn’t require sharing session state – **each server independently authenticates requests**. This aligns perfectly with horizontal scaling and microservices architectures.

**Spring Boot Example – JWT Auth**: In a Spring Boot app, you might use Spring Security with a JWT filter. For example, configure a `OncePerRequestFilter` that reads the `Authorization` header, parses/validates the JWT, and sets the user authentication in the security context. This way, your controllers can simply use `@PreAuthorize` or check `Principal` as needed. The filter uses a secret or public key (for RSA) to validate the signature. (See references for numerous tutorials on Spring Security JWT integration ([Securing Java Microservices with OAuth2 and JWT Authentication](https://medium.com/@gunvantdc2907/securing-java-microservices-with-oauth2-and-jwt-authentication-47bbca941be1#:~:text=Securing%20Java%20Microservices%20with%20OAuth2,scalability%20in%20a%20distributed)).)

### OAuth2

**OAuth2** is an authorization framework often used for delegated access (letting users authorize third-party apps to access their data) and can also underpin authentication (via OpenID Connect or similar). In the context of scaling web APIs:

- **OAuth2 Roles**: Typically involves an **Authorization Server** (issues tokens after authenticating user), a **Resource Server** (your API, which accepts tokens), and a **Client** (the front-end or app requesting access). In a microservices setup, you might use a dedicated OAuth2 server (like Keycloak, Okta, Auth0, or AWS Cognito) to centralize authentication.
- **Token Types**: OAuth2 issues **access tokens** (often JWTs or opaque strings) and optionally **refresh tokens**. Access tokens in JWT form are particularly convenient for scaling (as described above). In fact, using **OAuth2 with JWT allows microservices to remain stateless, which is vital for scalability** ([Securing Java Microservices with OAuth2 and JWT Authentication](https://medium.com/@gunvantdc2907/securing-java-microservices-with-oauth2-and-jwt-authentication-47bbca941be1#:~:text=Securing%20Java%20Microservices%20with%20OAuth2,scalability%20in%20a%20distributed)).
- **Flows**: For single-page apps or mobile, the Authorization Code flow (with PKCE for SPA) is common: user is redirected to auth server, logs in, and the app receives an access token (JWT). That token is then used in API calls. For machine-to-machine, Client Credentials flow might be used (server to server auth with no user).
- **Validation**: If tokens are JWTs, your API can validate them locally (no DB lookup) which is scalable. If using opaque tokens, the API (Resource Server) might have to introspect the token with the auth server – this can become a bottleneck if not cached. Many OAuth2 setups use JWTs as access tokens to avoid that overhead.
- **Scope & Claims**: OAuth2 tokens often carry scopes or claims about user roles. Your API can use these for authorization decisions (e.g., only users with `admin` scope can access certain endpoints). Since the token is presented on each request, any instance can enforce these rules.

**Integration example**: Spring Boot can use Spring Security OAuth2 Resource Server support to configure JWT decoding. For instance, if using Cognito or Auth0 (which issue JWTs), you set the issuer URI and the expected audience in configuration, and Spring Security will handle validating the token for each request. This offloads most auth concerns to the OAuth2 provider, and your API remains stateless and focused on business logic.

In summary, OAuth2 provides a standardized way to handle auth, and when combined with JWTs as tokens, it meets the demands of a scalable system (each API instance independently validates tokens without shared state).

### AWS Cognito

**AWS Cognito** is a fully managed user identity service that integrates well with AWS APIs. It essentially can serve as the **authentication and authorization server** for your applications:

- **User Pools**: Cognito User Pools manage user registration, login, and user directory. Cognito can handle email/phone verification, multi-factor authentication (MFA), password resets, etc. When users log in, Cognito issues tokens (an **ID token** and **Access token**, both JWTs, plus a Refresh token) ([Securing Your REST API with JWT Authorizers and Amazon Cognito - DEV Community](https://dev.to/aws-builders/securing-your-rest-api-with-jwt-authorizers-and-amazon-cognito-2dp2#:~:text=,API%20request%20for%20continued%20access)). These tokens follow OpenID Connect standards and OAuth2 flows.
- **Identity Pools**: (Now called Cognito Federated Identities) – this is for granting AWS credentials to users (if your app needs direct AWS access). For our API scenario, the User Pool is the main component for auth; Identity Pools are optional if you need AWS service access.
- **Using Cognito for API Auth**: There are two common ways:
  1. **JWT Authorizer**: If you use API Gateway or ALB, you can integrate Cognito so that it validates JWTs for you before hitting your service. For instance, API Gateway can be configured with a Cognito authorizer that checks the token and rejects unauthorized requests at the gateway level. This reduces load on your service as unauthorized requests never reach it.
  2. **Direct JWT validation**: If your architecture is ALB + EC2/EKS (no API Gateway), you can still use Cognito. Your front-end would use Cognito's Hosted UI or SDK to perform login. On successful login, it gets Cognito tokens. The front-end then calls your API (through CloudFront/ALB) with the Cognito Access JWT in the header. Your service must validate this JWT. You can fetch Cognito’s public keys (JWKS) and verify the token’s signature and claims (Cognito tokens include iss, sub, cognito:username, and any custom claims or group membership). This validation can be done via a library or at the load balancer: note that AWS Application Load Balancer has a feature to authenticate via Cognito User Pools directly, eliminating the need to handle tokens in your app if you use ALB authentication action.
- **Scaling and Cognito**: Cognito itself is a managed service that scales to millions of users. By offloading user management and auth to Cognito, your system can scale without worrying about the overhead of storing sessions or validating passwords. Each API server just trusts the JWT issued by Cognito ([Securing Your REST API with JWT Authorizers and Amazon Cognito - DEV Community](https://dev.to/aws-builders/securing-your-rest-api-with-jwt-authorizers-and-amazon-cognito-2dp2#:~:text=,API%20request%20for%20continued%20access)). Also, because Cognito is external, you can update user profiles, revoke tokens (by disabling users or using token revocation with App Client settings for refresh tokens), without modifying your API servers.

**Example**: Suppose we secure our API with Cognito. We create a User Pool on AWS, and an App Client for our application. Users authenticate against the User Pool (using AWS Amplify library or Cognito Hosted UI). Cognito returns an `idToken` and `accessToken` (JWTs). We configure our Spring Boot API to accept these tokens. If using Spring Security, we supply the Cognito JWKs URL (something like `https://cognito-idp.<region>.amazonaws.com/<userPoolId>/.well-known/jwks.json`) and issuer URL. Spring Security will validate incoming JWTs automatically. Now our API is secured without us having to implement login or token generation at all – Cognito did it.

**OAuth2 vs Cognito**: Cognito under the hood implements OAuth2 and OpenID Connect. The difference is Cognito is an AWS-managed service providing a quick setup for user auth, whereas a generic OAuth2 solution (like Okta/Auth0 or DIY Keycloak) might offer more flexibility or multi-cloud. Since this guide focuses on AWS, Cognito is a natural choice for managed auth.

### Best Practices for Auth in Scalable Systems

- **Prefer stateless tokens**: Whether you use raw JWT or OAuth2, using tokens that can be validated without server-side lookup (other than checking a signature/keystore) ensures your auth layer doesn’t become a scalability bottleneck.
- **Short-lived tokens**: To reduce risk if stolen and to allow changes in user permissions to reflect, keep access tokens short (e.g., 15 min) and use refresh tokens to get new ones. Cognito’s defaults (1 hour access, 30 days refresh) can be adjusted.
- **Protect authentication endpoints**: If you run your own OAuth2 server, ensure it can scale (Cognito automatically does). Brute-force protection, rate limiting login attempts, CAPTCHA or MFA for sensitive accounts – these guardrails prevent the auth system from being overloaded or misused.
- **Use HTTPS everywhere**: All token transport must be over TLS. For web apps, consider using Secure and HttpOnly cookies for tokens to mitigate XSS and man-in-the-middle risks.
- **Monitor and log**: Keep track of authentication attempts, issued tokens, etc. AWS Cognito can log to CloudWatch. This helps in detecting abuse (which could affect availability).

With authentication in place, the next step is provisioning the infrastructure that will host and scale our API. We will use **Terraform** to define AWS resources like VPCs, Auto Scaling Groups, Load Balancers, CloudFront distributions, and even an EKS cluster.

## Infrastructure Provisioning with Terraform

Infrastructure as Code (IaC) allows us to codify our AWS setup and easily recreate or modify it. **Terraform** is a popular IaC tool that works with AWS. We will outline how to provision the infrastructure for a scalable API, including an Auto Scaling group of EC2 instances behind an ALB (Application Load Balancer), a CloudFront distribution as a CDN in front, and optionally an EKS cluster for Kubernetes-based deployments. 

By using Terraform, you can version control your infrastructure, reuse configurations for different environments, and ensure consistency across deployments.

### Terraform Setup

First, ensure you have Terraform installed and AWS credentials configured (e.g., via environment variables or AWS config files). In your Terraform configuration, you will typically:

- Define the AWS provider and region.
- Define networking (VPC, subnets, security groups) or reference existing ones.
- Define an EC2 Launch Template (or Launch Configuration) specifying the AMI, instance type, security groups, and user data for launching your API instances.
- Define an Auto Scaling Group (ASG) resource that uses the launch template and spans across multiple AZs for high availability.
- Define an Elastic Load Balancing v2 resource (Application Load Balancer and Target Group) to distribute traffic to instances.
- Possibly define scaling policies (target tracking or step scaling) for the ASG.
- Define a CloudFront Distribution with the ALB as an origin.
- (If using EKS) Define EKS cluster, node groups, and necessary IAM roles.

Let’s go through these components with snippets:

**VPC and Networking** (briefly): A production setup should use a custom VPC with multiple subnets. For simplicity, assume we have a VPC with two public subnets (for ALB and instances). Terraform resources `aws_vpc`, `aws_subnet`, `aws_internet_gateway`, `aws_route_table` etc., would be used. Ensure the subnets for instances have access to the internet (for updates, etc.) either via public IPs or NAT gateways.

**Security Groups**: Create a Security Group for ALB that allows inbound HTTP/HTTPS (80/443) from the world, and a Security Group for EC2 instances that allows inbound traffic from the ALB’s group on the instance listener port (e.g., 8080 or 80). Also allow egress as needed. Terraform `aws_security_group` and `aws_security_group_rule` can define these.

**Launch Template**: This defines how to launch an EC2 instance of our API. For example:

```hcl
resource "aws_launch_template" "api" {
  name_prefix   = "api-server-"
  image_id      = "ami-0123456789abcdef0"  # (Use a real Linux AMI ID with Java or your app pre-baked)
  instance_type = "t3.small"
  key_name      = "my-keypair"            # if you need SSH
  security_group_names = [aws_security_group.api_sg.name]
  user_data = <<-EOF
              #!/bin/bash
              export JAVA_OPTS="-Xmx256m"
              aws s3 cp s3://mybucket/app.jar /home/ec2-user/app.jar
              java $JAVA_OPTS -jar /home/ec2-user/app.jar
              EOF
}
```

In this example, we assume the application JAR is in S3 and the instance pulls it on startup (or you could bake it into an AMI ahead of time). The user data script starts the Spring Boot app. In a real setup, you might use an AMI with the app or a Docker image (for ECS/EKS). Make sure the instance has an IAM role to fetch from S3 if you go that route.

**Auto Scaling Group**: Now we create an ASG that uses this launch template:

```hcl
resource "aws_autoscaling_group" "api_asg" {
  name_prefix          = "myapi-asg-"
  max_size             = 10
  min_size             = 2
  desired_capacity     = 2
  vpc_zone_identifier  = [aws_subnet.public1.id, aws_subnet.public2.id]
  launch_template {
    id      = aws_launch_template.api.id
    version = "$Latest"
  }
  target_group_arns    = [aws_lb_target_group.api_tg.arn]  # attach the ALB target group for health checks
  health_check_type    = "ELB"
  health_check_grace_period = 30
  tag {
    key                 = "Name"
    value               = "myapi-server"
    propagate_at_launch = true
  }
}
```

This ASG spans two subnets (for multi-AZ), starts with 2 instances, and can scale out to 10. It’s associated with a Target Group (defined below) for the load balancer, so instances will register and health-check through that. We use `health_check_type = "ELB"` so the ALB health status is used to determine instance health.

**Scaling Policies**: We can add policies to this ASG. A common approach is a **target tracking policy** on CPU usage. For example, to aim for ~70% CPU usage:

```hcl
resource "aws_autoscaling_policy" "cpu_target" {
  name                   = "cpu-target-tracking"
  autoscaling_group_name = aws_autoscaling_group.api_asg.name
  policy_type            = "TargetTrackingScaling"
  target_tracking_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ASGAverageCPUUtilization"
    }
    target_value = 70.0
  }
}
```

With this, AWS will adjust the ASG capacity to try to keep average CPU at 70%. If CPU goes beyond, it will scale out; if below, scale in (down to min_size). This dynamic scaling reacts to load in real-time ([EC2 Security Controls: Basic Dynamic Scaling for AWS Auto Scaling Group](https://asecure.cloud/a/p_basic_dynamic_scaling_for_aws_auto_scaling_group/#:~:text=target_tracking_configuration%20)). Alternatively, you can use step policies triggered by CloudWatch alarms (e.g., if CPU > 80% for 5 minutes then add 2 instances, if CPU < 30% then remove 1 instance, etc.), but target tracking is simpler.

**Load Balancer (ALB)**: We create an ALB to distribute requests to instances:

```hcl
resource "aws_lb" "api_alb" {
  name               = "myapi-alb"
  load_balancer_type = "application"
  subnets            = [aws_subnet.public1.id, aws_subnet.public2.id]
  security_groups    = [aws_security_group.alb_sg.id]
}

resource "aws_lb_target_group" "api_tg" {
  name     = "myapi-tg"
  port     = 8080
  protocol = "HTTP"
  vpc_id   = aws_vpc.main.id
  health_check {
    path                = "/actuator/health"  # assuming Spring Boot actuator health
    unhealthy_threshold = 3
    healthy_threshold   = 2
    interval            = 30
    timeout             = 5
  }
}
```

And a listener for the ALB:

```hcl
resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.api_alb.arn
  port              = 80
  protocol          = "HTTP"
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.api_tg.arn
  }
}
```

(This is for HTTP; for production, add an HTTPS listener on 443 with an SSL certificate from ACM. You might then redirect 80 to 443 or just use 443.)

Now our ALB will forward incoming requests to the EC2 instances on port 8080. Health checks ensure only healthy instances get traffic.

**CloudFront Distribution**: CloudFront will sit in front of the ALB to cache content globally and reduce latency. Even for API responses (which might be dynamic), CloudFront can improve performance by terminating TLS near the user and keeping persistent connections to the origin (ALB) ([API & dynamic acceleration](https://aws.amazon.com/developer/application-security-performance/articles/api-dynamic-acceleration/#:~:text=,and%20in%20%2074%20CloudFront)). We will configure CloudFront with no caching (or minimal caching) for API responses, unless certain endpoints are cacheable (e.g., a public GET that is the same for all users).

```hcl
resource "aws_cloudfront_origin_access_identity" "origin_oai" {
  comment = "OAI for ALB origin"
}

resource "aws_cloudfront_distribution" "api_cdn" {
  origin {
    domain_name = aws_lb.api_alb.dns_name
    origin_id   = "apiOrigin"
    custom_origin_config {
      origin_protocol_policy = "https-only"  # ALB supports HTTPS (recommended)
      http_port              = 80
      https_port             = 443
      origin_ssl_protocols   = ["TLSv1.2"]
    }
  }
  enabled             = true
  is_ipv6_enabled      = true
  comment             = "API distribution"
  default_root_object = "" 

  default_cache_behavior {
    allowed_methods  = ["GET", "HEAD", "OPTIONS", "PUT", "POST", "PATCH", "DELETE"]
    cached_methods   = ["GET", "HEAD", "OPTIONS"]  # only cache GET/HEAD by default
    target_origin_id = "apiOrigin"
    viewer_protocol_policy = "https-only"

    forwarded_values {
      query_string = true        # forward query params to origin
      headers      = ["Authorization", "Accept", "Content-Type"]  # forward auth header and others
      cookies {
        forward = "all"
      }
    }
    min_ttl                = 0
    default_ttl            = 0
    max_ttl                = 0
  }

  price_class = "PriceClass_100"
  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }

  viewer_certificate {
    acm_certificate_arn = var.acm_cert_arn  # ACM cert for your domain
    ssl_support_method  = "sni-only"
  }
}
```

In the above CloudFront config, we set caching TTLs to 0 for dynamic content (so CloudFront will always forward requests instead of caching, unless you add caching rules). We forward all HTTP methods (so APIs can use POST, PUT, etc.), and crucially, we forward the `Authorization` header and cookies so that JWTs or session cookies are sent to the origin. By default, CloudFront strips headers and cookies for cache efficiency ([API & dynamic acceleration](https://aws.amazon.com/developer/application-security-performance/articles/api-dynamic-acceleration/#:~:text=Managed%20Caching%20Policy%20in%20your,as%20reverse%20proxy%2C%20it%20does)), but we override to forward them (using `forwarded_values`). We could also use Cache Policy and Origin Request Policy (newer way) – e.g., use the *CachingDisabled* policy for this use case ([API & dynamic acceleration](https://aws.amazon.com/developer/application-security-performance/articles/api-dynamic-acceleration/#:~:text=To%20use%20CloudFront%20as%20a,by%20the%20IP%20of%20the)) – but the above explicit config works.

The distribution’s origin points to the ALB’s DNS name. We should also set up a Route53 DNS record for a friendly API domain (e.g., api.mydomain.com) that points to the CloudFront distribution (Alias record). This way clients hit CloudFront via a nice URL.

**Deploying with Terraform**: With all resources defined, run `terraform init`, `terraform plan`, and `terraform apply`. Terraform will create the VPC, ALB, ASG, etc. Once done, you will have:

- An ALB DNS (e.g., `myapi-alb-123.us-east-1.elb.amazonaws.com`).
- A CloudFront domain (e.g., `d111111abcdef8.cloudfront.net`).
- EC2 instances launching in the ASG, registering to the ALB target group.
- Auto-scaling policies in place.
- (If EKS is used, Terraform can create that too, which we cover later.)

At this point, infrastructure is up. Next, we need to deploy our application code (if not already baked in). If using the user_data pulling from S3 approach, ensure the S3 object is there. If using a custom AMI with the app or container images, those should be prepared (perhaps using a CI/CD pipeline or manual build).

Terraform is not the only IaC; CloudFormation or AWS CDK could also be used. But Terraform offers a unified approach if your stack includes multi-cloud or a preference for HCL syntax.

Now that our infrastructure is ready, let's look at how to implement the API itself in Spring Boot, and how it integrates with this environment.

## Implementing the API with Spring Boot: Examples

We will illustrate how to implement various API types (REST, GraphQL, WebSocket, gRPC) using Spring Boot. Spring Boot is a popular Java framework for building microservices and web applications, and it has support or extensions for all these API types. We'll also include how to handle authentication (JWT) in the code. These examples are simplified for clarity.

### REST API Example (Spring Web MVC)

For a RESTful API, you'd typically use Spring Web MVC (or Spring WebFlux for reactive). Using the annotation-based approach:

**Controller Example** – a simple REST controller with one GET endpoint:

```java
@RestController
@RequestMapping("/api")
public class ProductController {

    @GetMapping("/products/{id}")
    public ResponseEntity<Product> getProductById(@PathVariable Long id) {
        // Service layer call to fetch product (from DB, etc.)
        Product product = productService.findById(id);
        if (product != null) {
            return ResponseEntity.ok(product);
        } else {
            return ResponseEntity.notFound().build();
        }
    }

    @PostMapping("/products")
    public ResponseEntity<Product> createProduct(@RequestBody Product newProd) {
        // Save product and return created product with 201 Created status
        Product created = productService.save(newProd);
        URI location = URI.create("/api/products/" + created.getId());
        return ResponseEntity.created(location).body(created);
    }
}
```

This defines two endpoints: GET to retrieve a product by ID, and POST to create a new product. They return JSON (Spring Boot auto-converts POJOs to JSON via Jackson).

**JWT Security** – In Spring Boot, you can add Spring Security and configure it to require authentication for these endpoints. For instance, in a `WebSecurityConfigurerAdapter` (for Spring Security 5.x) or SecurityFilterChain (for Spring Security 6+):

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http.csrf().disable();
    http.authorizeHttpRequests(auth -> auth
            .requestMatchers("/api/auth/**").permitAll()  // allow auth endpoints if any
            .anyRequest().authenticated()
        )
        .oauth2ResourceServer(oauth2 -> oauth2.jwt()); // assuming using JWT as OAuth2 Resource Server

    return http.build();
}
```

The above config assumes JWT decoding is set up (e.g., in `application.yml` you have issuer URI for Cognito or your JWT provider). Spring Security will then automatically reject requests with no/invalid JWT. If valid, you can access the user's claims in controllers via `@AuthenticationPrincipal Jwt jwt` parameter or via the `SecurityContext`.

Alternatively, you could manually implement a filter that validates the `Authorization` header (using a JWT library) and sets `SecurityContextHolder`. But leveraging Spring’s built-in support is easier.

**Testing** – Once deployed, hitting the REST endpoints (through CloudFront domain or ALB directly) should return JSON. For example, `GET https://<cloudfront-domain>/api/products/1` should go through CloudFront -> ALB -> Spring Boot app and return a product JSON. If using JWT auth, include `Authorization: Bearer <token>` header. Without it, you'd get 401 Unauthorized.

This REST setup is stateless (no HTTP session used), so it will scale across instances seamlessly.

### GraphQL API Example (Spring for GraphQL)

Spring Boot has integration for GraphQL via the **Spring for GraphQL** project (which uses GraphQL Java under the hood). To implement a GraphQL endpoint:

- Define a GraphQL schema (e.g., in a `schema.graphqls` file on the classpath).
- Use controllers or resolvers to handle queries and mutations.

**Schema Example** (`schema.graphqls`):

```graphql
type Product {
    id: ID!
    name: String
    price: Float
}
type Query {
    product(id: ID!): Product
    products: [Product]
}
type Mutation {
    addProduct(name: String!, price: Float!): Product
}
```

**Resolver Example** – using Spring’s annotation-based GraphQL support:

```java
@GraphQlController
public class ProductGraphQLController {

    @QueryMapping
    public Product product(@Argument Long id) {
        return productService.findById(id);
    }

    @QueryMapping
    public List<Product> products() {
        return productService.findAll();
    }

    @MutationMapping
    public Product addProduct(@Argument String name, @Argument Float price) {
        return productService.create(name, price);
    }
}
```

With Spring for GraphQL, the `@QueryMapping` on a method named `product` automatically maps to the `product` query defined in the schema, and so on. The framework will expose a GraphQL endpoint (by default at `/graphql`). You can test it with GraphiQL or a curl POST. The endpoint expects a JSON payload with a GraphQL query.

**Security**: You can secure GraphQL similar to REST. The GraphQL endpoint is just one URL (`/graphql`), so you secure that. For example, require authentication on `/graphql` and then within resolvers, you can get the authentication info if needed. E.g., use `@AuthenticationPrincipal` in the controller to get user details for access control, or use a DataFetcher environment to get context.

GraphQL being stateless here (assuming each query is independent and carries any needed auth token in header) means it will scale like REST. One consideration: queries can be expensive. Monitor query performance and perhaps put limits (max depth, complexity) if needed to prevent abuse that could affect stability.

### WebSocket API Example (Spring WebSocket with STOMP)

Spring Boot supports WebSockets with a messaging architecture (using STOMP over WebSocket). A typical setup uses an in-memory broker (for simple cases) or RabbitMQ/Redis for scale (for multiple instance coordination).

**Configuration**:

```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws")  // WebSocket handshake endpoint
                .setAllowedOriginPatterns("*")
                .withSockJS();      // fallback to SockJS for older browsers
    }
    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.enableSimpleBroker("/topic", "/queue");  // simple in-memory broker
        config.setApplicationDestinationPrefixes("/app");
    }
}
```

This sets up a WebSocket endpoint at `/ws` that clients will connect to. The client will use a STOMP client to subscribe and send messages. The server here uses a simple broker (suitable for a single instance or a few). In a multi-instance scenario, the simple broker won’t share messages across instances; you’d use something like Spring Cloud AWS or Redis as a message broker to distribute messages.

**Controller**:

```java
@Controller
public class ChatController {

    @MessageMapping("/chat")        // listens for messages sent to /app/chat
    @SendTo("/topic/messages")     // broadcasts to subscribers of /topic/messages
    public ChatMessage send(ChatMessage message) {
        // Possibly add server-side timestamp or ID
        message.setTimestamp(Instant.now());
        return message;
    }
}
```

With this, if a client sends a STOMP message to `/app/chat` (e.g., a chat message), the server will broadcast it to all subscribed clients on `/topic/messages`. Clients connect via WebSocket to `/ws`, then subscribe to the topic.

**Security**: You can integrate Spring Security to secure WebSockets. For example, require an auth token during the WebSocket handshake. One way is to add an HTTP filter that intercepts the handshake request (which is an HTTP upgrade) and authenticate it. Another is using a channel interceptor in Spring’s messaging that checks a token header. If using JWT, the client can include the JWT in the WebSocket CONNECT message (like a header), and the server validates it and sets the user principal. Spring Security’s `@AuthenticationPrincipal` can then work inside message handling methods to identify the user sending the message. Additionally, you might restrict subscriptions or message destinations by user roles.

**Scaling WebSocket**: If you run multiple instances of this Spring Boot app, a client will connect to one of them via ALB. The simple broker in each instance only knows about its own clients. That means if a user on Instance A sends a message, only clients connected to A will see it. To fix that, use a centralized message broker. Spring’s `config.enableStompBrokerRelay(...)` can connect to a RabbitMQ or ActiveMQ, so all instances relay through a central broker. Alternatively, use Redis pub/sub via Spring Integration. This way, Instance A can send a message that the broker forwards to Instance B’s clients.

This ensures a truly scalable WebSocket setup: you can add instances, and all clients still get the messages. The trade-off is the external broker (RabbitMQ/Redis) becomes a critical component to scale/monitor as well.

However, if your WebSocket use-case doesn’t require cross-instance messaging (e.g., real-time updates only relevant to that user), you might be fine with independent instances. For example, a stock price update feed could be published from each instance independently to its connected clients (assuming each instance gets the data feed). But for chat, collaborative apps, etc., you want a shared message bus.

We mention WebSocket here to highlight how stateful interactions can be managed. It is absolutely possible to scale WebSocket apps on AWS (the ALB + Auto Scaling Group approach, or using AWS API Gateway WebSocket which offloads connections). Just design for the sticky nature of connections.

### gRPC API Example (Spring + gRPC)

Spring doesn’t natively include gRPC support out-of-the-box, but there are community starters (e.g., yidongnan’s grpc-spring-boot-starter) that make it easy. Alternatively, you can run gRPC in a plain Java service and still package it with Spring Boot (just not using Spring MVC).

**Defining the service**: You start by writing a `.proto` file:

```protobuf
syntax = "proto3";
option java_multiple_files = true;
option java_package = "com.example.demo";
option java_outer_classname = "ProductServiceProto";

service ProductService {
  rpc GetProduct(ProductRequest) returns (ProductResponse);
  rpc ListProducts(Empty) returns (ProductList);
}

message ProductRequest {
  int64 id = 1;
}
message ProductResponse {
  Product product = 1;
}
message ProductList {
  repeated Product products = 1;
}
message Product {
  int64 id = 1;
  string name = 2;
  double price = 3;
}
```

You would compile this with the protobuf Gradle plugin or Maven plugin to generate Java classes (including `ProductServiceGrpc.ProductServiceImplBase` to be extended for server implementation).

**Service Implementation**:

```java
@Service  // using Spring stereotype for component scan
public class ProductServiceImpl extends ProductServiceGrpc.ProductServiceImplBase {
    @Autowired
    private ProductRepository productRepo;

    @Override
    public void getProduct(ProductRequest request, StreamObserver<ProductResponse> responseObserver) {
        long id = request.getId();
        Product product = productRepo.findById(id);
        ProductResponse response;
        if (product != null) {
            response = ProductResponse.newBuilder()
                        .setProduct(convertToProto(product))
                        .build();
        } else {
            response = ProductResponse.newBuilder().build(); // empty or handle not found
        }
        responseObserver.onNext(response);
        responseObserver.onCompleted();
    }

    @Override
    public void listProducts(Empty request, StreamObserver<ProductList> responseObserver) {
        List<Product> all = productRepo.findAll();
        ProductList.Builder listBuilder = ProductList.newBuilder();
        all.forEach(prod -> listBuilder.addProducts(convertToProto(prod)));
        responseObserver.onNext(listBuilder.build());
        responseObserver.onCompleted();
    }

    private ProductProto convertToProto(Product p) {
        // convert entity to protobuf-generated Product message
        return ProductProto.newBuilder()
                 .setId(p.getId())
                 .setName(p.getName())
                 .setPrice(p.getPrice())
                 .build();
    }
}
```

This uses a hypothetical `ProductRepository` (could be Spring Data JPA or any data source). The service extends the gRPC base class and implements the RPC methods, writing responses via the `StreamObserver`. If you use the grpc-spring-boot-starter, it might automatically detect and start the gRPC server with this service.

**Server startup**: If not using a starter, you’d manually start a `Server` in your Spring Boot application, for example in `Application.run`:

```java
public static void main(String[] args) {
    ConfigurableApplicationContext context = SpringApplication.run(App.class, args);
    Server server = ServerBuilder.forPort(9090)
                    .addService(context.getBean(ProductServiceImpl.class))
                    .build()
                    .start();
    Runtime.getRuntime().addShutdownHook(new Thread(() -> {
        server.shutdown();
    }));
    server.awaitTermination();
}
```

This starts gRPC on port 9090. The ALB can be configured with a TCP or HTTP/2 listener on that port (ALB supports HTTP/2 for gRPC). We could also use an NLB if we wanted at L4.

**Client**: To test, a client would use the generated stub (e.g., `ProductServiceGrpc.newBlockingStub(channel)` etc.) in a Java program, or use `grpcurl` command-line tool for ad-hoc testing.

**Auth in gRPC**: gRPC can use tokens as metadata. For example, the client can send `Authorization: Bearer <token>` as a metadata header. The server, if using an interceptor, can intercept metadata and verify JWT similar to HTTP. In a Spring context, you could have a ServerInterceptor that checks for an auth header in the `ServerCall` and performs JWT validation (maybe using the same logic as your REST security). There’s no built-in Spring Security for gRPC, but you can integrate it manually or use the grpc-spring-boot-starter’s security features if available.

**Scaling**: Scaling gRPC is similar to REST – stateless calls scale easily. However, note that if a client maintains a long HTTP/2 connection and keeps making calls, that connection sticks to one server instance (LB won’t rebalance mid-connection). If that instance becomes hot, the client doesn’t automatically switch. But HTTP/2 does allow concurrency on one connection, so the client might not need to open multiple. For better load distribution, one approach is to have clients open multiple connections (some gRPC client libraries do this by default). From the server side, you scale by running more instances behind the LB. If one instance fails, the connection drops and client reconnects (possibly to another). So resilience and retry logic on the client side is important in gRPC scenarios.

All these API implementations (REST/GraphQL/WebSocket/gRPC) can coexist in a single Spring Boot application if needed, but typically you’d expose one style primarily. You could for example have REST and WebSocket in one app (common for an app that offers both HTTP APIs and a real-time update channel). gRPC might be separate (maybe just for internal microservice calls). GraphQL might replace some REST endpoints or be offered alongside.

Now that we have code running, let’s connect it back to AWS services for scaling and distribution.

## Integrating with AWS Auto Scaling and CloudFront

At this point, we have:

- A Spring Boot application (packaged as JAR or container) that can run our API.
- An AWS environment (ASG, ALB, CloudFront) ready to host it.

Integration mostly involves **deployment** and **configuration alignment**:

1. **Deploy application on EC2 instances**: If using the Launch Template user data to download and run the JAR, ensure the S3 bucket and object are correct and accessible (the instance’s IAM role should allow an S3 get). Alternatively, bake the app into an AMI (using Packer or an AWS AMI pipeline) and have the Launch Template use that AMI so instances come up with the app ready. In a container scenario, you'd have EC2 run Docker and pull an image from ECR. There are many ways to bootstrap instances – choose one that fits your workflow.
2. **Auto Scaling Group behavior**: The ASG will launch the number of instances we set (min 2, desired 2, for example). It will register them with the ALB’s target group. The ALB will start health-checking (hitting `/actuator/health` or whichever health endpoint you set). Only once an instance reports healthy will ALB send traffic. If an instance is not healthy (or during deploy if the app takes time to start), the health check will fail and ASG might replace the instance thinking it’s faulty. So tune the `health_check_grace_period` (we put 30 seconds in Terraform) to give the app time to start up.
3. **CloudFront origin**: The CloudFront distribution needs to know the ALB domain. After Terraform apply, there might be a slight lag until the ALB DNS is resolvable publicly (usually available quickly). CloudFront distribution creation might take ~15 minutes to deploy to all edge locations. Once it’s deployed, you should be able to hit the CloudFront URL and get a response from your API. For example, `https://d111111abcdef8.cloudfront.net/api/products/1` should route through. If you set up a custom domain and certificate, use that domain for API calls.
4. **Test end-to-end**: Try to call a GET endpoint through CloudFront. CloudFront should forward it to ALB (over HTTPS if configured) and ALB to an instance. The response comes back, CloudFront caches if allowed (we set no-caching for dynamic paths). Check CloudFront logs or stats to ensure requests are being served. Also test with authentication: e.g., login via Cognito to get a JWT, then call the API with `Authorization` header. CloudFront by default doesn’t cache responses when Authorization header is forwarded (and we've configured it to forward that header), so each request should go to origin (which is what we want for authenticated content).
5. **Auto Scaling in action**: To see ASG auto-scaling, you can simulate load. For example, use a tool (Apache JMeter, Artillery, or a simple script) to send many requests or heavy load to the API through CloudFront. CloudFront will pass them along (maybe making many origin connections if needed). The EC2 instances' CloudWatch metrics (like CPU) will rise. If average CPU goes above our 70% target, within a few minutes the ASG should scale out (launch new instances). These new instances register with ALB, become healthy, and start serving traffic. The load is then spread out more, hopefully reducing CPU on each instance. This demonstrates scaling out. Later, if traffic subsides, ASG will scale in (terminate extra instances) to save cost, while still meeting the min capacity.
6. **Sticky Sessions / Sessions**: We have not enabled any explicit sticky session on ALB for the REST/GraphQL/gRPC traffic, because our design is stateless (each request independent). ALB by default is round-robin (can be weighted round-robin if we had weights). WebSocket connections are effectively sticky naturally (they stick to one instance for the duration). If we had a requirement for sticky sessions (say we were using HTTP sessions in a stateful way), we could enable ALB stickiness. ALB supports two types: application-controlled (if app sets a cookie) or load-balancer-generated cookie stickiness. The latter is easiest – ALB will issue a cookie like `AWSALB` and the same client will be sent to the same target for the duration. For example, ALB can be configured with `stickiness.enabled = true` and it will keep a client on one instance ([Sticky sessions with load balancer generated cookies - AWS Prescriptive Guidance](https://docs.aws.amazon.com/prescriptive-guidance/latest/load-balancer-stickiness/alb-cookies-stickiness.html#:~:text=After%20traffic%20has%20been%20initially,instance%20for%20a%20specified%20duration)). But since we've designed for stateless JWT auth, we **do not need ALB sticky sessions** for our API. Avoiding stickiness generally improves load distribution.
7. **CloudFront considerations**: CloudFront gives us some added benefits:
   - It terminates the TLS closer to user (at edge), reducing latency of TLS handshake for repeated connections.
   - It can reuse connections to ALB (keep-alive), which reduces connection churn on ALB/instances ([API & dynamic acceleration](https://aws.amazon.com/developer/application-security-performance/articles/api-dynamic-acceleration/#:~:text=or%20when%20it%20is%20purely,because%20the%20DTO%20overhead%20of)). This can improve performance and reduce load on the servers, especially if many short-lived requests.
   - It provides a layer of DDoS protection and the ability to use AWS WAF. If we attach an AWS WAF web ACL to the CloudFront distribution, we can block common attacks (SQL injection, XSS, etc.) at the edge ([Web Application Firewall, Web API Protection - AWS WAF](https://aws.amazon.com/waf/#:~:text=Web%20Application%20Firewall%2C%20Web%20API,site%20scripting%20%28XSS)). This prevents malicious traffic from ever reaching our origin, improving security and stability.
   - We can enable logging at CloudFront and ALB to analyze traffic patterns.
   - If our users are global, CloudFront’s network will speed up responses by caching static content and moving dynamic content faster (over AWS backbone).
   - Cost: CloudFront has its own pricing, but often it can lower overall cost by caching or reducing origin data transfer in some cases. We configured no caching for dynamic content, but CloudFront still helps with TLS and global routing. For assets (if our app had static assets or an associated website), CloudFront **would** cache those.

One thing to watch: if CloudFront is configured to forward all headers (as we did for auth), it might reduce caching. But since API responses are mostly non-cacheable per-user content, that’s fine.

**Logging and Monitoring**: Use CloudWatch. ASG and EC2 and ALB all publish metrics. Set alarms if desired (e.g., on high error rates or high latency on ALB). CloudFront has metrics in CloudWatch CDN namespace. Enable ALB access logs (to S3) and CloudFront access logs (to S3 or CloudWatch) for audit.

Now, let's consider an alternative deployment using Kubernetes, and how auto-scaling is handled in that scenario.

## Scaling with Kubernetes on AWS (EKS)

Instead of (or in addition to) using raw EC2 Auto Scaling Groups, many teams deploy their applications on **Kubernetes** for portability and easier container management. AWS offers **EKS (Elastic Kubernetes Service)**, a managed Kubernetes control plane. With EKS, you still need worker nodes (which can be EC2 instances in an Auto Scaling Group, or AWS Fargate for serverless nodes). We will focus on the EC2-backed approach, as it closely parallels our earlier setup, but note that Fargate can eliminate the need to manage nodes (though with some limitations and cost differences).

### Deploying the Application to EKS

Assume we containerized our Spring Boot application (built a Docker image and pushed to ECR). To run it on EKS:

1. **EKS Cluster**: Create an EKS cluster (via Terraform, CloudFormation, or EKS CTL). This sets up control plane (managed by AWS). Then create a **Node Group** (this is an Auto Scaling group of EC2s configured to join the cluster). The Node Group can scale nodes up/down; EKS can integrate with Cluster Autoscaler for that (explained shortly).
2. **Kubernetes manifests**: Write a Deployment for the app, a Service, and perhaps an Ingress.
   - **Deployment** defines the desired number of pods (replicas) for the app and what container image to run. For example:
     ```yaml
     apiVersion: apps/v1
     kind: Deployment
     metadata:
       name: myapi-deployment
     spec:
       replicas: 2
       selector:
         matchLabels:
           app: myapi
       template:
         metadata:
           labels:
             app: myapi
         spec:
           containers:
           - name: myapi
             image: 123456789.dkr.ecr.us-east-1.amazonaws.com/myapi:latest
             ports:
             - containerPort: 8080
             env:
             - name: SPRING_PROFILES_ACTIVE
               value: "prod"
             - name: COGNITO_POOL_ID
               value: "<your user pool id>"
             # ... other env vars like DB creds or JWT secret if using self-managed JWT
     ```
   - **Service**: If we want the app accessible, we create a Service of type LoadBalancer or use an Ingress. A Service type LoadBalancer will provision an AWS ELB (actually an NLB or ALB depending on config; EKS favors NLB by default for TCP or can use ALB via the ALB Ingress Controller for HTTP). Suppose we use an ALB Ingress (preferred for HTTP, as it integrates well with path-based routing and TLS).
   - **Ingress**: Using AWS Load Balancer Controller (an ingress controller), you can define an Ingress that creates an ALB and Target Group for your service. For example:
     ```yaml
     apiVersion: networking.k8s.io/v1
     kind: Ingress
     metadata:
       name: myapi-ingress
       annotations:
         kubernetes.io/ingress.class: alb
         alb.ingress.kubernetes.io/scheme: internet-facing
         alb.ingress.kubernetes.io/listen-ports: '[{"HTTP":80},{"HTTPS":443}]'
         alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-1:... (your ACM ARN)
         alb.ingress.kubernetes.io/actions.ssl-redirect: >
           {"Type": "redirect", "RedirectConfig": { "Protocol": "HTTPS", "Port": "443", "StatusCode": "HTTP_301"}}
         alb.ingress.kubernetes.io/auth-type: cognito
         alb.ingress.kubernetes.io/auth-cognito-user-pool-id: <user-pool-id>
         alb.ingress.kubernetes.io/auth-cognito-user-pool-client-id: <app-client-id>
         alb.ingress.kubernetes.io/auth-cognito-user-pool-domain: <domain-prefix>.auth.us-east-1.amazoncognito.com
     spec:
       rules:
         - host: api.mydomain.com
           http:
             paths:
               - path: /* 
                 pathType: ImplementationSpecific
                 backend:
                   service:
                     name: myapi-service
                     port:
                       number: 8080
     ```
     This example uses ALB Ingress with Cognito authentication built-in (notice the annotations for auth-cognito: the ALB will perform Cognito OAuth flow, requiring users to login before accessing the API – this is another approach to secure APIs without coding it in the app). The ALB ingress controller will create an ALB, target group, etc., similar to our manual Terraform earlier, but managed by K8s resources.
   - If we didn’t want to use Cognito at the ALB layer, we could omit those auth annotations, and then the app would handle JWTs as before. Either way works.
3. **Horizontal Pod Autoscaler (HPA)**: Kubernetes has the HorizontalPodAutoscaler resource to scale pods based on metrics. You could deploy an HPA for the deployment:
   ```yaml
   apiVersion: autoscaling/v2
   kind: HorizontalPodAutoscaler
   metadata:
     name: myapi-hpa
   spec:
     scaleTargetRef:
       apiVersion: apps/v1
       kind: Deployment
       name: myapi-deployment
     minReplicas: 2
     maxReplicas: 10
     metrics:
     - type: Resource
       resource:
         name: cpu
         target:
           type: Utilization
           averageUtilization: 50
   ```
   This tells K8s to monitor CPU usage of the pods and keep it around 50%. It will add or remove pods within [2,10] replicas to achieve that ([Scale pod deployments with Horizontal Pod Autoscaler - Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/horizontal-pod-autoscaler.html#:~:text=)). This is very similar to the ASG target tracking but at the container level. You need the Metrics Server installed on EKS for this to work (EKS by default might have it, or you can install).
   
   HPA can also scale on custom metrics (like requests per second, or memory, etc.) if you have metrics adapters, but CPU is common.
4. **Cluster Autoscaler (CA)**: If HPA adds pods beyond current node capacity, you need more nodes. The **Cluster Autoscaler** is a component that watches for unscheduled pods and scales the node group. AWS’s EKS can use the standard Kubernetes Cluster Autoscaler (deployed as a Deployment in kube-system). You give it permissions to adjust the Auto Scaling Group of your nodes. When it sees pending pods (due to insufficient resources), it will request the ASG to scale out ([autoscaler/cluster-autoscaler/FAQ.md at master · kubernetes/autoscaler · GitHub](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#:~:text=Cluster%20Autoscaler%20increases%20the%20size,of%20the%20cluster%20when)). Likewise, if nodes are underutilized (and pods can be moved), it will scale in. Cluster Autoscaler ensures you have just enough EC2 instances to meet the cluster's needs. This is how Kubernetes ties into AWS auto scaling:
   - The node group ASG should have a range (min, max nodes). CA will not go beyond that max.
   - You tag or name your ASG so CA knows it (like using Auto Discovery tags: `k8s.io/cluster-autoscaler/<cluster-name>: owned` and `k8s.io/cluster-autoscaler/enabled: true`).
   - Install CA with those parameters (either via Helm or manifest from the official cluster-autoscaler project).
   
   With HPA + CA, your pods scale out, then nodes scale out to accommodate, achieving a two-level scaling. HPA reacts faster (within tens of seconds), CA might take a minute or more to add a node (similar to launching a new EC2).
5. **Stateful considerations**: If your app is stateless, HPA works great. If each pod was keeping unique state (not recommended), moving or scaling pods would be an issue. Kubernetes also has StatefulSets for stateful apps (like databases) where identities are persistent, but for our API, we’d stick to Deployments.
   - **Sticky sessions**: If using ALB Ingress, by default it’s round-robin, but you can enable stickiness on the Service or Ingress via annotations (e.g., `alb.ingress.kubernetes.io/target-group-attributes: stickiness.enabled=true, stickiness.lb_cookie.duration_seconds=300`). But again, strive to avoid needing that.
   - WebSockets in K8s: ALB Ingress supports WebSocket upgrade, so that should work out of the box. Just ensure the Ingress `timeout` is sufficient to keep connections (ALB has a default idle timeout of 60s, you might raise it for websockets).
   - gRPC in K8s: Also supported if ALB Ingress (which now supports HTTP/2). Or use an NLB for gRPC with headless service. ALB is simpler for HTTP/2 though.

**Monitoring in Kubernetes**: Use Prometheus for app metrics if needed, and CloudWatch Container Insights for cluster metrics, or AWS CloudWatch metrics for EKS (it can pull scheduler metrics etc.). You’ll monitor HPA events (e.g., `kubectl get hpa` to see status) and CA logs to ensure they act as expected.

**Rolling Updates**: With Deployment, any new version of the app can be rolled out gradually, which is another advantage: you can deploy new container image, Kubernetes will phase out old pods, bring in new, without downtime (if configured properly). With ASGs, you can achieve rolling updates too (e.g., using launch template version and increasing new instances, then terminating old – but it’s a bit more manual unless you use tools like CodeDeploy or Asg Rolling update hooks).

In short, EKS provides a powerful environment for scaling, with HPA handling scaling at the pod level (similar to ASG instance scaling) ([Scale pod deployments with Horizontal Pod Autoscaler - Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/horizontal-pod-autoscaler.html#:~:text=The%20Kubernetes%20Horizontal%20Pod%20Autoscaler,try%20to%20meet%20that%20target)) and cluster autoscaler handling infrastructure scaling ([autoscaler/cluster-autoscaler/FAQ.md at master · kubernetes/autoscaler · GitHub](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#:~:text=Cluster%20Autoscaler%20increases%20the%20size,of%20the%20cluster%20when)). It adds some complexity (managing a cluster), but if you already use Kubernetes, it integrates well with AWS auto scaling.

## Stateful vs Stateless Workloads (Sticky Sessions and Session Management)

Throughout this guide, we've emphasized **stateless** design for easier scaling. It's important to clearly distinguish these concepts:

- **Stateless Application**: The server does not retain information or context about the user between requests. Each request stands on its own, containing all necessary data (e.g., credentials, parameters) to be processed ([ REST vs. GraphQL vs. gRPC vs. WebSocket](https://www.resolutesoftware.com/blog/rest-vs-graphql-vs-grpc-vs-websocket/#:~:text=1,context%20information%20on%20the%20server)). Examples: authenticating each request via a token, storing data in a database rather than memory. Stateless apps can be freely duplicated behind load balancers – one request can go to Server A and the next to Server B without any issue or loss of continuity.
- **Stateful Application**: The server maintains session state or other context for a user. For example, after a user logs in, the server stores their session data in memory (or a local cache) and expects subsequent requests from that user to hit the same server to retrieve that session. If a different server gets the request, it won't find the session and the user might be logged out or see inconsistent data. This **ties the user to a specific server**, which complicates scaling and fault tolerance.

**Sticky Sessions** (Session Affinity) is a technique to support stateful apps by instructing the load balancer to consistently route a user's requests to the same backend instance. AWS load balancers accomplish this via cookies. For ALB, the default stickiness uses a load-balancer generated cookie (named `AWSALB` or similar) that encodes the target info. Once a client gets that cookie, the ALB will send them to the same target as long as the cookie is valid ([Sticky sessions with load balancer generated cookies - AWS Prescriptive Guidance](https://docs.aws.amazon.com/prescriptive-guidance/latest/load-balancer-stickiness/alb-cookies-stickiness.html#:~:text=After%20traffic%20has%20been%20initially,instance%20for%20a%20specified%20duration)). AWS also supports application-controlled stickiness (your app sets a custom cookie that the LB uses to route). Common uses for sticky sessions are things like classic web apps that use in-memory session objects, or any scenario where **session data is cached locally for performance** ([Choosing a stickiness strategy for your load balancer - AWS Prescriptive Guidance](https://docs.aws.amazon.com/prescriptive-guidance/latest/load-balancer-stickiness/welcome.html#:~:text=,cookies%20and%20load%20balancer%20cookies)).

**Downsides of Sticky Sessions**:
- If the instance holding a session goes down, that user's session is lost (unless you have some session replication, which is complex).
- It can lead to uneven load: one server might accumulate many sticky sessions (say one very active user per session) and overload while others are idle.
- Scaling in (reducing instances) is hard – you might drop users on that instance.
- In cloud auto-scaling, instances may come and go, which could disrupt stickiness or user experience.
- It couples your horizontal scaling – you can’t freely route traffic.

**Best Practice**: Design your services to be stateless whenever possible. This often means **externalizing state**:
- Use a **database** or **cache** to store data that persists between requests. For example, user profile, shopping cart items, etc., stored in a database table keyed by user.
- Use **JWT tokens** to keep track of authentication state (as discussed) instead of server sessions.
- For per-session data that’s too heavy for a token, use a distributed cache (like Redis or DynamoDB) to store session info, and use a session ID cookie. This way, any server can load the session from the central store. Frameworks like Spring Session can automate storing HTTP session data in Redis.
- If you have to maintain **in-memory state** for a user (like a multi-request transaction context), consider if you can redesign to avoid that, or ensure the client holds the needed state and sends it each time.

**When stateful is necessary**:
Some applications are inherently stateful, e.g., a multiplayer game server, or certain long workflow processes. In those cases, you'll likely partition users to specific servers intentionally (not just randomly load balanced). This is beyond typical stateless web APIs. If you must scale stateful workloads, you might use consistent hashing to route users to the same cluster partition, or sticky load balancing with careful monitoring.

For our typical web API scenario, state can often be confined to these categories:
- **Authentication state**: solved by tokens (stateless) or external session stores.
- **Caching**: e.g., caching user-specific data. This can be done with a distributed cache so all instances share it, rather than each building its own cache that might be inconsistent.
- **Files/user uploads**: if an instance handles a file upload and stores it locally, another instance won't see it. Solution: store in S3 or shared storage accessible by all.
- **Background jobs**: if one instance picks up a job, ensure others don't duplicate it (use a queue or distributed lock).
- **WebSocket connections**: as discussed, each connection is tied to a server. To scale, externalize the messages via a message broker so all servers can communicate needed info.

**Session Management Strategies**:
- **Client-side sessions (JWT)**: Already covered; stateless, scalable. Only downside is you cannot force logout easily (you can blacklist tokens, but that requires state somewhere or short expiry).
- **Server-side sessions with database**: e.g., store a session row in DynamoDB or RDS for each user. This is simple but could be slower (DB hit each request). Often done for small scale, but at high scale the DB could be a bottleneck unless it's very optimized (e.g., key-value store).
- **Server-side sessions with cache (Redis/Memcached)**: Very popular. Use a fast in-memory store accessible by all instances. This avoids database overhead and can scale (Redis cluster if needed). For example, in Spring Boot, use Spring Session with Redis – it will replace the default in-memory session storage with Redis. Now you can still use `HttpSession` in your code if needed, but it's not tying users to a server. As one reference puts it: *"Redis makes an excellent store for session data - it's fast and durable, and allows us to scale system components horizontally by adding more instances"* ([Scaling an Express Application with Redis as a Session Store](https://redis.io/learn/develop/node/nodecrashcourse/sessionstorage#:~:text=Fortunately%2C%20Redis%20makes%20an%20excellent,Checkin%20Receiver%20services%2C%20with%20minimal)).
- **Hybrid**: Some use JWT for auth but still have a server session for other data – try to consolidate to one approach to reduce complexity.

**Sticky vs stateless**: We should emphasize, stateless is preferred for scalability. Use sticky sessions only if you have no alternative and understand the drawbacks. AWS’s own guidance is to use stickiness for specific use cases (like quick-and-dirty caching on instance, or truly stateful legacy apps) ([Choosing a stickiness strategy for your load balancer - AWS Prescriptive Guidance](https://docs.aws.amazon.com/prescriptive-guidance/latest/load-balancer-stickiness/welcome.html#:~:text=Stickiness%20is%20a%20term%20that,available%20to%20the%20other%20server)), and even then to consider the two modes (application-controlled vs load-balancer cookie) based on needs.

### Security Best Practices

Security is a broad topic, but in context of scaling and AWS, some best practices include:

- **Use AWS WAF or similar** to shield your API from common attacks and abuse ([Web Application Firewall, Web API Protection - AWS WAF](https://aws.amazon.com/waf/#:~:text=Web%20Application%20Firewall%2C%20Web%20API,site%20scripting%20%28XSS)). This not only protects data, but also prevents malicious traffic from consuming resources (which could affect availability).
- **Encrypt everything**: Terminate HTTPS at the ALB or CloudFront. Use HTTPS between CloudFront and ALB (we did via origin protocol policy). Encrypt data at rest (RDS encryption, etc.). Use Cognito or KMS to manage secrets if needed.
- **Least privilege IAM**: If your EC2 instances need to access S3 or other AWS services (for example, for pulling code or writing logs), give them only the permissions required via an IAM role. Same for EKS: use IAM Roles for Service Accounts or other fine-grained access so that a compromised pod doesn’t have broad rights.
- **Security groups & Network ACLs**: We set up SGs to allow only necessary traffic (e.g., ALB to instances on the app port). Do not open unnecessary ports. Use private subnets for instances if possible (and only ALB in public).
- **Patching and AMI updates**: Regularly update the base AMI or container image to include security patches. If using AWS Linux or similar, consider automated patching or replace instances frequently (some orgs do immutable infrastructure: deploy new instances with new code and updated OS, then terminate old ones).
- **Monitoring and Alerting**: Set up CloudWatch Alarms for unusual patterns (spikes in 5xx errors could indicate attacks or issues, spikes in latency might indicate overload). Use AWS CloudTrail to audit any changes in the environment.
- **Pen testing and fuzz testing**: In staging, simulate high loads and malicious inputs. Ensure the system remains stable and secure under stress.
- **Graceful failure**: If load becomes too high, have timeouts and graceful degradation (circuit breakers) in place rather than total collapse. For instance, if DB is slow, use a timeout in your app so threads don’t all hang; this prevents one slow component from wrecking all servers which then auto-scale wildly.

In context of our Spring Boot app, applying Spring Security properly covers many issues (it can handle things like CSRF, CORS, etc.). Also validate inputs (especially if not using WAF). Use libraries to guard against SQL injection or other injection if not using an ORM.

Finally, keep in mind the **principle of least astonishment** for users: scaling should be transparent (except they notice better performance). A poorly managed state or auth could lead to users being randomly logged out or seeing inconsistent data. By following the stateless approach and robust auth, we avoid those issues.

## Complete Example: Putting it All Together

Let’s walk through a concrete scenario combining everything we’ve discussed:

**Scenario**: You are building a global e-commerce REST API using Spring Boot. It has endpoints for products, orders, user profiles, etc. Users must be authenticated (we’ll use AWS Cognito for user management and JWTs for auth). The API needs to handle high traffic during sales and scale accordingly. It should also provide near real-time order status updates to clients (we’ll use WebSockets for that part). We want a highly available deployment on AWS, using infrastructure as code and modern DevOps practices.

**Architecture**: The final architecture will look like this:

```mermaid
flowchart LR
    subgraph Client
      U[User Agent]\n(Web/Mobile App)
    end
    subgraph AWS Cloud
      CF[Amazon CloudFront]
      ALB[Application Load Balancer]
      ASG[(Auto Scaling Group\nEC2 Instances)]
      EKScluster[(EKS Cluster\nPods)]
      Cognito[(AWS Cognito User Pool)]
      DB[(Database)]
      Redis[(Session/Cache Store)]
    end
    U -->|Login| Cognito
    Cognito -->|JWT Tokens| U
    U -->|API Requests\n(with JWT)| CF
    CF --> ALB
    ALB --> ASG
    ALB --> EKScluster
    ASG --> DB
    EKScluster --> DB
    ASG --> Redis
    EKScluster --> Redis
```

*(Diagram: A user logs in to Cognito and receives a JWT. The user then makes API requests with the JWT. The requests go through CloudFront (edge), to ALB. The ALB distributes to either EC2 instances in an ASG or pods in EKS (depending on deployment scenario; one or the other, or both for hybrid). Those app servers talk to a database and use Redis for caching. Cognito is external for auth.)*

Let's break down each component and step in this flow:

1. **User Authentication**: The user opens the app (could be a single-page app or mobile). The app directs the user to Cognito Hosted UI (or uses Amplify libraries) for login. The user authenticates (Cognito checks password, possibly MFA). After auth, Cognito redirects back to the app with tokens (ID & Access JWT). Now the app stores the Access JWT (e.g., in memory or secure storage).
2. **API Request**: The app needs to fetch the user's profile and list of products. It calls `GET https://api.mydomain.com/user/profile` with header `Authorization: Bearer <JWT>`. The DNS `api.mydomain.com` is an alias to the CloudFront distribution. The request goes to the nearest CloudFront edge location.
3. **CloudFront**: CloudFront receives the request. Since this is dynamic (and we set no caching for this path), CloudFront quickly forwards it to the origin ALB. CloudFront keeps the client TCP connection open (if possible) for reuse. It establishes or reuses a connection to ALB (perhaps over HTTP/2 if ALB supports). CloudFront adds its own headers like X-Forwarded-For, etc., and passes along the Authorization header (since we configured that).
4. **Load Balancer**: ALB gets the request on port 443, decrypts SSL, and sees the path `/user/profile`. It has a rule that all paths go to the target group for our API service (we didn’t set any fancy path rules in this simple case). ALB chooses a target: say it picks one of the EC2 instances in the ASG. (If we had deployed on EKS only, it would pick a pod via the service's target group. If both ASG and EKS pods were behind it (hybrid), it could have target groups for each – but typically you wouldn’t do active-active on two different platforms for the same service; we include both in the diagram as alternate deployments).
5. **Application Server**: The Spring Boot app instance receives the request. A filter in the app (Spring Security) checks the JWT. It uses the JWKS from Cognito to validate signature; it sees token is valid and not expired. It sets the user principal (with roles/claims from token) for this request. Then the request hits the controller (`/user/profile`) which returns the profile data (from database). The app might query an RDS database (or DynamoDB, etc.) to get this info. Suppose it hits an RDS MySQL read replica for the profile. It gets the data and returns a JSON response. This JSON is sent back out.
6. **CloudFront caching**: CloudFront gets the response from ALB. By default, since we forwarded the Authorization header and have no caching, CloudFront will not cache this response (and it had `Cache-Control: private` perhaps from the app). So CloudFront simply streams it back to the user. The user receives their profile data with minimal added latency (maybe a few ms overhead from CloudFront).
7. **Scaling**: Now, suppose a surge of users comes in, all making requests. The ASG instances start getting high CPU usage. CloudWatch triggers the scaling policy; ASG launches 2 more instances. These instances take ~1-2 minutes to spin up, download the app, and start. Meanwhile, ALB shares load among existing instances. If they become too hot, some requests might slow down or in worst case some might time out. But soon the new instances come in and register. Now ALB spreads traffic to 4 instances instead of 2, reducing load on each. The responses speed up again.
   Simultaneously, CloudFront might be terminating a ton of client connections but only maintaining a few to ALB (connection reuse), which reduces stress on ALB. This is one way CloudFront helps under surge ([API & dynamic acceleration](https://aws.amazon.com/developer/application-security-performance/articles/api-dynamic-acceleration/#:~:text=or%20when%20it%20is%20purely,because%20the%20DTO%20overhead%20of)).
8. **WebSocket Update**: A user places an order via a `POST /orders` (goes through same path: CloudFront -> ALB -> app). The order is processed and stored in DB. The user’s app has opened a WebSocket connection to `wss://api.mydomain.com/notifications` (which CloudFront passes through to ALB, and ALB to one of the app instances). The app instance handling that WebSocket is subscribed (via our internal message broker or simple broker if single instance) to order status updates. When the order is created, maybe a background worker or the same request thread publishes an event "order 123 status = CREATED" to a topic. Because we have multiple instances, assume we configured a Redis pub/sub or an SNS topic such that all app instances get the message. The instance holding this user's WebSocket sees the message and sends it over the WebSocket to the client (e.g., as a JSON payload). The user sees "Order placed!" in real-time. This real-time path required coordination (Redis), but it scales: you can add more instances, as long as all connect to the same Redis or messaging system, all users get their messages.
9. **Kubernetes Alternative**: If we deployed on EKS, the flow is similar. CloudFront -> ALB -> service target (pod). The HPA might scale pods quickly if CPU or custom metric (like requests per second) increases. The Cluster Autoscaler would add nodes if those pods don't fit. From outside, it’s transparent whether the target was EC2 or pod. With ALB + target groups, it’s abstracted.
10. **Session Management**: Our design used Cognito JWT, so no server sessions. But let's say we also wanted to store shopping cart server-side for whatever reason. We could have the app use Redis (ElastiCache) to store session data keyed by a cookie. So even if the user bounces to another server, that server can retrieve the cart from Redis. There’s a slight performance hit (network call to Redis) but it’s fast (sub-millisecond in-memory). This ensures stateless *web tier* (state is in Redis which is a shared service). Redis itself can be clustered for scale or use memory-optimized RDS, etc.
11. **Cognito scaling**: Cognito can handle high auth volumes, but in extreme cases you might cache Cognito's public keys and user info. For example, to avoid hitting Cognito for user profile, your app might have cached the profile after first retrieval. Or Cognito can be set to share login tokens across edge via its hosted UI (less of our concern as we use JWTs after login). The point: managed services like Cognito and RDS also need to scale or be provisioned accordingly. RDS read replicas or Aurora can handle read-heavy loads, etc. Scaling is not just at the app layer.
12. **Deployments**: We manage code via CI/CD. E.g., push code to Git, CI runs tests, builds Docker image or JAR. If using Terraform, we might have it integrated to update Launch Template AMI ID or EKS Deployment with new image, and then apply. Auto-scaling also aids deployments: for zero downtime, one can deploy new instances/pods alongside old and then cut over. Using techniques like ALB target group weight shifting or Kubernetes rolling updates, you can deploy without taking down the whole service.

**End result**: We have a robust, scalable API:
- Under low load, minimal instances/pods run, saving cost.
- Under high load, the system automatically scales out (both at app layer and DB/cache layer if configured for that).
- CloudFront ensures global users get low latency and offloads a lot of network overhead.
- Authentication is secure and offloaded to Cognito, and each request is verified quickly via JWT.
- The architecture is infrastructure-as-code, so it’s reproducible in another region or environment by running Terraform (and Kubernetes manifests).
- Monitoring via CloudWatch and perhaps X-Ray (if we instrumented the app) gives insight into performance bottlenecks.
- If something fails (an AZ outage), auto-scaling can replace instances in healthy AZs, ALB will route to healthy ones, and the system continues running (maybe slightly degraded if capacity was lost, but it recovers).
- We used a combination of AWS managed services (Cognito, CloudFront, ALB, EKS) and open-source tech (Spring Boot, Kubernetes, Redis) to achieve a modern cloud-native setup.

## Conclusion

Scaling a web API on AWS requires a combination of good software architecture (statelessness, efficient code, caching), proper AWS infrastructure (load balancers, auto scaling groups or Kubernetes, CloudFront CDN, etc.), and automation tooling (Terraform for IaC, CI/CD for deployments). In this guide, we covered how different API types (REST, GraphQL, WebSockets, gRPC) influence scaling strategy and how to implement each with Spring Boot examples. We discussed securing the APIs using JWTs, OAuth2, and Cognito to ensure that security does not become a bottleneck but rather enhances the scalability by keeping things stateless and offloading heavy auth tasks to managed services.

Using **AWS Auto Scaling** (either directly with EC2 ASGs or indirectly via EKS HPA/Cluster Autoscaler) ensures your API can handle fluctuations in traffic by automatically adjusting capacity ([How to Deploy AWS Autoscaling Groups with Terraform](https://spacelift.io/blog/terraform-autoscaling-group#:~:text=AWS%20Auto%20Scaling%20Groups%20,availability%20and%20handle%20fluctuating%20workloads)) ([Scale pod deployments with Horizontal Pod Autoscaler - Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/horizontal-pod-autoscaler.html#:~:text=The%20Kubernetes%20Horizontal%20Pod%20Autoscaler,try%20to%20meet%20that%20target)). AWS CloudFront complements this by accelerating content delivery and shielding origin servers from excessive load and attacks (with AWS WAF) ([API & dynamic acceleration](https://aws.amazon.com/developer/application-security-performance/articles/api-dynamic-acceleration/#:~:text=,and%20in%20%2074%20CloudFront)) ([Web Application Firewall, Web API Protection - AWS WAF](https://aws.amazon.com/waf/#:~:text=Web%20Application%20Firewall%2C%20Web%20API,site%20scripting%20%28XSS)). Terraform allows us to declaratively set up all these components, and the provided code snippets give a starting point for your own infrastructure definitions.

**Key takeaways**:
- Design your API to be as stateless as possible. This simplifies scaling horizontally (add more instances or containers) without worrying about session affinity.
- Choose the right authentication mechanism – JWTs and Cognito are great for stateless auth ([JWT Authentication: A Secure & Scalable Solution for Modern Applications - Authgear](https://www.authgear.com/post/jwt-authentication-a-secure-scalable-solution-for-modern-applications#:~:text=If%20the%20JWT%20is%20valid%2C,reducing%20overhead%20and%20improving%20scalability)) ([Securing Your REST API with JWT Authorizers and Amazon Cognito - DEV Community](https://dev.to/aws-builders/securing-your-rest-api-with-jwt-authorizers-and-amazon-cognito-2dp2#:~:text=,API%20request%20for%20continued%20access)). They integrate well with ALB/Lambda authorizers or in-app security.
- Use Auto Scaling policies (target tracking on CPU or custom metrics) to automate adding/removing capacity. Monitor your app to choose sensible thresholds ([How to Deploy AWS Autoscaling Groups with Terraform](https://spacelift.io/blog/terraform-autoscaling-group#:~:text=Scaling%20policies%20define%20the%20conditions,traffic%2C%20or%20other%20custom%20metrics)).
- Leverage CloudFront for improved latency and offloading. It supports not just static but dynamic content, WebSockets, and even gRPC, which can significantly improve client experience for global users ([API & dynamic acceleration](https://aws.amazon.com/developer/application-security-performance/articles/api-dynamic-acceleration/#:~:text=Note%20that%20CloudFront%20supports%20APIs,based%20on%20WebSockets%20and%20gRPC)).
- If using Kubernetes, take advantage of HPA and Cluster Autoscaler to achieve similar scaling behavior at the pod level, and consider service mesh or other advanced features if needed.
- Plan for stateful needs by externalizing state (databases, caches). Avoid sticky sessions unless absolutely necessary ([Choosing a stickiness strategy for your load balancer - AWS Prescriptive Guidance](https://docs.aws.amazon.com/prescriptive-guidance/latest/load-balancer-stickiness/welcome.html#:~:text=Stickiness%20is%20a%20term%20that,available%20to%20the%20other%20server)), and even then, understand the cost.
- Implement security best practices at every layer – from using WAF to scanning your code for vulnerabilities – so that scaling doesn’t introduce security holes.

By following this guide, you should be able to construct a robust, scalable API platform on AWS. The combination of Terraform scripts and Spring Boot code examples can serve as a template for building out your own systems. As a next step, you can extend this setup with CI/CD pipelines (e.g., using GitHub Actions or Jenkins to apply Terraform and deploy app updates), automated database scaling (Aurora auto-scaling, etc.), and multi-region replication for disaster recovery.

Finally, always test your system under load (e.g., using load testing tools) before production, to ensure the auto-scaling reacts as expected and no single component becomes a bottleneck. Happy scaling!

## References

- AWS Prescriptive Guidance – *Sticky sessions with load balancer generated cookies* ([Sticky sessions with load balancer generated cookies - AWS Prescriptive Guidance](https://docs.aws.amazon.com/prescriptive-guidance/latest/load-balancer-stickiness/alb-cookies-stickiness.html#:~:text=After%20traffic%20has%20been%20initially,instance%20for%20a%20specified%20duration))
- AWS Prescriptive Guidance – *Choosing a stickiness strategy for your load balancer* ([Choosing a stickiness strategy for your load balancer - AWS Prescriptive Guidance](https://docs.aws.amazon.com/prescriptive-guidance/latest/load-balancer-stickiness/welcome.html#:~:text=Stickiness%20is%20a%20term%20that,available%20to%20the%20other%20server)) ([Choosing a stickiness strategy for your load balancer - AWS Prescriptive Guidance](https://docs.aws.amazon.com/prescriptive-guidance/latest/load-balancer-stickiness/welcome.html#:~:text=,cookies%20and%20load%20balancer%20cookies))
- AWS Developer Blog – *Dynamic content acceleration with CloudFront* (supports WebSocket and gRPC) ([API & dynamic acceleration](https://aws.amazon.com/developer/application-security-performance/articles/api-dynamic-acceleration/#:~:text=this%20workshop)) ([API & dynamic acceleration](https://aws.amazon.com/developer/application-security-performance/articles/api-dynamic-acceleration/#:~:text=,and%20in%20%2074%20CloudFront))
- AWS Documentation – *Amazon EKS Horizontal Pod Autoscaler* ([Scale pod deployments with Horizontal Pod Autoscaler - Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/horizontal-pod-autoscaler.html#:~:text=)) ([Scale pod deployments with Horizontal Pod Autoscaler - Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/horizontal-pod-autoscaler.html#:~:text=The%20Kubernetes%20Horizontal%20Pod%20Autoscaler,try%20to%20meet%20that%20target))
- Kubernetes GitHub – *Cluster Autoscaler FAQ* (scale up/down conditions) ([autoscaler/cluster-autoscaler/FAQ.md at master · kubernetes/autoscaler · GitHub](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#:~:text=Cluster%20Autoscaler%20increases%20the%20size,of%20the%20cluster%20when))
- Authgear – *JWT Authentication: Secure & Scalable Solution* (stateless tokens improve scalability) ([JWT Authentication: A Secure & Scalable Solution for Modern Applications - Authgear](https://www.authgear.com/post/jwt-authentication-a-secure-scalable-solution-for-modern-applications#:~:text=If%20the%20JWT%20is%20valid%2C,reducing%20overhead%20and%20improving%20scalability)) ([JWT Authentication: A Secure & Scalable Solution for Modern Applications - Authgear](https://www.authgear.com/post/jwt-authentication-a-secure-scalable-solution-for-modern-applications#:~:text=JWT%20authentication%20is%20widely%20used,side%20sessions))
- AWS Dev Blog – *Securing REST API with Cognito JWT Authorizer* (Cognito issues JWT for API use) ([Securing Your REST API with JWT Authorizers and Amazon Cognito - DEV Community](https://dev.to/aws-builders/securing-your-rest-api-with-jwt-authorizers-and-amazon-cognito-2dp2#:~:text=,API%20request%20for%20continued%20access))
- ByteByteGo Newsletter – *API Protocols 101* (protocols and scalability trade-offs) ([API Protocols 101: A Guide to Choose the Right One ](https://blog.bytebytego.com/p/api-protocols-101-a-guide-to-choose#:~:text=provides%20more%20control%20over%20data,fetching))
- Resolute Software Blog – *REST vs GraphQL vs gRPC vs WebSocket* (overview of technologies) ([ REST vs. GraphQL vs. gRPC vs. WebSocket](https://www.resolutesoftware.com/blog/rest-vs-graphql-vs-grpc-vs-websocket/#:~:text=gRPC%20is%20a%20robust%20open,API%20security%2C%20performance%2C%20and%20scalability)) ([ REST vs. GraphQL vs. gRPC vs. WebSocket](https://www.resolutesoftware.com/blog/rest-vs-graphql-vs-grpc-vs-websocket/#:~:text=1,context%20information%20on%20the%20server)) ([ REST vs. GraphQL vs. gRPC vs. WebSocket](https://www.resolutesoftware.com/blog/rest-vs-graphql-vs-grpc-vs-websocket/#:~:text=match%20at%20L254%20GraphQL%20gives,solves%20issues%20with%20bandwidth%20efficiency))
- Redis Labs – *Scaling with Redis Session Store* (using Redis for session data to enable horizontal scaling) ([Scaling an Express Application with Redis as a Session Store](https://redis.io/learn/develop/node/nodecrashcourse/sessionstorage#:~:text=Fortunately%2C%20Redis%20makes%20an%20excellent,Checkin%20Receiver%20services%2C%20with%20minimal))
- Spacelift Blog – *Terraform AWS Auto Scaling Group* (intro and best practices) ([How to Deploy AWS Autoscaling Groups with Terraform](https://spacelift.io/blog/terraform-autoscaling-group#:~:text=AWS%20Auto%20Scaling%20Groups%20,availability%20and%20handle%20fluctuating%20workloads)) ([How to Deploy AWS Autoscaling Groups with Terraform](https://spacelift.io/blog/terraform-autoscaling-group#:~:text=Scaling%20policies%20define%20the%20conditions,traffic%2C%20or%20other%20custom%20metrics))
- AWS Docs – *Use AWS WAF to protect APIs* (WAF blocks common attacks like SQLi, XSS) ([Web Application Firewall, Web API Protection - AWS WAF](https://aws.amazon.com/waf/#:~:text=Web%20Application%20Firewall%2C%20Web%20API,site%20scripting%20%28XSS))

