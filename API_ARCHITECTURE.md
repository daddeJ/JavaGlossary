# Zoo Simulation & API Architecture: Layers, Patterns, and SOLID Principles

This document illustrates how a Zoo simulation maps to a typical API structure, including layers, design patterns, session management, and SOLID principles.

---

## 📍 Layers

### 1. Domain Layer (Core / Entities)
- Defines business objects:
  - Example: `Visitor`, `Enclosure`, `Ticket`
- Rules/validations at the entity level:
  - Example: "A Visitor cannot enter without a Ticket."
- No infrastructure dependencies.

### 2. Infrastructure Layer (Data Access, EF)
- Handles persistence and optimization:
  - Example: `EF Core` repositories, `DbContext`
- Tasks: queries, caching, transactions
- Example: `VisitorRepository` with `FindVisitorByIdAsync()`

### 3. Application / Service Layer
- Coordinates between Domain and Infrastructure.
- Implements business use cases:
  - Example: `TicketService.BuyTicket(visitorId)` calls repository + applies domain rules
- Optimizes EF queries:
  - `.AsNoTracking()`, projections, batching

### 4. Presentation Layer (API / UI)
- Controllers, REST endpoints, GraphQL resolvers
- Handles:
  - Authorization
  - Validation
  - Middleware pipeline
- Example: `POST /visitors/{id}/buy-ticket` calls `TicketService`

---

## 🔑 Key Points

- **EF (Infrastructure)** → Optimized data retrieval and insertion
- **API (Presentation)** → Communication + authorization
- **Application Layer** → Bridges domain and infrastructure, orchestrates logic
- **DI Container** → Wires layers together, manages lifetimes (Scoped, Singleton, Transient)

---

## 🕰️ Lifetime Scopes in Spring Boot

- **Singleton** → One instance for the whole app (services, repositories)
- **Prototype (Transient)** → New instance per injection
- **Request Scope** → One per HTTP request (like Visitor session)
- **Session Scope** → One per HTTP session (per logged-in user)

---

## 🏗️ Design Patterns Commonly Used in APIs

- **Singleton** → Services, configurations
- **Factory** → Dynamically build objects (e.g., Person, Animal)
- **Strategy** → Different ways to validate tickets, pay, enter enclosures
- **Observer / Pub-Sub** → Events (e.g., visitor enters → notify security)
- **Repository (DAO)** → Database access
- **Builder** → Complex object creation from requests
- **Adapter** → Integrating external APIs
- **Decorator** → Add behaviors like logging or security

---

## ✅ SOLID Principles in APIs

- **Single Responsibility** → Each service does one thing (e.g., `TicketValidator`, `ShopService`)
- **Open/Closed** → Add new enclosures without modifying old code
- **Liskov Substitution** → `Person` can be `Visitor` or `Staff` and still work
- **Interface Segregation** → Separate services for `Payable`, `Enterable`, etc.
- **Dependency Inversion** → Services depend on interfaces, not implementations

---

## 🦓 Zoo Simulation Mapping to API Layers

| Layer | Web API Equivalent | Zoo Example | Responsibility | Pattern |
|-------|-----------------|------------|----------------|---------|
| Presentation | Controllers / Endpoints | `ZooEntrance` | Accept requests, validate ticket | Facade |
| Application | Services / Use Cases | `TicketValidatorService`, `NavigationService` | Orchestrates workflow | Command / Strategy |
| Domain | Entities / Business Rules | `Visitor`, `Shop`, `Enclosure` | Pure business rules | Factory / Singleton |
| Infrastructure | Persistence / Session Management | `SessionManager` | Store & retrieve session state | Repository / Singleton |
| Cross-cutting | Middleware | Logging, Security | Consistent monitoring | Decorator / Proxy |

---

## 🔄 Visitor Lifetime (like HTTP Request)

1. **Presentation Layer**: Visitor arrives at Entrance (Controller)
2. **Application Layer**: TicketValidator checks validity
   - Valid → go to Enclosure
   - Invalid → redirected to Shop
3. **Domain Layer**: Person/Visitor interacts with Shop/Enclosure
4. **Infrastructure Layer**: SessionManager tracks movement (cache/queue)
5. **Exit**: Visitor leaves → session removed

---

This layered approach ensures **modular, maintainable, and scalable applications**, while the Zoo simulation provides a tangible example of how API architecture, design patterns, and SOLID principles work together.
