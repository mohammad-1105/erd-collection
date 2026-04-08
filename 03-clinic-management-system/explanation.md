# Clinic Management System — Design Approach (ERD Explanation)

## 1. How I approached the problem

Before writing any tables, I tried to understand how a real clinic actually works.

A patient doesn’t just “exist” in the system — they go through a flow:

* they may book an appointment
* they visit a doctor
* doctor consults them
* doctor may prescribe tests
* tests are done and reports come later
* payment happens somewhere in between

So instead of thinking in terms of tables, I started thinking in terms of **events and flow**.

The key idea was:

> Don’t model data. Model what actually happens in real life.

---

## 2. Identifying the core entities

From the flow, I extracted the main concepts:

### Patients

Basic entity. A patient can visit multiple times.

### Doctors

Doctors are independent entities. They can handle many patients.

### Specialties

I kept this separate instead of a column because:

* multiple doctors can share the same specialty
* a doctor can have multiple specialties

This avoids duplication and keeps it flexible.

---

## 3. Appointment vs Consultation (important decision)

This was the most important part.

At first glance, it feels like appointment = visit, but that’s wrong.

* Appointment = scheduled intent
* Consultation = actual visit

Not every appointment becomes a consultation:

* patient cancels
* patient doesn’t show up

Also, some consultations happen without appointments (walk-ins).

So I designed:

* `appointments` as optional
* `consultations` as the main entity

And made:

* `consultation.appointmentId` nullable

This gives flexibility without breaking structure.

---

## 4. Why consultation is the central entity

Everything important happens during consultation:

* doctor meets patient
* diagnosis happens
* tests are prescribed

So instead of attaching things to patient or appointment, I attached them to consultation.

This keeps the model aligned with real-world logic.

---

## 5. Handling diagnostic tests

I avoided directly storing test names inside consultation.

Instead, I split it into two parts:

### Test Catalog (`tests`)

* defines what tests exist
* reusable

### Prescribed Tests (`prescribedTests`)

* actual instance of a test for a consultation
* stores status (prescribed, completed)

This separation is important because:

* same test can be used many times
* each prescription has its own lifecycle

This is a proper many-to-many resolution.

---

## 6. Reports design

Reports are generated after tests, not during consultation.

So:

* report belongs to `prescribedTest`
* not directly to patient or doctor

This ensures:

* traceability
* clear linkage to what was actually tested

---

## 7. Payment modeling

Payments are tricky because they can happen for:

* consultation
* tests

Instead of overcomplicating, I allowed:

* `consultationId`
* `prescribedTestId`

This keeps it simple and flexible for now.

A more advanced system could introduce billing/invoice tables, but that would be overengineering for this scope.

---

## 8. Where I used (and avoided) pivot tables

I didn’t add pivot tables everywhere.

I only used them where they are actually needed:

### Used:

* `doctorSpecialties` → many-to-many
* `prescribedTests` → many-to-many with state

### Avoided:

* unnecessary pivots for patient-doctor or appointment

Reason:

> Pivot tables are for relationships, not for “scalability hype”.

---

## 9. Scalability considerations

This design is scalable because:

* supports walk-ins and scheduled visits
* allows multiple tests per consultation
* allows reports to be generated later
* avoids duplication using catalog tables
* keeps relationships normalized

Also, most entities are independent and loosely coupled.

---

## 10. What makes this different from basic designs

Typical beginner mistakes:

* merging appointment and consultation
* storing test names directly
* attaching everything to patient
* no separation of lifecycle

In this design:

* consultation is treated as the core event
* reusable vs instance data is separated
* optional relationships are handled properly

---

## 11. Trade-offs

* Slightly more tables than a naive design
* Requires joins to fetch full data
* Payment model is simplified (not full billing system)

But this is acceptable for a clinic-level system.

---

## 12. Final thought

The goal was not to make the system complex.
The goal was to make it **correct and adaptable**.

If the real-world flow is modeled correctly, scaling the system later becomes much easier.
