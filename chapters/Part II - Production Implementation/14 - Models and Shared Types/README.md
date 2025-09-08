# Module 14 - Models & Shared Types

## Overview

In the previous modules, we built the foundation of our backend application:

- We understood the request lifecycle.
- We implemented Controllers, Services, and Repositories.
- We designed our database using PostgreSQL and Prisma.

At this point, our application can communicate with the database, but one important question still remains:

> **How should data be represented as it flows through different layers of the application?**

This is where Models and Shared Types become important.

Different layers of the application have different responsibilities, and they should not all use the same object.

For example:

- The database has its own model.
- Incoming HTTP requests have their own structure.
- Services receive Data Transfer Objects (DTOs).
- API responses expose only the required fields.
- Shared types, enums, and constants are reused across multiple modules.

Keeping these representations separate makes the application easier to maintain, extend, and test.

---

# Why This Module Exists

One of the most common mistakes developers make is using the same object throughout the entire application.

For example:

```text
HTTP Request

↓

Controller

↓

Service

↓

Repository

↓

Database
```

Every layer receives and returns exactly the same object.

Although this may work for small projects, it quickly becomes difficult to maintain as the application grows.

Instead, production applications transform data as it moves through the system.

```text
HTTP Request

↓

Request Model

↓

Validation

↓

DTO

↓

Service

↓

Repository

↓

Prisma Model

↓

Database

↓

Prisma Model

↓

Response Model

↓

HTTP Response
```

Each layer works only with the data it actually needs.

---

# What You Will Learn

In this module, we will learn how to design and organize the different types of models used in a backend application.

Topics include:

- Database Models
- Request Models
- Response Models
- Data Transfer Objects (DTOs)
- Shared Types
- Enums
- Constants
- Validation Models
- Folder Organization
- Best Practices
- Common Mistakes

Rather than focusing on Prisma itself, this module focuses on **how data is represented inside the application**.

---

# Module Structure

```text
14 Models & Shared Types

14.1 Introduction

14.2 Why Models Exist

14.3 Database Models (Prisma Models)

14.4 Request Models

14.5 Response Models

14.6 DTOs (Data Transfer Objects)

14.7 Shared Types

14.8 Enums

14.9 Constants

14.10 Validation Models (Zod)

14.11 Folder Structure

14.12 Best Practices

14.13 Common Mistakes

14.14 Models in Our Clinic Backend

14.15 Module Summary
```

---

# Data Representation Example

A single business entity may have multiple representations throughout the application.

## Database Model

```prisma
model Patient {

    id        String @id @default(cuid())

    firstName String

    email     String

}
```

---

## Request Model

```ts
export interface CreatePatientRequest {
  firstName: string

  email: string
}
```

---

## DTO

```ts
export interface CreatePatientDto {
  firstName: string

  email: string
}
```

---

## Response Model

```ts
export interface PatientResponse {
  id: string

  firstName: string
}
```

---

## Shared Type

```ts
export type PatientStatus = 'ACTIVE' | 'INACTIVE'
```

Each representation exists for a different purpose.

---

# Recommended Folder Structure

```text
src/

├── models/
│
├── requests/
│
├── responses/
│
├── dto/
│
├── enums/
│
├── constants/
│
├── shared/
│
├── repositories/
│
├── services/
│
├── controllers/
│
└── database/
```

This separation keeps the application organized and prevents unrelated responsibilities from being mixed together.

---

# Why This Matters

As applications grow, using separate models for different layers provides several benefits:

- Clear separation of responsibilities.
- Better maintainability.
- Easier testing.
- Improved readability.
- Reduced accidental data exposure.
- Stronger type safety.
- Simpler future refactoring.

These practices are commonly followed in production backend applications and help create a scalable architecture.

---

# Learning Outcome

After completing this module, you will understand:

- Why different models exist.
- When each model should be used.
- How data changes shape throughout the request lifecycle.
- How to organize models in a production backend project.
- How Models and Shared Types integrate with Controllers, Services, Repositories, and Prisma.

This module completes the application's data representation layer and prepares us for more advanced backend topics in the upcoming modules.
