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

## Part I — Architecture & Design

```
01 - Thinking Like a Backend Engineer

02 - Understanding the Business Domain

03 - Designing the Project Structure

04 - Understanding the Request Lifecycle

05 - Authentication Architecture

06 - Authorization Architecture

07 - Controllers

08 - Services

09 - Repositories

10 - Database Architecture
    ├── 10.01 - Thinking in Entities
    ├── 10.02 - Designing Relationships
    ├── 10.03 - Complete ER Diagram
    ├── 10.04 - Primary Keys & Foreign Keys
    ├── 10.05 - Constraints & Indexes
    ├── 10.06 - Soft Deletes & Audit Fields
    └── 10.07 - Senior Database Design Review

11 - Transactions

12 - Caching

13 - Background Jobs

14 - Validation Strategy

15 - Error Handling Strategy

16 - Logging & Observability

17 - Scalability & Stateless Backend Design

18 - Production Architecture Review
```

---

## Part II — Production Implementation

```
01 - Project Initialization

02 - Creating the Express Application

03 - Environment Configuration

04 - Project Configuration

05 - Logging

06 - Error Handling

07 - Database Connection

08 - Authentication

09 - Authorization

10 - Prisma Schema

11 - Models & Shared Types

12 - Validation

13 - Middleware

14 - Routes

15 - Controllers

16 - Services

17 - Repositories

18 - Transactions

19 - Caching

20 - Background Jobs

21 - File Uploads

22 - Notifications

23 - Testing

24 - Deployment

25 - Building the Complete Clinic Backend
```

---

## Goal

By the end of this repository you should be able to open any production backend project and understand **why each layer exists**, not just what it does.
