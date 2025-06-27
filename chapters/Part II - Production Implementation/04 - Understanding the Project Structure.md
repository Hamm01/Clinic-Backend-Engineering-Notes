# 04 - Understanding the Project Structure

> A folder is not just a place to store files.
>
> Every folder represents a responsibility.

---

# Objective

By now our backend can

- Start successfully
- Read configuration
- Load environment variables

Now we need to organize future code.

This chapter explains

- Why each folder exists
- What belongs inside it
- What should never be placed there
- How folders communicate with one another

The objective is not to memorize the structure.

The objective is to understand **ownership**.

---

# Current Project Structure

```text
src/

├── config/
├── controllers/
├── middleware/
├── models/
├── repositories/
├── routes/
├── services/
├── validators/
├── jobs/
├── types/
├── utils/

├── app.ts
└── server.ts
```

Every folder has exactly one responsibility.

---

# The Philosophy

When a request enters the backend,

every folder answers one question.

```
Who should handle this next?
```

No folder should attempt to solve every problem.

Responsibilities are passed from one layer to another.

---

# Complete Request Journey

```
Client

↓

Route

↓

Middleware

↓

Controller

↓

Service

↓

Repository

↓

Database

↓

Repository

↓

Service

↓

Controller

↓

HTTP Response
```

This journey will remain the same throughout the project.

Only the implementation changes.

---

# config/

## Responsibility

Application configuration.

Examples

- Environment variables
- Database configuration
- Logger configuration
- Cache configuration

---

## Allowed

```text
env.ts

database.ts

logger.ts
```

---

## Never Store

- Business logic
- Database queries
- Authentication logic

Configuration should never make business decisions.

---

# routes/

## Responsibility

Map HTTP requests to controllers.

Example

```http
GET /patients
```

↓

```ts
patientController.listPatients()
```

Routes should only decide

> Which controller handles this request?

Nothing more.

---

## Allowed

```ts
router.get(...)

router.post(...)
```

---

## Never Store

- Validation logic
- Database queries
- Business rules

Routes should remain extremely small.

---

# middleware/

## Responsibility

Run before or after controllers.

Examples

- Authentication
- Authorization
- Logging
- Request ID
- Error handling

Middleware operates on the request,

not the business.

---

## Never Store

Business workflows.

---

# controllers/

## Responsibility

Translate HTTP into business operations.

Controllers

- Read request data
- Call services
- Return responses

Controllers should understand

HTTP,

not business rules.

---

## Good Example

```
Receive request

↓

Call Service

↓

Return JSON
```

---

## Bad Example

```
Receive request

↓

Write SQL

↓

Hash password

↓

Send email

↓

Return response
```

Controllers become impossible to maintain.

---

# services/

## Responsibility

Business logic.

This is the heart of the backend.

Examples

- Schedule appointment
- Cancel appointment
- Register patient
- Prescribe medicine

Services answer

> What should happen?

---

## Allowed

Business decisions

Calculations

Transactions

Multiple repository calls

External APIs

---

## Never Store

Raw SQL

Prisma queries

HTTP responses

---

# repositories/

## Responsibility

Communicate with the database.

Repositories know

- Prisma
- SQL
- MongoDB

No other layer should.

---

## Good Example

```
findPatientById()

createAppointment()

updateDoctor()
```

---

## Never Store

Business rules.

Repositories retrieve data.

Services decide what to do with it.

---

# models/

## Responsibility

Database models.

In our project,

these become

- Prisma schema
- Database types

They describe

how data is stored.

Not

how data behaves.

---

# validators/

## Responsibility

Validate incoming requests.

Examples

```
Email format

Password length

Required fields

UUID format
```

Validation protects the application

before business logic executes.

---

# utils/

## Responsibility

Reusable helper functions.

Examples

- Date formatting
- String utilities
- Hash helpers
- UUID helpers

---

## Never Store

Business workflows.

If a function contains clinic-specific rules,

it belongs in a Service,

not Utils.

---

# jobs/

## Responsibility

Background processing.

Examples

- Send email
- Generate reports
- Daily cleanup
- Notifications

Jobs run outside the normal request lifecycle.

---

# types/

## Responsibility

Shared TypeScript types.

Examples

- DTOs
- Interfaces
- Enums
- Shared request types

Avoid duplicating types across modules.

---

# Communication Rules

Not every folder may communicate with every other folder.

Allowed

```text
Routes

↓

Controllers

↓

Services

↓

Repositories

↓

Database
```

Avoid

```text
Route

↓

Repository
```

or

```text
Controller

↓

Prisma
```

These shortcuts make large projects difficult to maintain.

---

# Responsibility Matrix

| Folder       | Knows HTTP | Knows Business | Knows Database |
| ------------ | :--------: | :------------: | :------------: |
| Routes       |     ✅     |       ❌       |       ❌       |
| Middleware   |     ✅     |       ❌       |       ❌       |
| Controllers  |     ✅     |    Limited     |       ❌       |
| Services     |     ❌     |       ✅       |       ❌       |
| Repositories |     ❌     |       ❌       |       ✅       |
| Validators   |     ✅     |       ❌       |       ❌       |
| Config       |     ❌     |       ❌       |       ❌       |
| Jobs         |     ❌     |    Depends     |    Usually     |

This table summarizes the ownership of each layer.

---

# Common Mistakes

## Business Logic in Controllers

Wrong.

Controllers should remain thin.

---

## Database Queries in Services

Avoid coupling business logic to Prisma.

Use repositories.

---

## HTTP Objects in Services

Services should not receive

`req`

or

`res`.

Pass only the data required.

---

## Utility Functions Becoming a Dumping Ground

If everything goes into `utils/`,

nothing has a clear home.

Keep utilities generic.

---

# Senior Engineer Review

Before approving the project structure, verify

- Does every folder have one responsibility?
- Can business logic be located quickly?
- Is database access isolated?
- Are HTTP concerns separated from business concerns?
- Can another developer understand where new code belongs?
- Can the project grow without reorganizing folders?

If these questions have clear answers,

the structure is ready for implementation.

---

# Project Status

```text
Backend Foundation

✅ Project Setup

✅ Express Application

✅ Environment Configuration

✅ Project Structure

⬜ Database Connection

⬜ Logging

⬜ Error Handling

⬜ Authentication

⬜ Authorization

⬜ Business Modules
```

---

# Summary

The project structure is more than a collection of folders.

It defines ownership.

Each layer has one responsibility, communicates with a limited number of other layers, and remains independent of unrelated concerns.

A well-designed structure makes the application easier to understand, easier to test, and easier to extend as new features are added.

---

# Next Chapter

**05 - Logging**

Before connecting to the database or implementing business features, we will introduce structured logging so that every request and every error can be observed consistently throughout the application.
