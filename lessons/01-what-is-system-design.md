# Lesson 1 — What is System Design? ⭐⭐⭐⭐⭐

## 1. What is System Design?

Suppose we need to build an application where users can apply for jobs and track their applications.

At first we may think:

```text
Frontend  → Next.js
Backend   → Node.js
Database  → PostgreSQL
```

But before writing code, larger questions exist:

- How will the frontend communicate with the backend?
- Where will user data be stored?
- How will authentication work?
- What happens if 100,000 users use the application?
- What if the database becomes slow?
- What happens if a server crashes?
- How will scheduled reminders work?
- How do we protect the system from excessive requests?
- How will the application be deployed?

**System Design is the process of deciding the architecture, components, data flow, interfaces, storage, scalability, reliability, and trade-offs of a software system.**

---

## 2. Coding vs System Design

### Coding

Coding asks:

> How do I implement this feature?

It focuses on functions, variables, database queries, error handling, and code organization.

### System Design

System Design asks:

> How should the whole system work?

For authentication, for example:

- Where should authentication happen?
- Sessions or JWT?
- Where should sessions be stored?
- How do multiple backend servers authenticate users?
- How do we rate-limit failed login attempts?
- What happens if the authentication service fails?

```text
CODING
   └── How do I implement this feature?

SYSTEM DESIGN
   └── How should the whole system work?
```

---

## 3. Start Simple

A small CareerLoop architecture could be:

```text
USER
 │
 ▼
Next.js
 │
 ▼
Node.js
 │
 ▼
PostgreSQL
```

For a small number of users, this may be enough.

> **Good System Design does not mean using more technologies.**

Do not introduce microservices, Kafka, Redis, Kubernetes, or other infrastructure unless the requirements justify them.

---

## 4. What Happens When the System Grows?

Imagine:

```text
100 users
   ↓
10,000 users
   ↓
100,000 users
   ↓
1,000,000 users
```

One server may become overloaded.

We could introduce multiple servers and a load balancer:

```text
                    Users
                      │
                      ▼
               Load Balancer
                      │
             ┌────────┼────────┐
             ▼        ▼        ▼
          Server 1 Server 2 Server 3
             │        │        │
             └────────┼────────┘
                      ▼
                   Database
```

If repeated database reads become expensive, caching may help:

```text
                  Servers
                     │
              ┌──────┴──────┐
              ▼             ▼
            Redis        PostgreSQL
            Cache
```

For asynchronous work such as notifications:

```text
User
 │
 ▼
API
 │
 ├──────────────► Database
 │
 ▼
Message Queue
 │
 ▼
Worker
 │
 ▼
Email
```

The important thinking pattern is:

```text
Problem
   ↓
Requirement
   ↓
Possible Solution
   ↓
Trade-offs
   ↓
Architecture Decision
```

We do **not** add technologies randomly.

---

## 5. System Design Starts with Requirements

### Functional Requirements

Functional requirements describe **what the system should do**.

For CareerLoop:

- Register and log in
- Add a job application
- Update application status
- Delete an application
- Create follow-up reminders
- Search applications

### Non-Functional Requirements

Non-functional requirements describe **how well the system should work**.

Examples:

- Fast
- Scalable
- Secure
- Reliable
- Highly available
- Fault tolerant
- Maintainable

We will study these in detail in Lesson 3.

---

## 6. Important System Design Characteristics

### Scalability

Can the system handle increasing traffic?

```text
1K → 10K → 100K → 1M users
```

### Availability

Can users access the system when they need it?

If the only server crashes and the application becomes unavailable, that is an availability problem.

### Reliability

Does the system behave correctly and consistently?

Example:

```text
Customer pays ₹5,000

Payment successful ✅
Order creation failed ❌
Inventory incorrect ❌
```

This is a reliability problem.

### Performance

How quickly does the system respond?

Latency is one important performance measurement.

### Fault Tolerance

Can the system continue operating when a component fails?

```text
              Load Balancer
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
       Server 1  Server 2  Server 3
          ✅        ❌        ✅
```

One server fails, but the system continues operating.

---

## 7. System Design Is About Trade-offs ⭐⭐⭐⭐⭐

There is usually no perfect architecture.

Suppose we introduce Redis caching:

```text
Application
     │
     ▼
   Redis
     │
     ▼
 Database
```

Possible benefits:

- Better read performance
- Lower database load

But we also introduce:

- More infrastructure
- More complexity
- Additional cost
- Cache invalidation and consistency problems

Therefore, the question is not:

> Is Redis good?

The better question is:

> Does caching solve an important problem in this system, and are its benefits worth the additional complexity?

---

## 8. Technology Choices Depend on Requirements

Instead of asking:

> Which is better: PostgreSQL or MongoDB?

Ask:

- What data do we have?
- What relationships exist?
- What queries will we perform?
- Do we need transactions?
- What consistency guarantees do we need?
- How much data will we have?
- How will the data grow?

Then choose the technology.

```text
Requirement
    ↓
Problem
    ↓
Possible Solutions
    ↓
Trade-offs
    ↓
Decision
```

---

## 9. HLD and LLD

System Design can be viewed through two important levels:

```text
                    SYSTEM DESIGN
                          │
              ┌───────────┴───────────┐
              ▼                       ▼
             HLD                     LLD
      High-Level Design       Low-Level Design
```

### HLD

HLD focuses on the overall architecture:

```text
Users
  │
  ▼
Load Balancer
  │
  ▼
Backend
  │
 ┌┴────────────┐
 ▼             ▼
Redis       Database
```

Typical HLD questions:

- Which services are needed?
- Which database?
- Where should caching happen?
- Do we need a queue?
- How do services communicate?
- How will the system scale?

### LLD

LLD focuses on internal code/module design:

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

Typical LLD questions:

- Which classes/modules?
- Which interfaces?
- Which design patterns?
- How should responsibilities be separated?
- How should objects interact?

Lesson 2 will study HLD vs LLD in detail.

---

## 10. System Design vs DevOps

System Design decides **what architecture should exist**.

DevOps helps build, deploy, operate, and maintain that architecture.

For example:

```text
System Design:
"We need multiple backend instances."
             ↓
Infrastructure:
Docker + Kubernetes
             ↓
CI/CD:
GitHub → Jenkins → Build → Test → Image → Deploy
```

The areas are related, but they solve different problems.

---

## 11. The System Design Mindset ⭐⭐⭐⭐⭐

Whenever designing a system, think in this order:

```text
1. What problem am I solving?
             ↓
2. What are the requirements?
             ↓
3. What scale am I designing for?
             ↓
4. What are the possible solutions?
             ↓
5. What are their advantages/disadvantages?
             ↓
6. Which trade-offs are acceptable?
             ↓
7. Design the architecture.
```

Do not begin with:

> Which technology should I use?

Begin with the problem and requirements.

---

## 🎯 Interview Answer

**Q: What is System Design?**

> System Design is the process of defining the architecture, components, interfaces, data flow, and storage of a software system while considering requirements such as scalability, availability, reliability, performance, and maintainability. It involves making architectural decisions and evaluating trade-offs rather than simply selecting technologies.

A useful interview flow is:

```text
Requirements
     ↓
Estimate Scale
     ↓
Architecture
     ↓
Data
     ↓
Communication
     ↓
Scaling
     ↓
Reliability
     ↓
Trade-offs
```

---

## ⭐ Key Takeaway

> **System Design is not about knowing Redis, Kafka, Kubernetes, microservices, or drawing lots of boxes. It is about understanding a system's requirements and making appropriate architectural decisions and trade-offs to satisfy those requirements.**

---

**Next:** [Lesson 2 — HLD vs LLD](02-hld-vs-lld.md)
