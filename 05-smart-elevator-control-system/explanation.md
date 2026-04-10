# Smart Elevator Control System — Design Approach (ERD Explanation)

## 1. How I approached the problem

Instead of jumping into tables, I tried to understand how a real elevator system works in large buildings.

This is not just about elevators — it is about handling **requests, allocation, and movement over time**.

A typical flow looks like:

* user presses a button on a floor
* system creates a request
* system decides which elevator should handle it
* elevator moves and completes the ride
* system logs the ride
* elevator status keeps changing continuously
* maintenance may interrupt normal operation

So I approached this as a **real-time system with events and state transitions**, not just static data.

The key idea was:

> Don’t model elevators. Model how elevator systems behave.

---

## 2. Identifying the core entities

From the flow, I extracted the main components.

### Buildings

Top-level container. Everything belongs to a building.

---

### Floors

Each building has multiple floors.

Reason:

* requests originate from floors
* elevators move between floors

---

### Shafts

Optional but realistic.

Reason:

* each shaft contains one elevator
* helps represent physical structure

---

### Elevators

Represents the actual elevator unit.

Important decision:

* do NOT store dynamic state here (like current ride or request)

---

## 3. Elevator ↔ Floor relationship

An important modeling decision.

* one elevator serves multiple floors
* one floor can be served by multiple elevators

So this becomes a **many-to-many relationship**.

### ElevatorFloors (pivot)

Used to map which elevator serves which floors.

---

## 4. Requests vs Rides (critical separation)

This is one of the most important decisions.

### FloorRequest

Represents user intent:

* button press
* direction (up/down)

### RideLog

Represents actual execution:

* elevator movement
* start and end time

Why separate?

* not every request is immediately served
* system may queue or delay requests

---

## 5. Ride Assignment

Between request and ride, there is a decision layer.

### RideAssignment

Represents:

* which elevator was chosen for a request

Why this exists:

* decouples request creation from system decision
* allows optimization logic later

---

## 6. Status tracking (important design choice)

Elevator status changes frequently:

* idle
* moving
* maintenance

Instead of storing:

* elevator.status

I created:

### ElevatorStatusLogs

Reason:

> Status is time-based, not static.

This allows:

* history tracking
* performance analysis
* debugging issues later

---

## 7. Maintenance tracking

Maintenance is not just a flag.

It has:

* start time
* end time
* description

So I created:

### MaintenanceLogs

Reason:

* keeps full history
* does not overwrite previous records

---

## 8. Why I avoided certain designs

### ❌ Storing dynamic data in elevators

Examples avoided:

* current request
* last ride
* status column

Reason:

* leads to inconsistency
* breaks historical tracking

---

### ❌ Merging request and ride

This would:

* mix intent with execution
* make system harder to reason about

---

## 9. Scalability considerations

This design supports:

* multiple buildings
* multiple elevators per building
* many-to-many floor coverage
* high volume request handling
* detailed ride analytics
* maintenance history tracking

Also:

* entities are loosely coupled
* dynamic data is separated from configuration

---

## 10. What makes this different from basic designs

Common beginner mistakes:

* storing everything in elevator table
* not separating request and ride
* ignoring time-based state
* not modeling many-to-many relationships

In this design:

* request → assignment → ride flow is clear
* time-based logs are preserved
* relationships reflect real-world behavior

---

## 11. Trade-offs

* more tables compared to simple design
* requires joins for queries
* slightly more complex to implement

But:

* system becomes reliable
* easy to extend later

---

## 12. Final thought

The goal was not to simplify the system artificially.

The goal was to make it:

* accurate
* scalable
* aligned with real-world elevator operations

Once the behavior is modeled correctly, adding features like optimization, analytics, or AI-based routing becomes much easier.
