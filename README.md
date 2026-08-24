# 🚀 LinkedIn Microservices Platform

> A LinkedIn-inspired professional networking platform built using **Java, Spring Boot, Microservices, Apache Kafka, PostgreSQL, Neo4j, Spring Cloud Gateway, and Eureka**.

This project demonstrates how a modern backend application can be designed using **independently deployable microservices**, combining synchronous REST communication with asynchronous event-driven communication.

---

## 📌 Overview

The platform is divided into multiple services based on business responsibilities.

It supports:

- 👤 User registration and authentication
- 🔐 JWT-based security
- 📝 Creating and managing posts
- ❤️ Liking and unliking posts
- 🤝 Sending and accepting connection requests
- 🕸️ Managing professional relationships using Neo4j
- 🔔 Event-driven notifications using Apache Kafka
- 🔎 Service discovery using Eureka
- 🚪 Centralized request routing using Spring Cloud Gateway

---

# 🏗️ Architecture

```mermaid
flowchart TB

    Client[👤 Client / Postman]

    Gateway[🚪 API Gateway<br/>Port: 8080]

    Eureka[🔎 Eureka Discovery Server<br/>Port: 8761]

    UserService[👤 User Service<br/>Port: 9020]
    PostService[📝 Posts Service<br/>Port: 9010]
    ConnectionService[🤝 Connections Service<br/>Port: 9030]
    NotificationService[🔔 Notification Service<br/>Port: 9040]

    UserDB[(PostgreSQL<br/>userDB)]
    PostDB[(PostgreSQL<br/>postsDB)]
    NotificationDB[(PostgreSQL<br/>notificationDB)]

    Neo4j[(🕸️ Neo4j<br/>Graph Database)]

    Kafka[📨 Apache Kafka<br/>Event Streaming]

    Client --> Gateway

    Gateway --> UserService
    Gateway --> PostService
    Gateway --> ConnectionService

    UserService --> UserDB
    PostService --> PostDB
    NotificationService --> NotificationDB

    ConnectionService --> Neo4j

    UserService --> Kafka
    PostService --> Kafka

    Kafka --> ConnectionService
    Kafka --> NotificationService

    UserService -. Registers .-> Eureka
    PostService -. Registers .-> Eureka
    ConnectionService -. Registers .-> Eureka
    NotificationService -. Registers .-> Eureka
    Gateway -. Service Discovery .-> Eureka
```


🧩 Microservices
👤 User Service

Port: 9020

Responsible for:

User signup
User login
Password hashing
JWT generation
User validation
Publishing user-related events
Flow

```mermaid
%%{init: {'theme': 'dark'}}%%
sequenceDiagram
    participant Client
    participant UserService
    participant PostgreSQL
    participant Kafka

    Client->>UserService: Signup Request
    UserService->>UserService: Validate User
    UserService->>UserService: Hash Password
    UserService->>PostgreSQL: Save User
    PostgreSQL-->>UserService: User Created
    UserService->>Kafka: Publish UserCreatedEvent
```

