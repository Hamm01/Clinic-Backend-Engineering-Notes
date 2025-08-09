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

This section transforms the architecture discussed in Part I into a complete production-ready Clinic Backend.

```
01 - Project Initialization
    ├── 01.1 - Overview
    ├── 01.2 - Prerequisites
    ├── 01.3 - Initializing Node.js
    ├── 01.4 - Installing Dependencies
    ├── 01.5 - TypeScript Configuration
    ├── 01.6 - ESLint & Prettier
    ├── 01.7 - Environment Variables
    ├── 01.8 - Scripts
    ├── 01.9 - Git Initialization
    ├── 01.10 - Project Verification

02 - Creating the Express Application
    ├── 02.1 - Overview
    ├── 02.2 - Creating Express App
    ├── 02.3 - Express Configuration
    ├── 02.4 - Application Bootstrap
    ├── 02.5 - Starting the Server

03 - Environment Configuration

04 - Project Structure

05 - Logging

06 - Error Handling

07 - Database Implementation
    ├── 07.1 - Designing the Database
    ├── 07.2 - Patient Schema
    ├── 07.3 - Clinic Schema
    ├── 07.4 - Doctor Schema
    ├── 07.5 - Appointment Schema
    ├── 07.6 - Medical Record Schema
    ├── 07.7 - Diagnosis Schema
    ├── 07.8 - Prescription Schema
    ├── 07.9 - Relationships
    ├── 07.10 - Indexes
    ├── 07.11 - Soft Deletes
    ├── 07.12 - Prisma Migrations
    ├── 07.13 - Seed Data
    └── 07.14 - Final Database Architecture

08 - Authentication
    ├── 08.1 - Overview
    ├── 08.2 - Password Security
    ├── 08.3 - Password Hashing
    ├── 08.4 - Password Verification
    ├── 08.5 - JWT Authentication
    ├── 08.6 - Access Tokens
    ├── 08.7 - Refresh Tokens
    ├── 08.8 - Login Flow
    ├── 08.9 - Logout Flow
    ├── 08.10 - Authentication Middleware
    ├── 08.11 - Common Authentication Mistakes
    └── 08.12 - Final Authentication Architecture

09 - Authorization
    ├── 09.1 - Overview
    ├── 09.2 - Roles vs Permissions
    ├── 09.3 - Permission Design
    ├── 09.4 - Permission Database Schema
    ├── 09.5 - Seeding Roles & Permissions
    ├── 09.6 - Authorization Middleware
    ├── 09.7 - Route Protection
    ├── 09.8 - Ownership Authorization
    ├── 09.9 - Dynamic Permissions
    ├── 09.10 - Common Authorization Mistakes
    ├── 09.11 - Final Authorization Architecture

10 - Controllers
    ├── 10.1 - Overview
    ├── 10.2 - Anatomy of a Controller
    ├── 10.3 - Request Object
    ├── 10.4 - Response Object
    ├── 10.5 - Building the Doctor Controller
    ├── 10.6 - Building the Patient Controller
    ├── 10.7 - Building the Appointment Controller
    ├── 10.8 - Building the Diagnosis Controller
    ├── 10.9 - API Response Standardization
    ├── 10.10 - Error Handling Inside Controllers
    ├── 10.11 - Thin Controllers vs Fat Controllers
    └── 10.12 - Final Controller Architecture

11 - Services
    ├── 11.1 - Overview
    ├── 11.2 - Anatomy of a Service
    ├── 11.3 - Service Responsibilities
    ├── 11.4 - Building the Doctor Service
    ├── 11.5 - Building the Patient Service
    ├── 11.6 - Building the Appointment Service
    ├── 11.7 - Building the Diagnosis Service
    ├── 11.8 - Transactions
    ├── 11.9 - Business Rules
    ├── 11.10 - Service vs Controller
    ├── 11.11 - Common Mistakes
    └── 11.12 - Final Service Architecture

12 - Repositories
    ├── 12.1 - Overview
    ├── 12.2 - Repository Pattern
    ├── 12.3 - Prisma Client
    ├── 12.4 - Doctor Repository
    ├── 12.5 - Patient Repository
    ├── 12.6 - Appointment Repository
    ├── 12.7 - Diagnosis Repository
    ├── 12.8 - Transactions
    ├── 12.9 - Pagination
    ├── 12.10 - Query Optimization
    ├── 12.11 - Common Mistakes
    └── 12.12 - Final Repository Architecture

13 - Prisma
    ├── 13.1 - Overview
    ├── 13.2 - Prisma Schema
    ├── 13.3 - Prisma Client
    ├── 13.4 - CRUD Operations
    ├── 13.5 - Relations
    ├── 13.6 - Transactions
    ├── 13.7 - Indexes
    ├── 13.8 - Soft Deletes
    ├── 13.9 - Migrations
    ├── 13.10 - Seeding
    ├── 13.11 - Best Practices
    └── 13.12 - Final Prisma Architecture

14 - Models & Shared Types

15 - Validation

16 - Middleware

17 - Routes

18 - Transactions

19 - Caching

20 - Background Jobs

21 - File Uploads

22 - Notifications

23 - Testing

24 - Deployment

25 - Complete Clinic Backend Review
```

---

## Goal

By the end of this repository you should be able to open any production backend project and understand **why each layer exists**, not just what it does.
