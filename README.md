# Redvike Assignment

## Overview

This repository presents a full-stack solution developed to meet the Redvike assignment requirements as part of the interview process. It features a scalable system design, a reliable order processing flow, and technical decisions that support clear business objectives.

Additionally, the repository includes code from the **Part 1** assignment with a fully dockerized setup for easy local installation, along with a minimal user interface powered by Swagger UI to visualize and test API interactions. This setup provides practical insights into my approach to clean coding, structured documentation, and effective containerization.

**Note:** Certain functionalities, such as detailed payments integration, are simplified or omitted intentionally to focus clearly on core architectural concepts and coding practices (clean code).

**Original Instructions:**

1. You may use your chosen tools and resources. Please make sure you use your own words to describe your reasoning.
2. Where diagrams are useful, feel free to provide a quick scatch or describe
   them clearly in the text.
3. This is not about “correct” answers but more about how you can approach
   complex issues and integrate business needs with technical design.

This assignment **README.md** document clearly outlines tasks, explanations, diagrams, and the proposed technical architecture to provide a comprehensive understanding of my approach and coding standards.

<div style="border: 1px solid red; padding: 10px; margin: 10px 0; background-color: #303030;">
  <strong style="color: red; font-size: 1.2em;">&#33; Important Note:</strong>
  <p style="margin: 5px 0 0 0;">
    Please note that there is no perfect architecture at the start of product development. As we onboard more clients, our database and overall architecture may need adjustments. Until we reach market fit, we need to expect significant changes. For a rapidly growing product, the ‘Modular Monolith’ pattern is a practical approach that allows for flexibility and easier refactoring.
  </p>
</div>

---

## Table of Contents

- [Overview](#overview)
- [How To Use This Document](#how-to-use-this-document)
- [Tasks](#tasks)
  - [Part 2: Code Analysis & Improvement](#part-2-code-analysis--improvement)
    - [Task Requirements (Part 2)](#task-requirements-part-2)
    - [My Explanation (Part 2)](#my-explanation-part-2)
      - [1. Identify Issues](#1-identify-issues)
      - [2. Improvements](#2-improvements)
      - [3. Broader Architectural Consideration](#3-broader-architectural-consideration)
      - [4. Fixed Pseudo Code](#4-fixed-pseudo-code)
  - [Part 3: Aligning Technology with Business](#part-3-aligning-technology-with-business)
    - [Task Requirements (Part 3)](#task-requirements-part-3)
    - [My Explanation (Part 3)](#my-explanation-part-3)

---

## How To Use This Document

I have addressed each task individually, clearly organizing instructions, explanations, diagrams, and architecture in their respective sections. Please refer to the Table of Contents to easily navigate through the document.

---

## Tasks

---

### Part 1: Real-World System Design

#### Task Requirements Part1
**Scenario**:
<br />
Company XYZ is launching an online marketplace that serves web and mobile
customers. <br /> The main requirements include:

1. Scalability & Availability - The system must be able to handle sudden spikes
   (e.g., flash sales, holiday shopping, marketing campaigns that suddenly drive
   traffic).
2. Data Consistency & Reliability - Orders must be processed reliably with
   emphasis on. consistent inventory updates.
3. Integration - Support for third-party payment providers and external inventory
   systems must be included.
4. Observability & Security - Robust monitoring/logging and strongauthorization
   measures.

**Tasks:**
<br />
1. Architecture Overview:
    - Provide a high-level design (diagram or detailed textual description) of the
      system. Include major components (e.g., API gateways, services,
      databases, message queues, etc.) and outline how they interact.
2. Design Rationale:
    - Explain your technology and design choices. Why did you choose
      particular patterns (e.g., microservices vs. modular monolith, eventual
      consistency vs. strong consistency in parts of the system, etc.)?
3. Handling Failures & Tradeoffs:
    - Identify potential bottlenecks, single points of failure, or performance issues.
    - Propose strategies to mitigate these risks.

#### My Explanation Part1

This section covers my proposed architecture and reasoning for **Part 1** of the assignment.

#### Architecture Overview

**High-Level Design**  
Below is a conceptual overview of how the system might look when delivering an online marketplace that handles both web and mobile customers:
```shell
┌─────────────────────────────────────────┐
│               Client Layer              │
│   (Web/Mobile - React, iOS, Android)    │
└─────────────────────────────────────────┘
                 │     ↑
    (HTTP/HTTPS) │     │ (HTTP/HTTPS)
                 ↓     │
┌─────────────────────────────────────────┐
│           API Gateway / Load Balancer   │
└─────────────────────────────────────────┘
                    │
                    ↓
┌─────────────────────────────────────────┐
│     Application Layer (Modular Monolith)│
│     - Express Server + TS               │
│       (Orders, Products, etc.)          │
│     - Common Modules                    │
│       for Auth, Payments, Observability │
└─────────────────────────────────────────┘
                    │
                    ↓
┌─────────────────────────────────────────┐
│         Database Layer (PostgreSQL)     │
│        + Caching (Redis or similar)     │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│ External Services / 3rd Parties         │
│  - Payment Providers (Stripe, PayPal)   │
│  - External Inventory Systems           │
└─────────────────────────────────────────┘
```

1. **Client Layer**:
    - Could be React-based web applications or mobile apps.
    - Communicates via REST (or potentially GraphQL) with the back end.

2. **API Gateway / Load Balancer**:
    - Provides a single entry point for clients, handling load balancing, rate limiting, and routing to the correct service instances if you scale horizontally.

3. **Application Layer (Modular Monolith)**:
    - Houses the core business logic for Orders, Products, Authentication, etc.
    - Organized into modules, as shown in the folder structure.
    - Provides a foundation that can be split out into microservices once traffic or feature demands necessitate it.

4. **Database Layer**:
    - Relational database (e.g., PostgreSQL) with ACID transactions ensures data consistency.
    - Caching layers (e.g., Redis) may be added to improve response times, especially for frequently accessed data.

5. **External Services**:
    - Payment providers (Stripe/PayPal) for handling transactions.
    - External inventory APIs to sync stock levels if you have multiple sales channels.
    - Observability tools (e.g., Datadog, Prometheus) for monitoring.

---

#### 2. Design Rationale

1. **Technology Choices**:
    - **Modular Monolith**:
        - Eases development in the early stages.
        - Maintains clear boundaries (Orders, Products, Payments), simplifying future refactoring.
        - Reduces operational overhead compared to microservices (fewer deployments, simpler networking).
    - **TypeScript + Node.js**:
        - Type safety ensures fewer runtime errors.
        - Excellent ecosystem and developer productivity.
    - **PostgreSQL**:
        - Provides robust transactions for critical operations like order creation, payments, and inventory updates.
        - Widely supported in the Node ecosystem, easy to scale vertically or horizontally.

2. **Strong vs. Eventual Consistency**:
    - **Orders and Payments**: Use strong consistency (ACID transactions) to avoid partial updates or missed inventory decrements.
    - **Analytics or Inventory Sync with External Systems**: Eventual consistency can be acceptable—updates can happen asynchronously (e.g., via message queues).

3. **Scalability and Availability**:
    - **Horizontal Scaling**: Spin up multiple instances of the main application behind a load balancer.
    - **Caching**: Use Redis (or similar) to cache frequently accessed data (e.g., product listings) to reduce DB load.
    - **Database Scalability**: Implement read replicas, or partition data if the user base grows exponentially.

---

#### 3. Handling Failures & Tradeoffs

1. **Potential Bottlenecks & Single Points of Failure**:
    - **Database**: If the database goes down or becomes a performance bottleneck, the entire system can stall. Mitigation includes replication, failover strategies, and robust backup policies.
    - **Network Spikes**: Sudden traffic surges (flash sales) can overwhelm the app layer. Horizontal scaling and queue-based processing help mitigate.
    - **3rd-Party Dependencies**: Payment providers can fail or slow down. Apply circuit breakers and proper retry logic.

2. **Strategies to Mitigate Risks**:
    - **Message Queues**: Decouple certain tasks (like sending order confirmation emails, performing analytics) to avoid holding up core order-processing requests.
    - **Retries and Dead-Letter Queues**: For external service calls, implement exponential backoff and dead-letter queues to handle repeated failures gracefully.
    - **Monitoring & Observability**: Real-time monitoring (Datadog, Prometheus) plus structured logging (Winston, Pino) to detect and resolve issues quickly.
    - **Circuit Breaker Pattern**: For third-party integrations, break the connection if requests repeatedly fail, preventing the entire system from slowing down.

3. **Tradeoffs**:
    - **Modular Monolith vs. Microservices**:
        - **Short-Term**: A modular monolith is simpler for a startup environment and reduces DevOps overhead.
        - **Long-Term**: If the application grows, one or more modules can be extracted into separate microservices for independent scaling, or to isolate failure domains.
    - **Strong Consistency vs. Eventual Consistency**:
        - **Critical Data** (Orders, Payments): Must be accurate and up-to-date to prevent over-selling or transaction issues.
        - **Non-Critical Data** (Analytics, Reporting): Can often tolerate delays. Reduces load on the primary database.


```shell
RedvikeAssignment/
├── README.md                                 # Contains All Information about tasks, my explanation and code architecture
├── docker-compose.yml                        # Docker configuration for multi-container setup
├── backend
│   ├── package.json
│   ├── tsconfig.json
│   ├── src
│   │   ├── app
│   │   │   ├── routes
│   │   │   │   └── index.ts                  # Aggregates and exposes all module routes
│   │   │   ├── middlewares
│   │   │   ├── controllers
│   │   │   │   └── health.controller.ts      # Example: global controllers
│   │   │   └── server.ts                     # Central Express setup (entry point)
│   │   ├── core
│   │   │   ├── config                        # Configuration logic (env, secrets)
│   │   │   │   └── index.ts
│   │   │   └── infrastructure
│   │   │       └── db
│   │   │           ├── prismaClient.ts       # e.g., Prisma client instance
│   │   │           └── migrations            # if using raw SQL migrations or similar
│   │   ├── modules
│   │   │   ├── orders
│   │   │   │   ├── domain
│   │   │   │   │   ├── entities              # Domain entity definitions
│   │   │   │   │   │   └── order.entity.ts
│   │   │   │   │   └── value-objects         # Value objects, e.g. OrderId, Price
│   │   │   │   ├── application
│   │   │   │   │   ├── dto                   # Data Transfer Objects
│   │   │   │   │   │   └── createOrder.dto.ts
│   │   │   │   │   └── services              # Application services containing business logic
│   │   │   │   │       └── order.service.ts
│   │   │   │   ├── infrastructure
│   │   │   │   │   ├── repositories          # Database interaction, queries
│   │   │   │   │   │   └── order.repository.ts
│   │   │   │   │   └── mappers               # Converting domain objects to DB records
│   │   │   │   └── interfaces
│   │   │   │       └── order.controller.ts   # Module-specific Express endpoints
│   │   │   └── products
│   │   │       ├── domain
│   │   │       │   └── entities
│   │   │       │       └── product.entity.ts
│   │   │       ├── application
│   │   │       │   └── services
│   │   │       │       └── product.service.ts
│   │   │       ├── infrastructure
│   │   │       │   └── repositories
│   │   │       │       └── product.repository.ts
│   │   │       └── interfaces
│   │   │           └── product.controller.ts
│   │   └── index.ts                          # A single aggregator entry point
├── frontend
│   ├── package.json
│   ├── tsconfig.json
│   ├── public
│   └── src
│       ├── index.tsx                         # React entry point
│       ├── App.tsx                           # Main app component
│       ├── components
│       │   └── OrderForm.tsx                 # Order form UI component
│       └── services
│           └── api.ts                        # API integration layer for HTTP requests
```

---

### Part 2: Code Analysis & Improvement

#### Task Requirements (Part 2)

**Given:**
Below is a simplified pseudocode snippet for an order creation endpoint. Review it
and answer the questions that follow.

```python
from flask import jsonify

def create_order(request):
    order_data = request.get_json()
    order_id = generate_unique_id()
    
    try:
        db.insert('orders', {'id': order_id, **order_data})
    except Exception as e:
        return jsonify({"error": "Order creation has failed"})
    
    product_id = order_data.get('productId')
    try:
        db.execute(f"UPDATE products SET stock = stock - 1 WHERE id = {product_id}")
    except Exception as e:
        return jsonify({"error": "Inventory update has failed"})
    
    return jsonify({"orderId": order_id, "status": "created"})
```

**Tasks:**

Here are the questions:
1. Identify Issues:
   a. What are the potential problems with this implementation regarding data
   consistency, error handling, concurrency, and security?
2. Improvements:
   a. How would you refactor or redesign this flow to ensure that order creation
   and inventory updates are reliable and consistent, especially in a high-load
   production environment?
   b. Discuss whether a transaction or a compensating strategy might be
   appropriate.
3. Broader Architectural Consideration:
   a. How could you structure this part of the system (or the entire flow) to
   better decouple concerns while still ensuring that business rules are
   enforced?

#### My Explanation Part 2

In this part, I look at the original pseudocode to uncover some real-world pitfalls—things like partial updates, race conditions, and unclear error messages. Below are the most common issues and how they can affect reliability, consistency, and security:

##### 1. Identify Issues

1. Data Consistency
    - Partial Updates: If the order is saved successfully but the inventory update fails, we end up with an order that doesn’t reduce stock.
    - No Atomicity: Each database operation runs on its own, so there’s no automated rollback mechanism if anything goes wrong.
2. Concurrency
    - Under heavy load, multiple users might purchase the same item at once. Without proper safeguards, stock could go negative or get oversold.
3. Error Handling
    - Currently, the code just returns generic messages and doesn’t revert changes if one step fails.
    - Missing structured logging or standardized error formats can make diagnosing problems a headache.
4. Security
    - The snippet doesn’t show parameterized queries, so there’s a risk of SQL injection if string concatenation is used.
    - No authentication or authorization checks—anyone could potentially create orders or modify stock.

##### 2. Improvements

   1. **Refactoring or Redesigning for Reliability & Consistency (High-Load Production)**

      - Use Transactions:
        - By wrapping order creation and stock updates in a single transaction, the database can automatically roll back if any step fails. This ensures that we never end up with mismatched order/stock data.
      - Parameterized Queries:
        - Protecting against SQL injection by passing parameters securely rather than building queries with string concatenation.
      - Concurrency Checks:
        - Ensuring stock updates happen only if stock > 0; if no rows are updated, we know it’s out of stock or there’s a concurrency conflict.
        - Depending on our database, consider row-level locks or appropriate isolation levels.
      - Clear Error Handling:
        - Differentiate 4xx client errors (like missing productId) from 5xx server errors (like a DB failure).
        - Providing more detailed logs so issues are easier to diagnose.

   2. **Transaction vs Compensating Strategy**

      - Transaction Approach:
        - Best suited for simpler, monolithic applications. Everything is in the same database, so a single transaction is straightforward and robust.
        - Great for smaller codebases or early-stage products/startups where we need guaranteed consistency and don’t have many external dependencies.
      - Compensating Actions (Saga Pattern):
        - Useful if the process is spread across multiple services or requires asynchronous steps.
        - Each step that fails triggers a “compensating” action to undo partial work (e.g., refunding a charge if the order can’t be finalized).
        - This can add complexity, so it’s usually more relevant in larger, distributed systems.

##### 3. Broader Architectural Consideration

   1. **Structuring for Decoupling & Enforcing Business Rules**

      - Decoupling:
        - Keeping order logic and inventory logic in separate modules but having a clear interface (or service layer) that orchestrates them. This way, we can isolate bugs and changes more easily.
        - If we eventually move to microservices, we can split them out with minimal refactoring (e.g., an “Order Service” and an “Inventory Service” communicating via a message bus).
      - Business Rules Enforcement:
        - Centralize core rules (like “must have stock > 0 to create an order”) in the domain or application service layer. This prevents direct DB calls from bypassing critical logic.
        - Ensuring consistent validations, logging, and error handling so each part of the system responds predictably when constraints aren’t met.

Overall, the key is to start with a transactional, modular approach that keeps our code clean and consistent, and then evolve to more sophisticated patterns (like compensating actions or fully decoupled services) as the business and technical needs grow.

##### 4. Fixed Pseudo Code

```python
from flask import jsonify

def create_order(request):
    order_data = request.get_json()
    
    # Basic validation
    product_id = order_data.get('productId')
    if not product_id:
        return jsonify({"error": "productId is required"}), 400

    order_id = generate_unique_id()

    try:
        # Begin a transaction (pseudo-code depends on our DB/ORM library)
        with db.transaction():
            # 1. Insert the new order record
            db.insert('orders', {'id': order_id, **order_data})

            # 2. Decrement the product stock atomically
            #    Parameterized query to prevent SQL injection
            #    'stock > 0' ensures we don't oversell
            result = db.execute(
                """
                UPDATE products
                SET stock = stock - 1
                WHERE id = %s
                  AND stock > 0
                RETURNING stock
                """,
                (product_id,)
            )

            # If no rows are returned, the product might be out of stock or invalid
            if not result or result[0]['stock'] < 0:
                raise Exception("Insufficient stock or concurrency conflict")

    except Exception as e:
        # Any exception inside the transaction automatically triggers rollback
        return jsonify({"error": str(e)}), 500

    # Successful transaction
    return jsonify({"orderId": order_id, "status": "created"}), 201
```

##### 2. Improvements: Key Improvements

   1. Transactions
      - By wrapping the order creation and stock update in a single transaction, if something goes wrong, everything rolls back automatically. This prevents those awkward partial updates where the order is created but the stock isn’t updated.
   2. Concurrency Controls
      - Adding WHERE stock > 0 ensures we don’t accidentally oversell when lots of people place orders at once.
      - If the query doesn’t return a row, we know there’s a concurrency conflict or not enough stock, so the process is immediately stopped.
   3. Parameterized Queries
      - By passing values securely into the query (rather than building strings), we protect against SQL injection attempts and keep data safe.
   4. Clear Error Handling
      - We send a 400 status if productId is missing, so the client immediately knows the request wasn’t valid.
      - We use a 500 status if the database operation fails (and log details on our end), ensuring any errors are rolled back without messing up the data.
   5. Meaningful Responses
      - We differentiate user mistakes (like missing productId) from bigger, internal issues (like a DB failure).
      - This way, the client can respond appropriately—maybe prompting the user to fix their input or retry the request if it’s a server-side hiccup.

---

### Part 3: Aligning Technology with Business

#### Task Requirements (Part 3)

**Given:**
Describe the following:
1. How do you ensure that the technical solutions you design align with and
   support the broader business goals of an organization, especially in rapidly
   changing markets?
2. Describe a real or hypothetical scenario where you had to balance factors
   such as technical debt, scalability, and evolving business requirements.
3. What was your approach, what tradeoffs did you consider, and what was the
   outcome?

#### My Explanation (Part 3)

1. **Ensuring Technical Solutions Align with Broader Business Goals**
   
   In fast-moving markets, we make a point to regularly meet with stakeholders—whether that’s product owners, marketing teams, or executive leadership—to understand current and upcoming priorities. We keep our eyes on the big picture: revenue drivers, user satisfaction metrics, and strategic deadlines. Whenever we propose a new feature or architectural change, we tie it back to a tangible business benefit, like improving user onboarding speed or reducing infrastructure costs. By demonstrating how each technical decision advances the company’s objectives, we stay in sync with evolving requirements.

2. **Real (or Hypothetical) Scenario Balancing Technical Debt, Scalability, and Evolving Requirements**

   Imagine we started out with a single-customer MVP for an e-commerce platform. Initially, everything was in one big file, and we took a few shortcuts (ahem, “technical debt”) to launch quickly. However, when two more clients came onboard, each with unique product catalog structures and payment preferences, we realized the existing code was about to collapse under the load.

   - Scalability: We anticipated the next few clients could bring in traffic spikes, so we introduced a more modular design—splitting product management, orders, and payment logic into separate domains within our monolith.
   - Evolving Requirements: Each client had distinct reporting needs, so we leveraged a flexible data model and a small analytics service that could adapt without touching core order logic.
   - Technical Debt: Instead of a massive rewrite, we tackled refactoring incrementally, cleaning up the parts of the code that caused the biggest issues first. We created a “tech debt” backlog and scheduled time each sprint to fix or improve one piece of the system.

3. **Our Approach, Tradeoffs, and Outcome**

   - Approach: We combined a short-term focus on shipping essential features with a mid-term plan to pay down the most harmful technical debt. We also established a consistent release process, including automated tests to reduce the risk of breaking critical functionality.
   - Tradeoffs: Some lower-priority features were delayed so we could stabilize the codebase. We accepted minor performance hits in non-critical areas to meet hard deadlines in others.
   - Outcome: We successfully onboarded new clients without major disruptions, and the system evolved into a more maintainable architecture. Our technical improvements also gave us a stable foundation to roll out new features faster, ultimately supporting both our company’s revenue goals and each client’s unique requirements.