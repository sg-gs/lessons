# Systems design: Key concepts and practical exercise

This README outlines the key points to cover a system design lesson, along with a practical exercise to reinforce understanding.

## Key Concepts (3')

### 1. Understanding requirements
   - **Functional requirements**: Define what the system must do (e.g., user authentication, data storage, API endpoints).
   - **Non-functional requirements**: Address performance, scalability, reliability, and availability (e.g., 99.9% uptime, low latency).
   - Clarify constraints like traffic volume, data size, and budget.

### 2. Scalability
   - **Vertical scaling**: Adding more resources (CPU, RAM) to a single machine.
   - **Horizontal scaling**: Adding more replicas to distribute the load.
   - Use load balancers to distribute traffic across servers.

### 3. Availability and Reliability
   - **Availability**: Percentage of time the system is operational (e.g., 99.9% uptime).
   - **Reliability**: Ensuring the system performs correctly under expected conditions.
   - Implement redundancy (e.g., multiple servers, failover mechanisms).

### 4. Data storage and management
   - **Relational Databases**: SQL-based (e.g., MySQL, PostgreSQL) for structured data.
   - **NoSQL Databases**: For unstructured or semi-structured data (e.g., MongoDB, DynamoDB).
   - **Caching**: Use in-memory stores like Redis or Memcached to reduce database load.
   - **consistency vs. availability**: Understand CAP theorem trade-offs (Consistency, Availability, Partition Tolerance).

### 5. API Design
   - Design RESTful or GraphQL APIs for clear, predictable communication.
   - Use versioning to manage API changes.
   - Ensure proper error handling and status codes.

### 6. Microservices vs. Monolith
   - **Monolith**: Single codebase, easier to develop but harder to scale.
   - **Microservices**: Independent services, easier to scale but complex to manage.
   - Use message queues (e.g., RabbitMQ, Kafka) for inter-service communication.

### 7. Latency and Performance
   - Optimize database Queries (e.g., indexing, query optimization).
   - Use Content Delivery Networks (CDNs) for static content.
   - Minimize network round-trips.

### 8. Security
   - Implement authentication (e.g., OAuth, JWT) and authorization.
   - Encrypt data in transit (TLS/SSL) and at rest.
   - Protect against common vulnerabilities (e.g., SQL injection, XSS).

## Practical Exercise: Design a URL Shortener (10')

### Objective
Design a URL shortening service like Bit.ly, which takes a long URL and generates a short, unique alias that redirects to the original URL.

### Deliverable
By the end of the session, we should have:
- A high-level system diagram (can be a rough sketch).
  - Suggested tools: [Excalidraw](https://excalidraw.com)
- A brief explanation of design choices (e.g., why NoSQL vs. SQL).
- Identification of at least one potential bottleneck and a mitigation strategy.

### Steps to follow
1. **Gather requirements**
   - Functional: Shorten URLs, redirect from short to long URLs, track click analytics.
   - Non-Functional: Handle 1M requests/month, low latency, high availability.
   - Constraints: Assume a single-region deployment for simplicity.

2. **High-level design**:
   - Sketch a system diagram including:
     - API endpoints (e.g., POST /shorten, GET /{shortURL}).
     - Database for storing URL mappings.
     - Load balancer for distributing traffic.
     - Cache for frequently accessed short URLs.
   - Discuss trade-offs (e.g., database choice: SQL vs. NoSQL).

3. **Detailed components**:
   - **Short URL generation**: Use a hash function (e.g., MD5) or base62 encoding on a counter.
   - **Database schema**: Table with fields `short_url`, `long_url`, `created_at`, `click_count`.
   - **Scalability**: Propose horizontal scaling for the API servers and sharding for the database.
   - **Caching**: Suggest Redis for caching short-to-long URL mappings.
   - **Error Handling**: Discuss handling invalid URLs or duplicate short URLs.

4. **Wrap-up**:
   - Review the design and discuss potential bottlenecks (e.g., hash collisions, database load).
   - Suggest improvements like adding a CDN or analytics for click tracking.
