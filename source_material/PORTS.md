# Definition: Driven Ports
In the context of Hexagonal Architecture (Ports and Adapters), a **Driven Port** (also known as a secondary or outbound port) is an interface defined by the application core that allows it to interact with external systems, such as databases, message buses, or third-party APIs.

Unlike "Driver" ports (which handle incoming requests), Driven ports are invoked *by* the application to trigger actions in the outside world. The key goal is to decouple the business logic from infrastructure details.

---

# Core Design Principles

## 1. Dependency Inversion Principle (DIP)

The most critical principle for driven ports is that **the domain defines the interface, and the infrastructure implements it**.

## 2. Interface Segregation Principle (ISP)

Avoid creating massive, monolithic interfaces (e.g., a single `GlobalDataService`).
Interfaces should be specific to the client using them. If a use case only needs to find a customer by ID, it should not depend on an interface that also includes methods for deleting or updating customers.

## 3. Define by Intent, Not Implementation

The port interface should reflect the *business intent*, not the underlying technology.
*   **Business Language:** Method signatures should use the Ubiquitous Language of the domain (e.g., `scheduleDelivery(...)` or `findCustomerContactInfo(...)`) rather than generic CRUD terminology or SQL-like syntax.
*   **Technology Agnostic:** The interface must not leak implementation details. For example, it should not throw SQL exceptions or accept JDBC connection parameters.

---

# Common Patterns for Driven Ports

## 1. The Repository Pattern
This is the standard pattern for accessing persistent storage.
*   **Illusion of Memory:** The repository interface should provide the illusion of an in-memory collection of domain objects.
*   **Methods:** It typically includes methods for adding, removing, and retrieving aggregates (e.g., `save(Order)`, `orderOfId(OrderId)`).

## 2. The Gateway / Proxy Pattern
When communicating with external services (like a payment processor or a legacy monolith), use a Gateway interface.
*   **Encapsulation:** The interface encapsulates the mechanism used to fetch data (REST, gRPC, SOAP).
*   **Translation:** The implementation (Adapter) is responsible for marshalling and unmarshalling data, leaving the Port interface clean of protocol-specific artifacts.

## 3. The Output Port Pattern (Event Publisher)
Used for asynchronous communication or event-driven architectures.
*   **Write-Only:** An application service might simply "write" a domain object or event to an output port.
*   **Observer/Mediator:** The application defines an interface (observer) that the infrastructure implements. When the application performs an action, it notifies the interface, which pushes the event to a message bus (e.g., RabbitMQ).

---

# Design Guidance for Data Exchange

Designing the method signatures for driven ports requires careful consideration of what data crosses the boundary.

## Data Structures vs. Domain Objects
There is a tension between strict decoupling and pragmatism regarding what data is passed through a port:
- **Pragmatic DDD:** In Domain-Driven Design, Repositories often accept and return **Aggregates** (Domain Entities). The infrastructure adapter is responsible for mapping these rich objects to database rows (Data Mappers).
- **Strict Decoupling:** Ideally, pass simple data structures (DTOs, structs, hashmaps) across boundaries. This ensures the infrastructure does not depend on the domain's internal logic and vice versa.
- **Rule of Thumb:** Do not pass frameworks-specific objects (like Hibernate proxies or active record rows) into the domain.

Recommendation: Prefer 'Pragmatic DDD' over 'Strict decoupling'.

## The Anti-Corruption Layer (ACL)

- **Translation:** The Port defines the model in *your* terms.
- **Protection:** This prevents "messy" legacy models or volatile upstream changes from leaking into your clean core.

---

### Testing Advantages
Properly designed driven ports are the key to testability.
*   **Test Doubles:** Because the application depends on an interface (Port), you can easily swap the heavy infrastructure implementation for a lightweight **Mock**, **Stub**, or **Fake** during testing.
*   **Speed:** This allows you to test business logic without spinning up a database or connecting to a network, significantly increasing test speed.

### Summary Checklist for Port Design
| Aspect | Guidance |
| :--- | :--- |
| **Location** | Interface defined in the Inner Circle (Domain/Use Case layer). |
| **Naming** | Named after the *purpose* (e.g., `OrderRepository`), not the technology (e.g., `PostgresDAO`). |
| **Inputs/Outputs** | Use Domain Objects or simple DTOs; avoid framework-specific types. |
| **Exceptions** | Catch technical exceptions (e.g., `SQLException`) in the Adapter and re-throw business exceptions defined in the Port. |
| **Granularity** | Follow the Interface Segregation Principle; create specific interfaces for specific clients. |