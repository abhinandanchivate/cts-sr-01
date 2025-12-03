

# 📄 **CASE STUDY DOCUMENT**

## **SmartMart – Real-Time Order Fulfillment Engine**

### **Callback + Arrow Function Architecture (JavaScript)**

---

# 🏢 **1. Background**

SmartMart is a mid-size e-commerce company dealing with **electronics, home appliances, and accessories**.
Customer orders arrive via Website, Mobile App, and Partner Platforms.
In 2025, SmartMart wants to modernize its **Order Fulfillment Engine** using **JavaScript callback-based plug-ins** to support:

* New discounting logic
* Dynamic warehouse allocations
* Real-time analytics events
* Fraud checks
* Third-party payment gateway integrations

The key business goal is to enable **flexibility and extensibility**—allowing business teams to plug in new rules without rewriting the core pipeline.

---

# 🎯 **2. Business Problem**

The existing monolithic fulfillment pipeline suffers from:

* Hardcoded validation rules
* Sequential processing → slow order placement
* No plug-in capability for discount/loyalty rules
* Payment and inventory services tightly coupled
* No event-based notifications to Analytics or Fraud systems

This causes:

* Slow order confirmations
* Frequent code changes
* High risk during deployment
* Difficulty adding new promotions or warehouses

---

# 🧪 **3. Goal of the New Engine**

The new engine must:

1. Use **callback-based plug-in design** for each processing stage
2. Use **arrow functions** to define lightweight rule logic
3. Support **parallel execution** (e.g., Inventory check + Coupon calculation simultaneously)
4. Implement an **async pipeline** that merges results
5. Use a **final callback** to return the complete processed order
6. Emit **audit, analytics, and fraud events** without blocking the pipeline
7. Allow each functional module to be **replaced or extended** independently

---

# 🧱 **4. Functional Workflow**

A single order goes through these mandatory stages:

### **Step 1 — Customer Validation**

* Validate customer identity
* Check account status (Active/Blocked/GOLD/SILVER)
* Callback invoked: `(error, customerData)`

---

### **Step 2 — Fetch Cart Items**

* Retrieve customer’s cart
* Resolve product SKUs, quantity, pricing
* Callback invoked: `(error, cartItems)`

---

### **Step 3 — Run Parallel Processes**

Two functions run **simultaneously** using callbacks:

#### **3.1 Coupon & Discount Engine (Callback Plug-in)**

* Check if customer is eligible for coupon
* Apply loyalty-based offer (e.g., 10% for GOLD users)

#### **3.2 Inventory Allocation Engine (Callback Plug-in)**

* Select best warehouse
* Validate stock availability
* Confirm allocation count

Outputs are merged after both callbacks complete.

---

### **Step 4 — Payment Processing**

* Forward final amount to payment gateway
* Handle retries for failures
* Payment callback returns status & transaction ID

---

### **Step 5 — Event Publishing**

Three asynchronous callbacks emit:

1. **AuditLog Event**
2. **Analytics Event**
3. **Fraud Detection Event**

These run independently without blocking the main pipeline.

---

### **Step 6 — Final Invoice Generation**

A final callback function returns:

* Customer details
* Cart details
* Discount summary
* Selected warehouse
* Final amount after discount
* Payment confirmation
* Timestamp
* Event publishing status

This is delivered to the UI / mobile app instantly.

---

# 🧠 **5. Why Callback + Arrow Function Architecture?**

### ✔ Dynamic Business Rules

Example:

```js
const goldDiscountRule = order => order.amount * 0.10;
```

Business teams can plug in new rules without touching the core pipeline.

---

### ✔ Parallel Execution

Inventory lookup & coupon lookup can run together using **non-blocking callbacks**.

---

### ✔ Extensibility

Each stage exposes a callback interface, allowing:

* Warehouse selection plug-ins
* Custom fraud detection modules
* A/B testing for different discount strategies

---

### ✔ Real-World Fail Handling

Callback error channels support:

* Payment gateway failure
* Inventory service down
* Cart service timeout
* Coupon engine crash

Pipeline can abort and return meaningful error messages.

---

# 🔐 **6. Non-Functional Requirements**

### Performance

* Entire pipeline must complete < 2 seconds under peak load
* Parallel callbacks reduce latency by 40%

### Reliability

* Each step must provide controlled failure paths
* Default retry logic for unstable services

### Scalability

* Event emitter callbacks integrate with Kafka / PubSub
* Plug-in architecture supports multiple warehouses or coupon engines

### Security

* Customer validation callback enforces identity checks
* All analytics callbacks sanitize PII

---

# 💡 **7. Use Cases Supported**

### **UC1: Order with Loyalty Discount**

* GOLD user receives 10% discount
* Warehouse allocation by nearest location
* Payment processed
* Events emitted

### **UC2: Order with No Discount**

* Silver user
* Coupon engine callback returns neutral rule
* Normal pricing applied

### **UC3: Out-of-stock Scenario**

* Inventory callback fails
* Pipeline aborts
* Final callback returns a stock error

### **UC4: Payment Failure**

* Payment callback rolls back warehouse allocation
* Pipeline ends with Payment Failed status

### **UC5: Fraud Suspicion**

* Fraud emitter flags high-value order
* Pipeline still continues, but fraud event escalates

---

# 🗺 **8. ARCHITECTURE FLOW DIAGRAM**

```
                                     ┌──────────────────────────────┐
                                     │        Order Request          │
                                     └──────────────────────────────┘
                                                 │
                                                 ▼
                                   ┌───────────────────────────┐
                                   │ 1. Customer Validation    │
                                   │ (callback plug-in)        │
                                   └───────────────────────────┘
                                                 │
                                                 ▼
                                   ┌───────────────────────────┐
                                   │   2. Cart Fetch           │
                                   │   (callback plug-in)      │
                                   └───────────────────────────┘
                                                 │
                           ┌────────────────────────────────────────────────┐
                           │                  PARALLEL                      │
                           │                                                │
                           ▼                                                ▼
         ┌────────────────────────────┐                    ┌─────────────────────────────┐
         │ 3A. Coupon/Discount Engine│                    │ 3B. Inventory Allocation    │
         │   (callback + arrow fn)   │                    │   (callback plug-in)        │
         └────────────────────────────┘                    └─────────────────────────────┘
                           │                                                │
                           └──────────────────────┬─────────────────────────┘
                                                  ▼
                                   ┌───────────────────────────────┐
                                   │    4. Payment Processing       │
                                   │    (callback w/ retry logic)   │
                                   └───────────────────────────────┘
                                                  │
                                                  ▼
                         ┌───────────────────────────────┬──────────────────────────────┬──────────────────────────────┐
                         │                               │                              │                              │
                         ▼                               ▼                              ▼
           ┌──────────────────────┐         ┌────────────────────────┐      ┌────────────────────────┐
           │ Audit Event Emitter │         │ Analytics Emitter       │      │ Fraud Event Emitter    │
           └──────────────────────┘         └────────────────────────┘      └────────────────────────┘
                         │                               │                              │
                         └───────────────┬───────────────┴───────────────┬──────────────┘
                                         ▼                              
                           ┌──────────────────────────────────────────┐
                           │        5. Final Invoice Callback          │
                           └──────────────────────────────────────────┘
```

---

# 📌 **9. Output of the Pipeline**

A unified **Final Invoice Object** containing:

* customer profile
* cart items
* discount applied
* selected warehouse
* final amount
* payment receipt
* event statuses

Delivered via the **final callback(orderSummary)**.

---

# ✔ READY FOR TRAINING / PRESENTATION

This case study is **clean**, **industry-level**, and suitable for sessions on:

* JavaScript callbacks
* Arrow function design patterns
* Async pipeline orchestration
* Event-driven architectures

---


