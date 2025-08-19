# Repositories

> The Repository layer is responsible for communicating with the database. It provides a clean interface for retrieving and persisting data while hiding the implementation details of the underlying database technology. In a production backend, Services should never worry about how data is stored—they should only focus on business logic. Repositories make that separation possible.

---

# Module Information

**Module**

Repositories

**Purpose**

Persistence Layer

**Current Folder**

```
12 - Repositories
```

**Technology**

- Prisma ORM
- PostgreSQL
- TypeScript

**Business Example**

Clinic Management System

---

# Why This Module Exists

Until now, our Services have coordinated business workflows.

For example,

```
Create Appointment

↓

Validate Doctor

↓

Validate Patient

↓

Save Appointment
```

The Service decides **what** should happen.

However,

the Service should not decide **how** data is stored.

That responsibility belongs to the Repository layer.

---

# What Is a Repository?

A Repository is a class responsible for communicating with the database.

Instead of writing database queries inside Services,

Services delegate persistence to Repositories.

Example

```
AppointmentService

↓

AppointmentRepository

↓

Prisma

↓

PostgreSQL
```

The Service asks for data.

The Repository knows how to retrieve it.

---

# Why Not Use Prisma Directly?

Consider the following Service.

```ts
async create(dto: CreatePatientDto) {

    const patient = await prisma.patient.create({
        data: dto
    });

    return patient;

}
```

This works.

But now the Service has two responsibilities:

- Business logic.
- Database access.

As the application grows, these responsibilities become tightly coupled.

---

# Using a Repository

Instead,

the Service delegates persistence.

```ts
async create(dto: CreatePatientDto) {

    return this.patientRepository.create(dto);

}
```

Now the Service focuses on the business workflow,

while the Repository focuses on data access.

Each layer has a single responsibility.

---

# Repository Layer in Our Architecture

```
Client

↓

Routes

↓

Middleware

↓

Controller

↓

Service

↓

Repository

↓

Prisma ORM

↓

PostgreSQL
```

Repositories sit between the business layer and the database.

They isolate the rest of the application from persistence details.

---

# Responsibilities of a Repository

Repositories are responsible for:

- Reading data.
- Writing data.
- Updating records.
- Deleting records.
- Executing database queries.
- Mapping database operations.

Repositories are **not** responsible for:

- Business rules.
- HTTP requests.
- Validation logic.
- Authentication.
- Authorization.
- Transactions involving multiple business workflows.

Those responsibilities belong to other layers.

---

# How This Module Builds

This module progresses from simple database operations to production-ready repository design.

```
Repository Overview

↓

Why Repositories Exist

↓

Repository Responsibilities

↓

Building Patient Repository

↓

Building Doctor Repository

↓

Building Appointment Repository

↓

Query Organization

↓

Reusable Repository Methods

↓

Repository Anti-Patterns

↓

Repository Architecture Review
```

Each chapter introduces one new concept while building the Clinic Backend.

---

# Business Example

When a receptionist creates a patient,

the request flows like this.

```
Receptionist

↓

PatientController

↓

PatientService

↓

PatientRepository

↓

Prisma

↓

PostgreSQL
```

Notice that the Service never communicates directly with Prisma.

The Repository becomes the single gateway to the database.

---

# What You Will Learn

By the end of this module, you will be able to:

- Design production-ready Repositories.
- Keep Services independent of database technology.
- Organize complex database queries.
- Reuse persistence logic across the application.
- Understand where database code belongs.
- Avoid common Repository design mistakes.

---

# Learning Philosophy

Like every module in this handbook,

we will not learn Repositories by memorizing methods.

Instead, every chapter answers:

```
WHY

↓

HOW

↓

WHERE

↓

CODE

↓

PRODUCTION NOTES

↓

COMMON MISTAKES

↓

SUMMARY
```

The goal is not only to write database queries,

but to understand why production backends isolate persistence behind a Repository layer.

---

# Prerequisites

Before starting this module, you should already understand:

- Controllers
- Services
- Prisma Schema
- Database Relationships
- Business Workflows
- Transactions

Repositories build directly on top of those concepts.

---

# Files in This Module

```
README.md

12.1 - Repository Overview.md

12.2 - Why Repositories Exist.md

12.3 - Repository Responsibilities.md

12.4 - Building the Patient Repository.md

12.5 - Building the Doctor Repository.md

12.6 - Building the Appointment Repository.md

12.7 - Organizing Database Queries.md

12.8 - Reusable Repository Methods.md

12.9 - Repository vs Prisma.md

12.10 - Repository Anti-Patterns.md

12.11 - Repository Architecture Review.md
```

---

# Summary

The Repository layer forms the boundary between business logic and data persistence.

Throughout this module, we will learn how production applications organize database access, keep Services focused on business workflows, and design Repositories that remain simple, reusable, and maintainable as applications grow.

---

# Next Chapter

**12.1 - Repository Overview**

In the next chapter, we will begin by understanding what a Repository is, where it fits in the backend architecture, and why almost every production backend introduces this layer between Services and the database.
