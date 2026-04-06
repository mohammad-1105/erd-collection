# Instagram Thrift Creator Store — Database Design Explanation

## 1. Problem Framing

This system is not a typical ecommerce store.

It has two fundamentally different product behaviors:

* **Thrift items** → unique, single-piece inventory
* **Handmade items** → reproducible, multi-quantity inventory

If modeled incorrectly, this leads to:

* overselling unique items
* inconsistent stock tracking
* rigid schema that breaks when scaling

The design focuses on **separating abstraction from physical inventory**.

---

## 2. Core Design Principle

We divide the system into four conceptual layers:

| Layer       | Responsibility             |
| ----------- | -------------------------- |
| Product     | What is being sold         |
| Inventory   | How many exist physically  |
| Order       | Who is buying what         |
| Fulfillment | Payment and delivery state |

This separation prevents tight coupling and keeps the system extensible.

---

## 3. Entity Design Rationale

### 3.1 Customers

Stores identity across platforms (Instagram, WhatsApp).

Why:

* This business does not rely on email-based authentication
* Orders originate from social platforms

---

### 3.2 Products (Abstract Layer)

Represents the idea of a product.

Key decision:

* `productType = THRIFT | HANDMADE`

Why:

* Business logic differs depending on type
* Enables branching behavior without duplicating tables

---

### 3.3 Product Variants

Used for attributes like:

* size
* color
* material

Why not store directly in Product?

* Avoid schema explosion (adding columns repeatedly)
* Supports combinations (e.g., Size M + Red)

---

### 3.4 Inventory Items (Critical Layer)

This is the most important table in the system.

It represents **real, sellable stock**.

Key fields:

* `quantity`
* `isUnique`

Behavior:

| Case          | Representation                 |
| ------------- | ------------------------------ |
| Thrift item   | quantity = 1, isUnique = true  |
| Handmade item | quantity > 1, isUnique = false |

Why this design:

* Unifies both product types in one system
* Avoids separate tables for thrift vs handmade
* Keeps stock logic centralized

---

### 3.5 Orders

Represents a customer's purchase intent.

Important:

* Does NOT store product details directly
* Only references the customer and total state

---

### 3.6 Order Items (Junction Table)

Handles the many-to-many relationship:

* One order → multiple items
* One inventory item → can appear in many orders (over time)

Key decision:

* References `inventoryId`, not `productId`

Why:

* Orders must reflect **actual stock**, not abstract products
* Prevents selling the same thrift item twice

Also stores:

* `priceAtPurchase`

Why:

* Product price can change later
* Orders must preserve historical pricing

---

### 3.7 Payments

Separated from orders.

Why:

* Payment lifecycle differs from order lifecycle
* Supports retries, failures, partial payments in future

---

### 3.8 Shipments

Also separated.

Why:

* Shipping has independent states:

  * pending
  * shipped
  * delivered

* Prevents bloating the Order table

* Enables tracking and logistics extensions

---

## 4. Relationship Design

### Key Relationships

* Customer → Orders (1:N)
* Order → OrderItems (1:N)
* Product → InventoryItems (1:N)
* InventoryItem → OrderItems (1:N)
* Order → Payment (1:1)
* Order → Shipment (1:1)

Why this structure:

* Matches real-world flow of commerce
* Keeps each concern isolated

---

## 5. Why Not Simpler Designs?

### ❌ Putting quantity in Product

Breaks:

* cannot track unique thrift items
* cannot handle variants properly

---

### ❌ Skipping Inventory Table

Breaks:

* stock tracking
* overselling prevention
* future scalability

---

### ❌ Linking OrderItem → Product

Breaks:

* no guarantee of stock correctness
* no way to handle unique items safely

---

## 6. Edge Cases Handled

This design correctly supports:

* Selling one-of-one thrift items
* Selling multiple handmade items
* Variant-based products (size/color)
* Price changes over time
* Payment failures
* Delayed shipping
* Customers placing multiple orders

---

## 7. Scalability Considerations

This model allows future extensions without refactoring:

* Discounts / Coupons
* Returns / Refunds
* Product media (images/videos)
* Order tracking history
* Multi-vendor support

---

## 8. Mental Model Summary

* **Product** → concept
* **InventoryItem** → actual stock
* **Order** → intent
* **OrderItem** → snapshot of purchase
