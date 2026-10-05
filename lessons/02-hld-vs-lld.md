# Lesson 2 — HLD vs LLD ⭐⭐⭐⭐⭐

System Design can be understood at different levels of abstraction. Two of the most important are **High-Level Design (HLD)** and **Low-Level Design (LLD)**.

> **Simple mental model:** HLD = zoom out and understand the whole architecture. LLD = zoom in and design the internals of a component.

---

## 1. The Big Picture

Consider an e-commerce application such as ShopHub.

At a high level:

```text
                         Users
                           │
                           ▼
                      Next.js App
                           │
                           ▼
                       Backend
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
          MongoDB        Redis       Payment
                                    Gateway
```

This answers questions about major components, communication, storage, caching, and external services.

That is **High-Level Design (HLD)**.

Now zoom inside the payment component:

```text
PaymentService
      │
      ├── PaymentProvider
      │       ├── CashfreeProvider
      │       └── RazorpayProvider
      │
      ├── PaymentRepository
      └── PaymentValidator
```

Now we are discussing modules, interfaces, responsibilities, methods, and code organization.

That is **Low-Level Design (LLD)**.

```text
SYSTEM DESIGN
     │
     ├────────────── HLD
     │                └── Architecture
     │
     └────────────── LLD
                      └── Internal / Code Design
```

---

## 2. What is HLD?

**HLD = High-Level Design**

HLD describes the **overall architecture of a system** and focuses on the major components and how they interact.

Example for CodeBuddy:

```text
                       Users
                         │
                         ▼
                    Frontend
                         │
                         ▼
                   Load Balancer
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
          Backend 1             Backend 2
              │                     │
              └──────────┬──────────┘
                         │
             ┌───────────┼────────────┐
             ▼           ▼            ▼
          MongoDB      Redis       Socket.IO
```

We are not discussing individual JavaScript functions or classes. We are discussing the system as a whole.

### Typical HLD Questions

- What major services are required?
- Which database should we use?
- How do services communicate?
- Where should caching happen?
- Do we need a message queue?
- How will the system scale?
- How do we handle failures?
- How do we distribute traffic?

Typical HLD components include:

- Clients
- APIs
- Backend services
- Load balancers
- API gateways
- Databases
- Caches
- CDNs
- Message queues
- Object storage
- WebSocket servers
- Workers
- Microservices

---

## 3. What is LLD?

**LLD = Low-Level Design**

LLD describes the **internal design of an application, service, or component**.

Instead of asking:

> What services does our system need?

we ask:

> How should this particular service be structured internally?

Suppose HLD contains:

```text
Frontend
   │
   ▼
Order Service
   │
   ▼
Database
```

LLD opens the Order Service:

```text
OrderService
     │
     ├── OrderRepository
     ├── InventoryService
     ├── PaymentService
     └── NotificationService
```

We may then define operations such as:

```text
OrderService

+ createOrder()
+ cancelOrder()
+ getOrder()
+ updateOrderStatus()
```

Typical LLD concerns include:

- Classes
- Modules
- Interfaces
- Methods
- Object relationships
- SOLID principles
- Design patterns
- Dependency management
- Separation of responsibilities

---

## 4. HLD vs LLD Using One Example ⭐⭐⭐⭐⭐

Imagine an e-commerce system.

### HLD

```text
                         Users
                           │
                           ▼
                          CDN
                           │
                           ▼
                     Load Balancer
                           │
                           ▼
                       Backend
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
          Product        Order         Payment
          Service        Service       Service
             │             │             │
             ▼             ▼             ▼
          Database      Database      Payment
                                      Gateway
```

Here we discuss:

- Services
- Databases
- Communication
- Scalability
- Caching
- Load balancing
- Reliability

This is **HLD**.

### LLD

Now zoom into the Payment Service:

```text
                    PaymentService
                           │
                           ▼
                    PaymentProvider
                    /             \
                   /               \
                  ▼                 ▼
       CashfreeProvider      RazorpayProvider
                  │                 │
                  └────────┬────────┘
                           ▼
                  PaymentRepository
```

An interface could conceptually provide:

```text
PaymentProvider

createPayment()
verifyPayment()
refundPayment()
```

CashfreeProvider and RazorpayProvider can implement the same abstraction.

This is **LLD**.

---

## 5. Easy Analogy: City Map

Think of a city map viewed from far away:

```text
City
 │
 ├── Airport
 ├── Railway Station
 ├── Hospital
 ├── Highway
 └── Shopping Mall
```

You understand how major locations connect.

That is like **HLD**.

Now enter the hospital:

```text
Hospital
 │
 ├── Reception
 ├── Emergency
 ├── Pharmacy
 ├── ICU
 ├── Operation Theatre
 └── Patient Rooms
```

Now you are looking at its internal structure.

That is like **LLD**.

```text
HLD = Zoom Out 🔭
LLD = Zoom In  🔎
```

---

## 6. Database Design Can Appear at Both Levels

At HLD:

```text
User Service
     │
     ▼
 PostgreSQL
```

We may discuss:

- PostgreSQL vs MongoDB
- Replication
- Sharding
- Read replicas
- Database availability

This is mostly HLD.

At a more detailed level:

```text
users
----------------
id
name
email
password_hash
created_at

applications
----------------
id
user_id
company
position
status
created_at
```

And:

```text
User
 │ 1
 │
 │ N
 ▼
Applications
```

Now we are closer to detailed schema/data design.

Therefore, HLD and LLD are **levels of abstraction**, not completely isolated categories.

---

## 7. API Design Can Also Appear at Both Levels

At HLD:

```text
Frontend
   │
   │ REST API
   ▼
Backend
```

This describes how major components communicate.

At a detailed level:

```text
POST   /api/orders
GET    /api/orders/:id
PATCH  /api/orders/:id
DELETE /api/orders/:id
```

Now we are defining detailed interfaces.

Think of it as:

```text
HLD
 │
 │ progressively more detail
 ▼
LLD
```

---

## 8. HLD Usually Comes First

When building a house, we normally decide the overall layout before choosing details such as a bedroom door handle.

Software follows a similar flow:

```text
Requirements
      ↓
     HLD
      ↓
     LLD
      ↓
Implementation
```

First understand the requirements.

Then design the major architecture.

Then design individual components.

Finally implement them.

---

## 9. CareerLoop Example

### CareerLoop HLD

An early architecture could be:

```text
                         User
                          │
                          ▼
                     Next.js App
                          │
                          ▼
                     Backend/API
                          │
              ┌───────────┴───────────┐
              ▼                       ▼
          PostgreSQL              Background
                                   Worker
                                      │
                                      ▼
                                  Reminder
                                   Emails
```

At larger scale:

```text
                         Users
                           │
                           ▼
                      Load Balancer
                           │
               ┌───────────┴───────────┐
               ▼                       ▼
           Server 1                Server 2
               │                       │
               └───────────┬───────────┘
                           │
                 ┌─────────┼─────────┐
                 ▼         ▼         ▼
              Redis    PostgreSQL   Queue
                                     │
                                     ▼
                                   Worker
```

This is HLD.

### CareerLoop LLD

Zoom into application management:

```text
ApplicationService
       │
       ├── ApplicationRepository
       ├── ApplicationValidator
       └── ReminderService
```

Possible service operations:

```text
createApplication()
updateApplication()
deleteApplication()
getApplication()
listApplications()
```

Possible repository operations:

```text
create()
findById()
findByUser()
update()
delete()
```

This is LLD.

---

## 10. CodeBuddy Example

At HLD:

```text
                      Users
                        │
                        ▼
                  React Frontend
                        │
                ┌───────┴────────┐
                │                │
              HTTP           WebSocket
                │                │
                ▼                ▼
           Express API      Socket.IO
                │                │
                └───────┬────────┘
                        │
                    MongoDB
```

This describes the architecture.

Now zoom into chat:

```text
ChatService
    │
    ├── RoomService
    ├── MessageRepository
    ├── SocketHandler
    └── MessageValidator
```

This is LLD.

---

## 11. HLD vs LLD Comparison

| HLD | LLD |
|---|---|
| Overall architecture | Internal component design |
| Services | Classes/modules |
| Databases | Models/entities |
| Load balancers | Methods/functions |
| Caches | Interfaces |
| Message queues | Design patterns |
| Service communication | Object interaction |
| Scalability | Code maintainability |
| Availability | SOLID principles |
| Fault tolerance | Dependency management |

### Simple distinction

> **HLD tells us what major components the system contains and how they interact. LLD tells us how those components are internally designed and implemented.**

---

## 12. HLD and LLD for a Full-Stack Developer

A full-stack developer should understand both.

A practical priority is:

```text
                    SYSTEM DESIGN
                          │
             ┌────────────┴────────────┐
             │                         │
            HLD                       LLD
          ⭐⭐⭐⭐⭐                    ⭐⭐⭐⭐
             │                         │
       Scalability                    SOLID
       Databases                      Patterns
       Caching                        Modules
       Queues                         Interfaces
       Load Balancing                Clean Design
       Reliability
```

For this curriculum, we will spend significant time on practical HLD while learning enough LLD to design clean applications and backend services.

---

## 13. Interview Scenario

Suppose the interviewer asks:

> Design an online shopping platform.

Do not immediately answer with a list of technologies.

First understand requirements, then design HLD:

```text
                       Client
                         │
                         ▼
                    API Gateway
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       Product         Order         Payment
       Service         Service       Service
          │              │              │
          ▼              ▼              ▼
         DB             DB          Payment
                                     Gateway
```

If the interviewer then asks:

> Design the Payment Service internally.

Switch from HLD to LLD:

```text
PaymentService
      │
      ▼
PaymentProvider
   /        \
  ▼          ▼
Cashfree   Razorpay
```

Recognizing when to change abstraction level is an important System Design skill.

---

## 14. Important Misconception

Do not think:

```text
HLD = Senior Developer
LLD = Junior Developer
```

Both are important.

Developers frequently use LLD when designing modules and features. Senior engineers may spend more time on HLD because architectural decisions can affect multiple teams and systems.

The main difference is **scope and abstraction**, not job title.

---

## 🎯 Interview-Ready Answer

### Q: What is the difference between HLD and LLD?

> **High-Level Design (HLD)** describes the overall architecture of a system, including major components such as services, databases, caches, load balancers, queues, and how those components communicate and scale.
>
> **Low-Level Design (LLD)** describes the internal implementation of those components, including classes, modules, interfaces, methods, object relationships, SOLID principles, and design patterns.

Example:

```text
E-commerce

HLD:
Client
  ↓
Order Service
  ↓
Database

LLD:
OrderService
  ├── OrderRepository
  ├── InventoryService
  ├── PaymentService
  └── NotificationService
```

---

## 🧠 Lesson 2 Mental Model

```text
                 REQUIREMENTS
                      │
                      ▼
             ┌─────────────────┐
             │       HLD       │
             │                 │
             │ System          │
             │ Architecture    │
             └────────┬────────┘
                      │
                      ▼
             ┌─────────────────┐
             │       LLD       │
             │                 │
             │ Component /     │
             │ Code Design     │
             └────────┬────────┘
                      │
                      ▼
               IMPLEMENTATION
```

The shortest version to remember:

```text
HLD = WHAT major components exist + HOW they connect

LLD = HOW each component is internally designed
```

---

## ⭐ Key Takeaway

Think about **zoom level**:

```text
               SYSTEM
                 │
        HLD ─────┤ ← Zoomed out
                 │
                 ▼
              SERVICE
                 │
        LLD ─────┤ ← Zoomed in
                 │
                 ▼
          Classes / Modules
```

Do not only memorize definitions. Learn to recognize the level of abstraction being discussed.

---

**Previous:** [Lesson 1 — What is System Design?](01-what-is-system-design.md)

**Next:** [Lesson 3 — Functional vs Non-Functional Requirements](03-functional-vs-non-functional-requirements.md)
