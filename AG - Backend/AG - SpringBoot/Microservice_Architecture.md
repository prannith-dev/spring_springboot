# Microservices Architecture & API Integration

## 1. The Concept
Microservice Architecture is a design pattern where a large application is broken down into smaller, independent, and loosely coupled services. Instead of a single application containing the database connection, business logic, and UI (a Monolith), you have a **Consumer (Client)** and a **Provider (Source of Truth)** communicating over the internet via HTTP REST APIs.

## 2. The Problem it Solves
In traditional monolithic applications:
*   **Data Ownership:** If a third party (like John Doe Railway) owns the data, you physically cannot connect to their database. 
*   **Scalability:** If only the ticket-booking feature receives high traffic, you have to duplicate the entire monolithic server to scale it.
*   **Single Point of Failure:** If one part of the monolith crashes (e.g., payment processing), the entire application (including searching for trains) goes down.

## 3. What is its Use?
By separating the `train-service` (Provider on Port 8080) and the `demo-train-service-client` (Consumer on Port 8081), we achieve:
*   **Delegation:** The Provider handles raw data, while the Client focuses on user requests and custom business rules.
*   **Flexibility:** The Client can be updated, restarted, or modified without ever touching the core railway database.

## 4. Implementation in the Code
Here is how this architecture is implemented in your projects, complete with comments.

### The Consumer (REST Client)
Location: [TrainRestClient.java](file:///C:/SpringBoot/Phase%202%20Projects/demo-train-service-client/demo-train-service-client/src/main/java/com/training/demo_train_service_client/Client/TrainRestClient.java)

```java
package com.training.demo_train_service_client.Client;

import org.springframework.core.ParameterizedTypeReference;
import org.springframework.stereotype.Component;
import org.springframework.web.client.RestClient;
import java.util.List;
// ... other imports

@Component
public class TrainRestClient {
    
    // Spring's modern RestClient for making HTTP calls
    private final RestClient restClient;

    // CONSTRUCTOR INJECTION: Safer than @Autowired. Ensures the client cannot be built without the RestClient.
    public TrainRestClient(RestClient.Builder builder){
        // We set the base URL to point to the Provider (John Doe Railway)
        this.restClient = builder.baseUrl("http://localhost:8080").build();
    }

    public List<Train> getAllTrains(){
        return restClient.get()
                // Appends to the base URL -> http://localhost:8080/trains/getAll
                .uri("/trains/getAll") 
                // Secures the API call using Basic Authentication
                .headers(headers -> headers.setBasicAuth("admin","admin123"))
                .retrieve()
                // ParameterizedTypeReference handles the complex deserialization of a JSON Array into a Java Generic List
                .body(new ParameterizedTypeReference<List<Train>>() {});
    }
}
```

### The Consumer (Controller)
Location: [TrainController.java](file:///C:/SpringBoot/Phase%202%20Projects/demo-train-service-client/demo-train-service-client/src/main/java/com/training/demo_train_service_client/Controller/TrainController.java)

```java
package com.training.demo_train_service_client.Controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import java.util.List;
// ... other imports

@RestController("/trains")
public class TrainController {
    
    @Autowired
    TrainRestClient trainRestClient;

    // The user hits port 8081, and this method delegates the work to the RestClient (which hits port 8080)
    @GetMapping
    public List<Train> getAllTrains(){
        return trainRestClient.getAllTrains();
    }
}
```

## 5. Limitations, Exceptions, and Loopholes

### Limitations
*   **Network Latency:** Communicating over HTTP is exponentially slower than querying a local database directly.
*   **Complex Debugging:** When a bug occurs, you now have to trace logs across multiple servers and ports (8081 -> 8080) instead of a single application.

### Exceptions
*   **`ResourceAccessException` / `ConnectException`:** Occurs when the target server (Port 8080) is offline or unreachable.
*   **`HttpClientErrorException` (4xx Errors):** Thrown when the Client sends a bad request (e.g., `401 Unauthorized` for wrong passwords).
*   **`HttpServerErrorException` (5xx Errors):** Thrown when the Provider crashes internally while processing the request.

### Loopholes (The "Type Erasure" Loophole)
*   **JSON Deserialization Trap:** In Java, Generics (like `List<Train>`) suffer from "Type Erasure" at runtime. If you try to tell Jackson to map `.body(List.class)`, Java forgets what is *inside* the List, causing a `ClassCastException` or returning a List of `LinkedHashMap` instead of `Train` objects. 
*   **The Fix:** You *must* use `ParameterizedTypeReference<List<Train>>(){}` to trick Java into retaining the generic type during runtime.
