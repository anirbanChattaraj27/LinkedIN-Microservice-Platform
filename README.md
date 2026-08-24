sequenceDiagram

    participant Client
    participant UserService

    Client->>UserService: Email + Password
    UserService->>UserService: Validate Credentials
    UserService->>UserService: Generate JWT
    UserService-->>Client: JWT Token
