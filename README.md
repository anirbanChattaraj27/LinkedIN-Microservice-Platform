🚀 LinkedIn Microservices Platform

A backend-focused, LinkedIn-inspired professional networking platform built using Java, Spring Boot, Spring Cloud, Apache Kafka, PostgreSQL, Neo4j, and Microservices Architecture.

The project demonstrates how a real-world backend can be decomposed into independent services for authentication, posts, professional connections, and notifications, while combining synchronous REST communication with asynchronous event-driven communication using Kafka.

📌 Table of Contents
Overview
Architecture
Core Features
Microservices
How a Request Flows
Event-Driven Architecture
Databases
Authentication & Security
Backend Design & LLD Concepts
HLD Concepts
Technology Stack
Maven & Project Structure
Logging & Observability
Error Handling
Planned Docker & AWS Deployment
Scalability
How to Run Locally
Future Improvements
🎯 Overview

This project is a LinkedIn-inspired professional networking backend designed using a microservices architecture.

Instead of building one large monolithic application, the system is divided into independent services based on business responsibilities.

The major domains include:

👤 User Management
        │
        ├── Signup
        ├── Login
        └── JWT Authentication

📝 Post Management
        │
        ├── Create Post
        ├── Fetch Post
        └── Like / Unlike Post

🤝 Professional Connections
        │
        ├── Send Connection Request
        ├── Accept Connection
        ├── Reject Connection
        └── Find First-Degree Connections

🔔 Notifications
        │
        ├── Post Created
        └── Post Liked

The project uses:

⚡ Synchronous communication where an immediate response is required.
📩 Apache Kafka for asynchronous event-driven workflows.
🗄️ PostgreSQL for relational data.
🕸️ Neo4j for graph-based professional relationships.
🔎 Eureka for service discovery.
🚪 Spring Cloud Gateway as the central entry point.
🏗 Architecture
                              ┌─────────────────┐
                              │     Client      │
                              │ Browser/Postman │
                              └────────┬────────┘
                                       │
                                       ▼
                         ┌─────────────────────────┐
                         │       API Gateway       │
                         │      Port: 8080         │
                         │                         │
                         │ JWT Authentication      │
                         │ Request Routing         │
                         └───────────┬─────────────┘
                                     │
              ┌──────────────────────┼──────────────────────┐
              │                      │                      │
              ▼                      ▼                      ▼
     ┌────────────────┐    ┌────────────────┐    ┌────────────────────┐
     │  User Service  │    │  Posts Service │    │ Connections Service│
     │   Port: 9020   │    │   Port: 9010   │    │    Port: 9030      │
     └───────┬────────┘    └───────┬────────┘    └─────────┬──────────┘
             │                     │                        │
             ▼                     ▼                        ▼
        PostgreSQL            PostgreSQL                  Neo4j
          userDB                postsDB                 Graph DB
             │                     │
             │                     │
             ▼                     ▼
                       ┌──────────────────┐
                       │      Kafka       │
                       │ Event Streaming  │
                       └────────┬─────────┘
                                │
                                ▼
                     ┌─────────────────────┐
                     │ Notification Service│
                     │     Port: 9040      │
                     └──────────┬──────────┘
                                │
                                ▼
                           PostgreSQL
                         notificationDB


                  ┌────────────────────┐
                  │ Eureka Discovery   │
                  │    Port: 8761      │
                  └────────────────────┘
🧩 Microservices
👤 User Service

Port: 9020

Responsible for:

User signup
User login
Password hashing
JWT generation
Publishing user creation events
Flow
Signup Request
      │
      ▼
UserController
      │
      ▼
AuthService
      │
      ├── Check existing user
      │
      ├── Hash password
      │
      ├── Save User
      ▼
 PostgreSQL
      │
      │ UserCreatedEvent
      ▼
    Kafka

When a user is successfully created, the service publishes:

user_created_topic

The Connections Service consumes this event and creates the corresponding graph node in Neo4j.

📝 Posts Service

Port: 9010

Responsible for:

Creating posts
Fetching posts
Fetching posts by user
Liking posts
Unliking posts
Fetching first-degree connections
Publishing post events

The Posts Service communicates synchronously with the Connections Service using OpenFeign.

Example flow
User creates a post
        │
        ▼
Posts Service
        │
        ├── Save Post → PostgreSQL
        │
        ├── Call Connections Service
        │      using OpenFeign
        │
        ▼
Get First-Degree Connections
        │
        ▼
Publish PostCreated Events
        │
        ▼
Kafka Topic
🤝 Connections Service

Port: 9030

The Connections Service manages professional relationships between users.

It supports:

Sending connection requests
Accepting requests
Rejecting requests
Finding first-degree connections
Why Neo4j?

Professional networking is naturally relationship-oriented.

Instead of representing connections only as rows and joins, the application models them as a graph.

        CONNECTED_TO
User A ─────────────── User B
  │
  │ CONNECTED_TO
  │
User C

A connection request can be represented as:

User A ── REQUESTED_TO ──► User B

After acceptance:

User A ── CONNECTED_TO ── User B

The project uses custom Cypher queries through Spring Data Neo4j to perform graph operations.

🔔 Notification Service

Port: 9040

The Notification Service consumes Kafka events asynchronously.

It listens to events such as:

post_created_topic
post_liked_topic
Example
User creates post
        │
        ▼
Posts Service
        │
        ▼
Kafka
        │
        ▼
Notification Service
        │
        ▼
Create Notification
        │
        ▼
PostgreSQL

This means the Posts Service does not need to synchronously wait for notification processing.

🚪 API Gateway

The API Gateway acts as the single entry point to the backend.

Instead of the client directly knowing every service URL:

Client
   │
   ├── User Service URL
   ├── Posts Service URL
   └── Connections Service URL

the client communicates through:

Client
   │
   ▼
API Gateway
   │
   ├── User Service
   ├── Posts Service
   └── Connections Service
Example routes
/api/v1/users/**
        │
        ▼
USER-SERVICE

/api/v1/posts/**
        │
        ▼
POSTS-SERVICE

/api/v1/connections/**
        │
        ▼
CONNECTIONS-SERVICE

The gateway uses service discovery-based routing:

lb://USER-SERVICE
lb://POSTS-SERVICE
lb://CONNECTIONS-SERVICE
🔐 Authentication & Security

Authentication is implemented using JWT (JSON Web Tokens).

Login flow
User
 │
 │ Email + Password
 ▼
User Service
 │
 ├── Validate credentials
 │
 ├── Verify hashed password
 │
 ▼
Generate JWT
 │
 ▼
Client

For protected requests:

Client Request
       │
       │ Authorization: Bearer <JWT>
       ▼
API Gateway
       │
       ├── Validate JWT
       │
       ├── Extract User ID
       │
       └── Forward X-User-Id
                │
                ▼
         Downstream Service

The downstream services use an authentication context to access the authenticated user.

🔄 How Microservices Communicate

This project uses two communication styles.

1️⃣ Synchronous Communication — REST / OpenFeign

Used when one service requires an immediate response from another service.

Example:

Posts Service
      │
      │ OpenFeign Request
      ▼
Connections Service
      │
      ▼
First-Degree Connections

The Posts Service needs the list of connected users before publishing notifications for a new post.

2️⃣ Asynchronous Communication — Apache Kafka

Used when services should not be tightly coupled or when immediate processing is unnecessary.

Producer Service
       │
       │ Publish Event
       ▼
   Kafka Topic
       │
       ▼
Consumer Service

Implemented events include:

user_created_topic
post_created_topic
post_liked_topic
⚡ Event-Driven Architecture
User Signup
User Signup
    │
    ▼
User Service
    │
    ├── Save User in PostgreSQL
    │
    └── Publish UserCreatedEvent
              │
              ▼
       user_created_topic
              │
              ▼
      Connections Service
              │
              ▼
     Create Person Node
              │
              ▼
            Neo4j

This is a good example of event-driven synchronization between services.

Post Creation
Create Post
     │
     ▼
Posts Service
     │
     ├── Save Post
     │
     ├── Fetch Connections
     │
     ▼
Publish PostCreated Events
     │
     ▼
Kafka
     │
     ▼
Notification Service
     │
     ▼
Store Notification
Post Like
Like Post
    │
    ▼
Posts Service
    │
    ├── Validate Post
    ├── Prevent Duplicate Like
    ├── Save Like
    │
    ▼
Publish PostLiked Event
    │
    ▼
Kafka
    │
    ▼
Notification Service
🔎 Service Discovery with Eureka

In a distributed environment, service addresses should not be manually hardcoded.

This project uses Netflix Eureka.

                Eureka Server
                     │
          ┌──────────┼───────────┐
          │          │           │
          ▼          ▼           ▼
       User       Posts      Connections
      Service     Service      Service

Each service registers with Eureka.

The API Gateway and other services can discover services using their logical names instead of fixed IP addresses.

Example:

USER-SERVICE
POSTS-SERVICE
CONNECTIONS-SERVICE

This helps decouple service communication from static infrastructure addresses.

🗄️ Database Architecture

The project uses polyglot persistence, meaning different database technologies are selected based on the nature of the data.

PostgreSQL

Used for structured relational data:

User Service
    └── userDB

Posts Service
    └── postsDB

Notification Service
    └── notificationDB

The project uses:

Spring Data JPA
Hibernate
Repository layer
Entity mapping
Transactions
Neo4j

Used by the Connections Service.

(:Person)

represents a user node.

Relationships include:

(:Person)-[:REQUESTED_TO]->(:Person)

(:Person)-[:CONNECTED_TO]-(:Person)

Neo4j is particularly suitable for traversing relationship-oriented data such as professional connections.

🧠 Backend Design & LLD Concepts

This project follows several important low-level design and backend engineering principles.

Layered Architecture

Each service follows a clear separation:

Controller
    │
    ▼
Service
    │
    ▼
Repository
    │
    ▼
Database
Controller Layer

Responsible for:

Receiving HTTP requests
Validating/request mapping
Returning HTTP responses
Service Layer

Responsible for:

Business logic
Orchestrating workflows
Calling repositories and external services
Publishing Kafka events
Repository Layer

Responsible for:

Database interaction
JPA operations
Neo4j operations
Custom queries
Dependency Injection

Spring manages dependencies and injects required components.

Example conceptually:

private final UserRepository userRepository;
private final JwtService jwtService;
private final KafkaTemplate<Long, UserCreatedEvent> kafkaTemplate;

This reduces tight coupling and improves testability.

Repository Pattern

Database access is abstracted through repositories.

Examples:

UserRepository
PostRepository
PostLikeRepository
NotificationRepository
PersonRepository

This separates persistence logic from business logic.

DTO Pattern

DTOs are used to transfer data between API layers instead of directly exposing entities.

Examples include:

SignupRequestDto
LoginRequestDto
UserDto
PostCreateRequestDto
PostDto
PersonDto

Benefits:

Better API contract control
Reduced entity exposure
Separation between persistence and API models
Global Exception Handling

Each service contains centralized exception handling.

Typical custom exceptions include:

BadRequestException
ResourceNotFoundException

A GlobalExceptionHandler converts exceptions into structured API responses.

This avoids repeating error handling logic inside every controller.

Transaction Management

Critical database operations such as liking a post use transactional boundaries.

Conceptually:

Start Transaction
      │
      ├── Validate Post
      ├── Check Existing Like
      ├── Save Like
      │
      ▼
Commit

If an operation fails within the transaction, Spring can roll back the database changes.

🏛️ HLD Concepts Demonstrated

This project demonstrates several high-level system design concepts.

Microservices Architecture

The application is divided based on business capabilities.

API Gateway Pattern

A single entry point handles routing and authentication.

Service Discovery

Eureka enables services to discover each other dynamically.

Event-Driven Architecture

Kafka decouples producers and consumers.

Polyglot Persistence

PostgreSQL and Neo4j are used for different data models.

Database Ownership

Services logically own their data instead of exposing direct database access to other services.

Synchronous + Asynchronous Communication
Immediate Response Needed
        │
        ▼
REST / OpenFeign


Background / Event Processing
        │
        ▼
Kafka
📦 Maven & Build Management

Each microservice is an independent Spring Boot application with its own:

pom.xml
src/
application.properties
application.yml

Maven is responsible for:

Dependency management
Build lifecycle
Packaging applications
Running tests
Creating executable JAR files

Typical workflow:

./mvnw clean install

or:

mvn clean install

Each service can be built independently.

🪵 Logging with SLF4J

The project uses Lombok's:

@Slf4j

to provide structured application logging.

Examples of logged operations include:

User signup
User login
Post creation
Post retrieval
Post likes
Connection requests
Kafka event consumption

Example:

log.info("Creating post for user with id: {}", userId);

Logging is useful for:

Debugging distributed services
Tracking application flow
Investigating failures
Understanding Kafka event processing
🛠️ Technology Stack
Category	Technologies
Language	Java
Backend Framework	Spring Boot
Microservices	Spring Cloud
API Gateway	Spring Cloud Gateway
Service Discovery	Netflix Eureka
Synchronous Communication	REST APIs, OpenFeign
Asynchronous Communication	Apache Kafka
Relational Database	PostgreSQL
Graph Database	Neo4j
ORM	Spring Data JPA / Hibernate
Graph Data Access	Spring Data Neo4j
Authentication	JWT
Password Security	BCrypt
Build Tool	Maven
Logging	SLF4J / Lombok
Mapping	ModelMapper
Testing	Spring Boot Test / Kafka Test dependencies
🐳 Planned Docker Deployment

The backend services are designed to be independently containerized.

Conceptually:

User Service
     │
     ▼
Docker Image

Posts Service
     │
     ▼
Docker Image

Connections Service
     │
     ▼
Docker Image

Notification Service
     │
     ▼
Docker Image

API Gateway
     │
     ▼
Docker Image

Each service can have its own Dockerfile and image.

Example deployment lifecycle:

Spring Boot Application
        │
        ▼
      Maven
        │
        ▼
   Executable JAR
        │
        ▼
     Docker Image
        │
        ▼
   Container Registry
        │
        ▼
     Runtime
☁️ Planned AWS Architecture

The intended AWS deployment architecture is:

                    ┌────────────────────┐
                    │    AWS ECS/Fargate │
                    │                    │
                    │  API Gateway       │
                    │  User Service      │
                    │  Posts Service     │
                    │  Connections       │
                    │  Notification      │
                    └─────────┬──────────┘
                              │
               ┌──────────────┴──────────────┐
               ▼                             ▼
     ┌───────────────────┐          ┌───────────────────┐
     │ AWS RDS           │          │   Neo4j AuraDB    │
     │ PostgreSQL        │          │   Graph Database  │
     │                   │          │                   │
     │ userDB            │          └───────────────────┘
     │ postsDB           │
     │ notificationDB    │
     └───────────────────┘
Planned deployment flow
Docker Image
      │
      ▼
Amazon ECR
      │
      ▼
AWS ECS / Fargate
      │
      ├── PostgreSQL → AWS RDS
      │
      └── Graph Data → Neo4j AuraDB

Note: Docker and AWS deployment configuration can be added to the repository as the infrastructure implementation is completed.

📈 Scalability

The architecture allows services to be scaled independently.

For example:

Low Traffic:

User Service → 1 Task


High Traffic:

Posts Service → Multiple Tasks

The important concept is:

Same Docker Image
        │
        ├── ECS Task 1
        ├── ECS Task 2
        └── ECS Task 3

This enables horizontal scaling of individual services rather than scaling the entire application.

Kafka consumers can also be scaled independently using consumer groups and partitions.

The current project should be described as designed for independent service scaling. Production-scale infrastructure features should only be claimed after they are configured and tested.

📁 Project Structure
linkedin-microservices-platform
│
├── APIGateway
│   ├── Authentication Filter
│   ├── JWT Validation
│   └── Service Routing
│
├── DiscoverServer
│   └── Eureka Service Discovery
│
├── userService
│   ├── Authentication
│   ├── JWT Generation
│   ├── PostgreSQL
│   └── Kafka Producer
│
├── postsService
│   ├── Posts
│   ├── Likes
│   ├── PostgreSQL
│   ├── OpenFeign
│   └── Kafka Producer
│
├── ConnectionsService
│   ├── Connection Requests
│   ├── Graph Relationships
│   ├── Neo4j
│   └── Kafka Consumer
│
└── notification-service
    ├── Kafka Consumers
    └── PostgreSQL Notifications
🚀 How to Run Locally
Prerequisites

Make sure the following are available:

Java
Maven
PostgreSQL
Neo4j
Apache Kafka
Configure Databases

Create the PostgreSQL databases:

userDB
postsDB
notificationDB

Configure Neo4j for the Connections Service.

Update the relevant:

application.properties
application.yml

files with your local credentials and connection details.

Start Order

A recommended startup sequence is:

1️⃣ Start PostgreSQL

2️⃣ Start Neo4j

3️⃣ Start Kafka

4️⃣ Start Eureka Discovery Server

5️⃣ Start User Service

6️⃣ Start Posts Service

7️⃣ Start Connections Service

8️⃣ Start Notification Service

9️⃣ Start API Gateway
🔮 Future Improvements

Possible next improvements include:

🐳 Add Dockerfiles for all services
📦 Add Docker Compose for local development
☁️ Deploy services to AWS ECS/Fargate
🗄️ Move PostgreSQL to AWS RDS
🕸️ Move Neo4j to AuraDB
🔐 Store credentials in AWS Secrets Manager
📊 Add distributed tracing and centralized logging
⚡ Add Redis caching for frequently accessed data
🔄 Implement Kafka retry and dead-letter topic handling
📨 Improve event reliability using the Transactional Outbox pattern
❤️ Add Spring Boot Actuator health checks
🧪 Increase unit and integration test coverage
📈 Add autoscaling after the initial deployment is stable
🎓 Key Concepts Demonstrated
Java & Spring Boot
        │
        ├── REST APIs
        ├── Dependency Injection
        ├── Layered Architecture
        ├── JPA / Hibernate
        ├── Transactions
        └── Global Exception Handling

Microservices
        │
        ├── API Gateway
        ├── Service Discovery
        ├── OpenFeign
        ├── Independent Services
        └── Database Ownership

Apache Kafka
        │
        ├── Producers
        ├── Consumers
        ├── Topics
        └── Event-Driven Communication

Databases
        │
        ├── PostgreSQL
        └── Neo4j

Security
        │
        ├── JWT
        └── BCrypt

Cloud & Deployment
        │
        ├── Docker
        ├── Amazon ECR
        ├── Amazon ECS/Fargate
        ├── Amazon RDS
        └── Neo4j AuraDB
