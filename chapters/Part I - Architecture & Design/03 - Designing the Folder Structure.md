# 03 - Designing the Folder Structure

> A good folder structure reflects the business, not the framework.

---

# Objective

In the previous chapters, we understood the business and identified the domain.

Now we need to answer an important question:

> **How should we organize the code so that it remains maintainable as the project grows?**

The purpose of a folder structure is **not** to make the project look clean.

Its purpose is to:

- Separate responsibilities.
- Reduce coupling.
- Improve maintainability.
- Make features easy to locate.
- Allow multiple developers to work independently.
- Keep business logic isolated from infrastructure.

---

# The Wrong Way to Think

Many developers begin with:

```
Express

↓

Routes

↓

Controllers

↓

Done
```

This works for small CRUD projects.

As the project grows:

- controllers become 500+ lines
- SQL appears inside controllers
- validation is duplicated
- business rules are scattered
- testing becomes difficult

The project slowly becomes harder to change.

---

# The Right Way to Think

Always organize around the business.

Instead of asking:

> Where should I put this Express file?

Ask:

> Which part of the business owns this responsibility?

---

# Recommended Project Structure

```
src/

    app.ts
    server.ts

    config/

    modules/

        auth/

        clinic/

        doctor/

        patient/

        appointment/

        consultation/

    shared/

    infrastructure/

    database/

    cache/

    jobs/

    common/
```

Notice something.

The first folders represent **business modules**.

Not Express.

Not Prisma.

Not PostgreSQL.

---

# Why Modules?

Suppose someone asks:

> "Where is the appointment logic?"

It should be obvious.

```
modules/

    appointment/
```

Everything related to appointments should live there.

---

# Module Structure

Each business module follows the same structure.

Example:

```
modules/

    patient/

        patient.routes.ts

        patient.controller.ts

        patient.service.ts

        patient.repository.ts

        patient.validator.ts

        patient.types.ts

        patient.mapper.ts

        patient.constants.ts
```

Every file has one responsibility.

---

# Why This Structure?

Instead of searching across the entire project:

```
controllers/

repositories/

validators/

routes/
```

everything related to a feature stays together.

Example:

Patient feature.

Everything is inside

```
patient/
```

This is called **feature-based organization**.

---

# Responsibility of Each Layer

---

## Routes

Purpose

Connect an HTTP endpoint to a controller.

Example

```
GET /patients/:id
```

Routes should contain:

- URL
- Middleware
- Controller mapping

Routes should NOT contain:

- SQL
- Business logic
- Validation logic
- Calculations

Think of routes as traffic controllers.

---

## Controllers

Purpose

Convert an HTTP request into a business request.

Controller responsibilities:

- Read request
- Read params
- Read body
- Read authenticated user
- Call service
- Return response

Controllers should remain very small.

Good controller:

```ts
async getPatient(req, res) {
    const patient = await patientService.getById(
        req.params.id,
        req.user.id
    );

    return res.json(patient);
}
```

Controllers should not know how data is stored.

---

## Services

This is the heart of the application.

Business logic belongs here.

Example

```
Doctor opens patient history.
```

Service decides

- Can doctor access patient?
- Is doctor active?
- Does patient belong to doctor's clinic?
- Should sensitive fields be hidden?

The service coordinates the business.

---

## Repository

Repository talks to the database.

Nothing else.

Example

Instead of writing:

```ts
prisma.patient.findUnique(...)
```

inside every service,

the service calls

```ts
patientRepository.findById(id)
```

Repository hides database implementation.

---

## Validators

Purpose

Validate incoming data.

Example

Patient registration.

Rules:

- name required
- phone required
- age positive

Validation belongs here.

Not inside controller.

---

## Mapper

Sometimes database objects and API responses differ.

Example

Database

```
first_name
last_name
```

API

```
fullName
```

Mapper converts between formats.

---

## Types

Contains

- interfaces
- request types
- response types
- DTOs

Keeps types close to the business module.

---

## Constants

Stores values used by the module.

Example

```
AppointmentStatus

Scheduled

Cancelled

Completed
```

Avoid hardcoded strings.

---

# Shared Folder

Contains reusable code.

Examples

```
shared/

    logger/

    errors/

    utils/

    pagination/

    response/
```

If something belongs to multiple modules,

place it here.

---

# Infrastructure Folder

Contains integrations with external systems.

Examples

```
Redis

Cloudinary

AWS

Email

SMS

Payment Gateway
```

Business logic should never directly depend on these services.

---

# Database Folder

Contains database configuration.

Example

```
database/

    prisma/

    migrations/

    seed/

    client.ts
```

Notice something.

Business modules never own the database.

The database is shared infrastructure.

---

# Cache Folder

Contains cache implementation.

Example

```
Redis Client

Cache Keys

TTL Configuration
```

Business services decide **when** to cache.

The cache folder decides **how** to cache.

---

# Jobs Folder

Background processing.

Examples

- Reminder emails
- SMS notifications
- Daily reports
- Cleanup jobs

Jobs execute outside the HTTP request.

---

# Common Folder

Contains application-wide code.

Example

```
Enums

Exceptions

Middlewares

Base Classes
```

Use carefully.

Avoid turning it into a miscellaneous folder.

---

# Why Keep Business Logic Inside Services?

Consider this requirement:

> A doctor can only access patients from their own clinic.

Bad

Controller

```ts
if (doctor.clinicId !== patient.clinicId) {
  throw Error()
}
```

Now GraphQL needs the same rule.

Duplicate.

Tomorrow

Background Job.

Duplicate.

Instead

```
DoctorService
```

owns this rule.

Every caller benefits.

---

# Why Keep Database Code Inside Repositories?

Suppose today we use Prisma.

Tomorrow

Company changes to MongoDB.

Only repository changes.

Service remains identical.

Business rules remain identical.

This is separation of concerns.

---

# When Repository Pattern May Be Unnecessary

Repository pattern is useful when:

- medium projects
- large projects
- multiple developers
- long-term maintenance

It may be unnecessary when:

- learning project
- coding interview
- prototype
- simple CRUD API
- one-week internal tool

Avoid adding abstractions without a reason.

---

# When Service Layer May Be Thin

Some endpoints are pure CRUD.

Example

```
GET Clinic By ID
```

No permissions.

No calculations.

No business rules.

A thin service is acceptable.

Do not invent business logic where none exists.

---

# Database Schema Comes After This

Notice that until now we have never discussed:

- PostgreSQL
- MongoDB
- Prisma Schema

This is intentional.

Order of design:

```
Business

↓

Modules

↓

Responsibilities

↓

Folder Structure

↓

Business Rules

↓

Database Schema

↓

API Implementation
```

The database should represent the business.

The business should never be designed around the database.

---

# Next Chapter

The next chapter follows the complete lifecycle of an incoming request.

Instead of immediately designing the database, we first need to understand how a backend application processes a request from start to finish.

Using the Clinic Management System, we will trace a single request—from the moment it reaches the server until the response is returned to the client.

Topics covered:

- How a request enters the application
- Route matching
- Middleware execution
- Authentication
- Authorization
- Request validation
- Controller responsibilities
- Service execution
- Repository interaction
- Response generation
- Error handling
- Where business logic is executed
- Where database access actually happens

By the end of the chapter, you should clearly understand the responsibility of every layer involved in processing a request and why each layer exists.

## The database schema will be designed only after the complete request lifecycle is understood, ensuring that the persistence layer is driven by business requirements rather than implementation details.

# Summary

A good backend structure separates responsibilities instead of files.

- Routes receive requests.
- Controllers translate requests.
- Services implement business rules.
- Repositories access data.
- Validators protect input.
- Infrastructure integrates external services.

When each layer has a single responsibility, the project becomes easier to understand, test, and maintain as it grows.
