# Comic-Con Parking System — Design Approach (ERD Explanation)

## 1. How I approached the problem

Instead of starting with tables, I tried to visualize what actually happens in a parking system during a big event like Comic-Con.

A vehicle doesn’t just “exist” — it goes through a flow:

* vehicle arrives at the venue
* system identifies vehicle type
* a suitable parking spot is assigned
* a ticket is generated
* vehicle may stay or even move to another spot
* vehicle exits
* system calculates fee
* payment is made

So I approached this as a **time-based resource allocation problem**, not just a CRUD system.

The key idea was:

> Don’t model parking data. Model how parking actually works over time.

---

## 2. Identifying the core entities

From the flow, I extracted the main concepts:

### Vehicles

Represents the physical vehicle entering the system.

Important decision:

* A vehicle can visit multiple times → so it should NOT store entry/exit info

---

### Vehicle Categories

Used to classify vehicles (bike, car, SUV, EV).

Reason for separation:

* pricing depends on category
* compatibility with parking spots

---

### Parking Spots

Represents actual physical parking slots.

Each spot belongs to:

* a zone/level
* a category (VIP, EV, staff, etc.)

---

### Zones

Grouping of parking spots.

Reason:

* helps track availability per zone/level
* improves querying and organization

---

### Spot Categories

Represents reserved or specialized spots:

* VIP
* staff
* EV charging
* general

Important:

* this is attached to the spot, not the vehicle

---

## 3. Parking Session (core decision)

This is the most important entity in the system.

Instead of storing entry/exit in vehicle, I introduced:

### Parking Sessions

Represents one visit of a vehicle.

Contains:

* entry time
* exit time

Why:

* one vehicle can enter multiple times
* keeps history clean and normalized

---

## 4. Spot Allocation (critical pivot)

Initially, it feels like:

* one session → one spot

But that breaks if:

* vehicle is reassigned
* spot changes due to constraints

So I introduced:

### Spot Allocations

This tracks:

* which spot was used
* during what time

This converts the system into:

> session ↔ spot over time

This is the most important scalability decision.

---

## 5. Ticket vs Session

I separated:

### Ticket

* external identifier (what user sees)

### Session

* internal system record

Reason:

* ticket can be printed, scanned, reissued
* session stores actual logic

---

## 6. Pricing Strategy

Pricing depends on:

* vehicle type
* spot category
* duration

So I avoided storing price directly in session.

Instead, I created:

### Pricing Rules

This defines:

* vehicleCategory + spotCategory → price per hour

Also added:

* `validFrom`, `validTo`

Reason:

* pricing can change during the event
* old sessions should not be affected

---

## 7. Payments

Payments are linked to session.

Reason:

* billing is based on total parking usage
* session is the source of truth

Also allows:

* multiple payments (partial, retries)

---

## 8. Availability (important decision)

I intentionally did NOT store:

* `isAvailable` in parking spots

Reason:

> Availability is not data. It is derived from active allocations.

A spot is occupied if:

* there exists an allocation with no `endTime`

This avoids:

* race conditions
* inconsistent data

---

## 9. Compatibility Layer

Not all vehicles can use all spots.

So I introduced:

### spotCategoryVehicleCategories

This defines:

* which vehicle types can park in which spot categories

This prevents invalid assignments like:

* SUV in bike slot

---

## 10. Where I used (and avoided) pivot tables

### Used:

* `spotAllocations` → session ↔ spot over time
* `spotCategoryVehicleCategories` → compatibility mapping

### Avoided:

* unnecessary pivots where relationships are simple (1:N)

Reason:

> Pivot tables are for real many-to-many relationships, not for overengineering.

---

## 11. Scalability considerations

This design supports:

* multiple entries of same vehicle
* reuse of parking spots
* dynamic pricing changes
* spot reassignment during session
* tracking of full parking history

Also:

* no derived fields stored
* everything is based on source-of-truth entities

---

## 12. What makes this different from basic designs

Common beginner mistakes:

* storing `isAvailable`
* linking session directly to one spot
* storing price in session
* not separating ticket and session

In this design:

* time-based allocation is modeled properly
* pricing is abstracted
* compatibility is enforced
* history is preserved

---

## 13. Trade-offs

* more tables than a simple system
* requires joins for queries
* pricing calculation logic is external

But:

* this avoids future redesign
* keeps system correct under load

---

## 14. Final thought

The goal was not to make the system complex.

The goal was to make it:

* correct
* flexible
* aligned with real-world behavior

If the flow is modeled correctly, scaling and extending the system becomes much easier later.
