# Thinking Like a Backend Engineer

> Building a Production Clinic Backend with Express.js & TypeScript

---

## About

This repository is **not an Express tutorial**.

It is not a Node.js tutorial.

It is not a PostgreSQL tutorial.

Instead, it documents the thought process behind building a production backend application.

Throughout the repository we build a **Clinic Management System** while discussing the architectural decisions that experienced backend engineers make.

The goal is to answer questions like:

- Why do we create Services?
- Why do Controllers stay thin?
- Why use a Repository layer?
- Where should business logic live?
- How do transactions work?
- How do we scale the backend?
- Why should the backend remain stateless?
- How should folders be organized?
- How should features be separated?
- How do senior engineers think before writing code?

This repository intentionally focuses on **engineering decisions**, not framework syntax.

---

# Tech Stack

- Node.js
- Express.js
- TypeScript
- PostgreSQL
- Prisma ORM
- Redis
- JWT Authentication

---

# Running Example

Every chapter builds the same application.

```
Clinic Management System
```

Actors

- Admin
- Doctor
- Receptionist

Domain

- Clinics
- Doctors
- Patients
- Appointments
- Consultations
- Diagnoses
- Prescriptions
- Lab Tests

---

# Learning Philosophy

Every topic answers

```
WHY

↓

HOW

↓

WHERE

↓

CODE

↓

TRADE-OFFS

↓

SUMMARY
```

instead of simply explaining APIs.

---

# Repository Structure

```
01 - Thinking Like a Backend Engineer

02 - Understanding the Business Domain

03 - Designing the Folder Structure

04 - Request Lifecycle

05 - Authentication

06 - Authorization

07 - Controllers

08 - Services

09 - Repositories

10 - Database Design

11 - Transactions

12 - Caching

13 - Background Jobs

14 - Error Handling

15 - Validation

16 - Logging

17 - Scaling

18 - Refactoring

19 - Production Practices

20 - Building the Complete Clinic Backend
```

---

## Goal

By the end of this repository you should be able to open any production backend project and understand **why each layer exists**, not just what it does.
