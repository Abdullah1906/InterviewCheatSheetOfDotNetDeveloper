# System Design & Software Architecture Interview Guide

**SOLID | Design Patterns | Clean Architecture | Security | Scalability**

## 1. SOLID Principles

### Q1. SOLID কী?
Object-oriented design-এর পাঁচটি principle:
- **S — Single Responsibility:** class/module-এর একটি প্রধান responsibility থাকা।
- **O — Open/Closed:** extension-এর জন্য open, modification-এর প্রয়োজন কম।
- **L — Liskov Substitution:** derived type base contract ভঙ্গ না করে substitute হতে পারবে।
- **I — Interface Segregation:** ছোট ও focused interface।
- **D — Dependency Inversion:** high-level policy abstraction-এর ওপর নির্ভর করবে।

### Q2. SRP-এর উদাহরণ?
একটি `EmployeeService`-এ employee business rule, SQL query, email sending এবং PDF generation সব না রেখে আলাদা service/repository-তে responsibility ভাগ করো।

### Q3. DIP কীভাবে ASP.NET Core-এ প্রয়োগ হয়?
Application layer `IEmployeeRepository` interface define করবে; Infrastructure layer implementation দেবে; DI container runtime-এ implementation inject করবে।

## 2. Design Patterns

### Q4. Repository Pattern কী?
Data access abstraction। Business/application logic সরাসরি SQL/data-access detail-এর ওপর নির্ভর না করে repository contract ব্যবহার করে।

### Q5. Unit of Work কী?
একটি business operation-এর একাধিক persistence operation-কে একই transaction boundary-তে coordinate করে।

### Q6. Factory Pattern কী?
Object creation logic centralize করে এবং caller-কে concrete creation detail থেকে আলাদা রাখে।

### Q7. Strategy Pattern কী?
একই contract-এর একাধিক algorithm/provider interchangeably ব্যবহার করা। যেমন `IPaymentGateway`-এর SSLCommerz, bKash বা অন্য provider implementation।

### Q8. Singleton Pattern কী?
একটি নির্দিষ্ট scope-এ shared instance নিশ্চিত করে। Mutable shared state থাকলে thread safety বিবেচনা করতে হবে।

### Q9. CQRS কী?
Command Query Responsibility Segregation—state পরিবর্তনের command এবং data পড়ার query আলাদা করা। সব project-এ CQRS প্রয়োজন নয়; complexity অনুযায়ী ব্যবহার করতে হয়।

## 3. Clean Architecture

### Q10. Clean Architecture কী?
Business rule-কে framework ও infrastructure থেকে আলাদা রাখার architecture style।

সাধারণ layer:
1. **Domain:** Entity, Value Object, Domain Rule।
2. **Application:** Use Case, Command/Query, DTO, Interface, Validation।
3. **Infrastructure:** ADO.NET, SQL Server, Email, File Storage, External Gateway।
4. **API/Presentation:** Controller, Middleware, HTTP contract।

### Q11. Dependency Rule কী?
Dependency direction core business logic-এর দিকে থাকবে। Domain layer ASP.NET Core, SQL Server বা Infrastructure-এর ওপর নির্ভর করবে না।

### Q12. Clean Architecture-এর সুবিধা ও trade-off?
সুবিধা: testability, maintainability, replaceable infrastructure, separation of concerns। Trade-off: বেশি project/interface/boilerplate; ছোট CRUD application-এ অতিরিক্ত abstraction হতে পারে।

## 4. API ও Distributed System

### Q13. Monolith ও Microservices?
Monolith-এ application একটি deployable unit হিসেবে থাকে। Microservices-এ business capability অনুযায়ী independently deployable service থাকে। Microservices-এর সঙ্গে network failure, observability, distributed data ও deployment complexity বাড়ে।

### Q14. API Gateway কী?
Client-এর জন্য single entry point; routing, authentication integration, rate limiting, aggregation ইত্যাদি করতে পারে।

### Q15. Message Queue কেন ব্যবহার হয়?
Producer ও consumer আলাদা করতে, asynchronous processing এবং temporary traffic buffering করতে। উদাহরণ: RabbitMQ, Azure Service Bus।

### Q16. Idempotency কী?
একই request একাধিকবার এলে intended business effect একবারের বেশি না হওয়া। Payment/order API-তে idempotency key ও database uniqueness ব্যবহার করা যায়।

### Q17. Eventual Consistency কী?
Distributed system-এ update সব replica/service-এ সঙ্গে সঙ্গে না পৌঁছালেও কিছু সময় পর consistent state-এ পৌঁছায়।

## 5. Database & Reliability

### Q18. ACID কী?
Atomicity, Consistency, Isolation, Durability—transaction-এর চারটি বৈশিষ্ট্য।

### Q19. Optimistic ও Pessimistic Concurrency?
- Optimistic: conflict হওয়ার সম্ভাবনা কম ধরে; update-এর সময় version/token দিয়ে conflict detect করে।
- Pessimistic: operation চলাকালে lock দিয়ে conflict ঠেকায়।
- SQL Server-এ `rowversion` optimistic concurrency implement করতে সাহায্য করে।

### Q20. Retry policy-তে কী সতর্কতা?
শুধু transient failure retry করো; exponential backoff ও jitter ব্যবহার করো। Non-idempotent operation blind retry করলে duplicate side effect হতে পারে।

### Q21. Circuit Breaker কী?
বারবার ব্যর্থ external dependency-তে request পাঠানো সাময়িক বন্ধ করে recovery সুযোগ দেয়। সাধারণ state: Closed, Open, Half-Open।

## 6. Security

### Q22. Authentication ও Authorization?
Authentication পরিচয় যাচাই; Authorization access permission যাচাই।

### Q23. JWT security consideration?
HTTPS, signature validation, issuer/audience validation, short expiry, key rotation এবং sensitive claim না রাখা জরুরি। JWT payload encrypted না-ও হতে পারে।

### Q24. XSS কী?
Attacker-এর script browser-এ execute হওয়া। Output encoding, safe DOM API, CSP এবং untrusted HTML sanitize করা প্রতিরোধে সহায়ক।

### Q25. CSRF কী?
Browser-এ authenticated session ব্যবহার করে attacker unwanted request trigger করে। Anti-forgery token, SameSite cookie ও origin validation বিবেচনা করো।

### Q26. SQL Injection কী?
Untrusted input SQL command-এর structure বদলে দেয়। Parameterized query ব্যবহার করো; dynamic identifier হলে allowlist validation প্রয়োজন।

## 7. Scalability ও Observability

### Q27. Caching-এর ধরন?
In-memory cache এক instance-এ; distributed cache একাধিক instance share করতে পারে। Cache invalidation, TTL ও stale data strategy design করতে হবে।

### Q28. Horizontal ও Vertical Scaling?
Vertical: machine-এর CPU/RAM বাড়ানো। Horizontal: একাধিক instance যোগ করা। Horizontal scaling-এ stateless design, shared cache/session এবং load balancing গুরুত্বপূর্ণ।

### Q29. Logging, Metrics, Tracing?
- Logs: নির্দিষ্ট event ও context।
- Metrics: সংখ্যাভিত্তিক time-series measurement।
- Traces: distributed request-এর service-to-service journey।
একত্রে observability উন্নত করে।

### Q30. Rate Limiting কী?
Client বা identity অনুযায়ী request quota enforce করে। IP-only limit-এর পাশাপাশি user/API key/tenant policy বিবেচনা করা যায়।

## Scenario-Based Questions

### Q31. Payment সফল কিন্তু API timeout—কী করবে?
1. Client retry-এর জন্য idempotency key দেবে।
2. Payment attempt persistent record করবে।
3. Gateway status server-to-server verify করবে।
4. Webhook signature validate করবে।
5. State transition idempotent করবে।
6. Audit trail রাখবে।
7. Pending state reconciliation job দিয়ে resolve করবে।

### Q32. Multi-tenant data isolation কীভাবে করবে?
Authenticated identity থেকে tenant context resolve করবে; প্রতিটি query/write tenant-scoped হবে; tenant ID শুধু frontend input থেকে বিশ্বাস করবে না; cross-tenant integration test লিখবে।

### Q33. Application slow—কোথা থেকে শুরু করবে?
Metrics/trace দিয়ে bottleneck identify; তারপর API latency, SQL execution plan, network, external service, memory/CPU profile বিশ্লেষণ। অনুমান করে optimization নয়—measure, change, measure।

## System Design Interview Framework

1. Functional requirement বোঝো।
2. Non-functional requirement: latency, availability, scale, security।
3. API contract ও data model নির্ধারণ।
4. High-level component diagram।
5. Critical data flow।
6. Failure mode ও recovery।
7. Security ও observability।
8. Trade-off ব্যাখ্যা।

## Revision Checklist

- [ ] SOLID
- [ ] Repository / Unit of Work / Strategy
- [ ] CQRS
- [ ] Clean Architecture dependency rule
- [ ] Monolith vs Microservices
- [ ] API Gateway ও Message Queue
- [ ] Idempotency ও Eventual Consistency
- [ ] Concurrency ও rowversion
- [ ] JWT, XSS, CSRF, SQL Injection
- [ ] Caching, scaling, observability
