# Layered architecture idea of kxphotographer

This repository stores my, kxphotographer's, idea of the layered architecture.

---

# Overview

The project consists of the following layers:

- Runtime
- Domain
- User interface
- Application (Use case)
- Infrastructure

## Dependency rules

- Runtime layer depends on implementation of user interface, application and infrastructure layers.
- User interface, application and infrastructure layers depends on domain layer.
- User interface layer's implementation depends on application port (interface).
- Application layer implements application port defined in user interface layer.
- Application layer's implementation depends on infrastructure layer port.
- Infrastructure layer implements infrastructure port defined in application layer.

When the whole implementation is run as an application, implementation in the user interface layer calls a function in the application layer, and implementation in the application layer calls a function in the infrastructure layer as a result, but actually implementations just depend on ports (or interfaces) defined in the same layer, and the lower layer implements them. So the actual direction of the dependencies are infrastructure layer to application layer, and application layer to user interface layer. This is because the [dependency inversion principal](https://en.wikipedia.org/wiki/Dependency_inversion_principle) is applied.

```mermaid
graph LR

subgraph "Runtime layer"
    E["Application entrypoint"]
end

subgraph "User interface layer"
    Ui["Implementation"]
    Ap["Application port<br>(interface)"]
end

subgraph "Application layer"
    Ai["Application implementation"]
    Ip["Infrastructure port<br>(interface)"]
end

subgraph "Infrastructure layer"
    Ii["Infrastructure implementation"]
end

subgraph Ld["Domain layer"]
    Da["Aggregate"]
    De["Entity"]
    Dv["Value object<br>(or branded type)"]
end

E -->|depends on| Ui
E -->|depends on| Ai
E -->|depends on| Ii

Ui -->|depends on| Ld
Ap -->|depends on| Ld
Ai -->|depends on| Ld
Ip -->|depends on| Ld
Ii -->|depends on| Ld

Ui -.->|depends on| Ap
Ai -.->|depends on| Ip

Ai -.-o|implements| Ap
Ii -.-o|implements| Ip

Da --> De
Da --> Dv
De --> Dv
```

---

# Runtime layer

This layer contains one or more entry point scripts to run the app. A script is responsible to instantiate instances from user interface, application and infrastructure layers resolving their dependencies so that implementation of the whole project runs as an application.

---

# Domain layer

This layer defines data: more specifically, how a value can be, what attributes a unit of data have, and what a consistent state is, in terms of the business the application covers.

---

# User interface layer

This layer is responsible for calling a function of the application layer converting/validating user input into values defined in the domain layer, and display output of the application layer function's result to the user. Here, "user" does not necessarily mean a human; it can be a web browser, a mobile app, a remote server etc.

This layer also defines ports (or interfaces) for application layer, which should implement them.

---

# Application (Use case) layer

This layer is responsible to orchestrate data using domain objects and functions of infrastructure layer as requested by user interface layer.

This layer also defines ports for infrastructure layer, which should implement them.

This layer should be able to decide the boundary of atomicity, in other word, the range of transaction, without depending on the concrete technology used in the infrastructure. Therefore, this layer should also define a port for the transaction wrapper, and infrastructure layer should implement it.

```ts
// Example of transaction wrapper type in TypeScript
type TransactionWrapper<TransactionType> =
    <const CallbackResult extends { commit: boolean }>(
        callback: (transaction: TransactionType) => CallbackResult,
    ) => CallbackResult;
```

---

# Infrastructure layer

This layer is responsible to communicate with concrete external resources such as a relational database, an in-memory key-value store, an object storage, an external API, and so on, converting data from/to that defined in the domain layer. This layer should implement the infrastructure ports defined in the application layer.
