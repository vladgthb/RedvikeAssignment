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
    - [Fixed Pseudo Code](#fixed-pseudo-code)

---

## How To Use This Document

I have addressed each task individually, clearly organizing instructions, explanations, diagrams, and architecture in their respective sections. Please refer to the Table of Contents to easily navigate through the document.

---

## Tasks

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

1. **Potential Problems**

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

#### Fixed Pseudo Code
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

**Key Improvements**
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

