# Fitness Influencer Coaching Platform — Database Design Explanation

## 1. Problem Context

This system models an **online coaching ecosystem**, not a traditional gym or simple ecommerce platform.

Key characteristics:

* Trainers manage multiple clients
* Clients can subscribe to multiple plans over time
* Coaching includes sessions, progress tracking, and periodic check-ins
* Not all users follow the same journey (consultation-only vs full coaching)

Because of this, the design must support:

* flexible relationships
* time-based data
* multiple interaction types

---

## 2. Core Design Philosophy

The design follows **separation of concerns** and **real-world modeling**.

We divide the system into distinct domains:

| Domain        | Responsibility                 |
| ------------- | ------------------------------ |
| Users         | Identity (trainer or client)   |
| Plans         | Reusable coaching programs     |
| Subscriptions | Client-specific plan lifecycle |
| Sessions      | Scheduled interactions         |
| Progress      | Structured measurable data     |
| Check-ins     | Subjective reports             |
| Payments      | Financial transactions         |

This prevents tight coupling and ensures scalability.

---

## 3. Key Design Decisions

### 3.1 Users + Profiles Separation

* `users` → core identity
* `trainerProfiles` / `clientProfiles` → role-specific data

**Why:**

* avoids null-heavy tables
* supports role evolution
* keeps authentication separate from domain data

---

### 3.2 Plans vs Subscriptions

We separate:

* **Plans** → reusable templates created by trainers
* **Subscriptions** → client-specific enrollment

**Why this matters:**

* one plan → many clients
* one client → many plans over time

Direct linking (user → plan) would break lifecycle tracking.

---

### 3.3 Subscription as a First-Class Entity

`subscriptions` stores:

* start date
* end date
* status

**Why:**

* models real business flow
* supports renewals, cancellations
* enables historical tracking

---

### 3.4 Sessions with Pivot Table

Instead of:

* directly linking trainer ↔ client

We use:

* `sessions` (event)
* `sessionParticipants` (pivot)

**Why:**

* supports group sessions
* supports multiple trainers
* tracks attendance

This converts a rigid 1:N model into a flexible many-to-many system.

---

### 3.5 Progress vs Check-ins Separation

We explicitly separate:

| Type          | Nature               |
| ------------- | -------------------- |
| Progress Logs | structured, numeric  |
| Check-ins     | subjective, periodic |

**Why:**

* clean queries (analytics vs reports)
* avoids mixed data types
* improves data integrity

---

### 3.6 Plan Content Structure

We introduce:

* `planModules`
* `planItems`

**Why:**

* supports structured programs (workout/diet)
* enables multi-day routines
* future-proof for media/content expansion

---

### 3.7 Payment Linked to Subscription

Payments are tied to subscriptions instead of users.

**Why:**

* aligns with business logic (pay for a plan)
* supports retries, failures, multiple payments
* avoids ambiguity

---

### 3.8 Optional Trainer-Client Mapping

`trainerClients` provides a direct relationship layer.

**Why:**

* faster queries (no joins across subscriptions/sessions)
* supports non-paying clients (consultation only)

---

## 4. Relationships Overview

The system includes:

* **1:1** → user ↔ profile
* **1:N** → trainer → plans, client → subscriptions
* **M:N (resolved)**:

  * client ↔ plan via subscriptions
  * users ↔ sessions via sessionParticipants

This ensures relational integrity without redundancy.

---

## 5. Benefits of This Design

### ✅ 1. Scalability

* supports growing number of users and plans
* handles multiple interaction types

---

### ✅ 2. Flexibility

* clients can:

  * subscribe to multiple plans
  * attend sessions optionally
* trainers can manage multiple clients

---

### ✅ 3. Clean Separation

* avoids “God tables”
* each entity has a single responsibility

---

### ✅ 4. Extensibility

Future features can be added without schema redesign:

* chat systems
* notifications
* AI recommendations
* media-heavy plans

---

### ✅ 5. Data Integrity

* structured vs unstructured data separated
* avoids duplication and inconsistency

---

## 6. Trade-offs / Costs

### ⚠️ 1. Increased Complexity

* more tables than a basic design
* requires proper joins

---

### ⚠️ 2. Query Overhead

* retrieving full data (e.g., client dashboard) requires multiple joins

---

### ⚠️ 3. Overhead for Small Systems

* may feel heavy for very small-scale usage

---

### ⚠️ 4. Requires Discipline

* developers must respect relationships
* improper queries can hurt performance

---

## 7. Alternatives Considered

### ❌ Direct user → plan mapping

Rejected because:

* no lifecycle tracking
* no history

---

### ❌ Single “activity” table for progress + check-ins

Rejected because:

* mixes structured and unstructured data
* complicates queries

---

### ❌ Sessions without pivot table

Rejected because:

* limits scalability
* cannot support group interactions

---

## 8. When This Design Works Best

This design is ideal when:

* the platform is growing
* multiple trainers and clients exist
* structured coaching is required

---

## 9. Final Thought

This database is designed to model **real-world coaching behavior**, not just store data.

The key idea behind this system is:

> Separate reusable concepts (plans) from user-specific state (subscriptions),
> and separate structured data (progress) from human interaction (check-ins).

This ensures the system remains:

* maintainable
* scalable
* aligned with business logic

---
