# 10 - Repositories

> A Repository is responsible for one thing:
>
> **Persisting and retrieving data.**
>
> It should know everything about the database and nothing about the business.

---

# Objective

Every backend eventually needs to store and retrieve data.

The Service Layer understands **what** data it needs.

The Repository Layer understands **how** to obtain it.

This separation allows the business to remain independent from the database implementation.

---

# Where Does the Repository Fit?

```
Controller

↓

Service

↓

Repository

↓

Database
```

The repository is the boundary between the application and the persistence layer.

Above it is business.

Below it is infrastructure.

---

# Why Does the Repository Exist?

Suppose our service directly executes Prisma queries.

```
PatientService

↓

prisma.patient.findUnique()

↓

prisma.consultation.findMany()

↓

prisma.prescription.create()
```

Initially this looks simple.

After six months,

every service contains hundreds of Prisma queries.

Changing the database becomes difficult.

Testing becomes difficult.

Database logic becomes duplicated.

Repositories prevent this.

---

# Think Like a Backend Engineer

Never ask

> "How do I write this Prisma query?"

Instead ask

> "What information does the business need?"

Example

Business

```
Load patient's consultation history.
```

Repository

```
Find patient

↓

Load consultations

↓

Return data
```

The service never asks **how**.

It asks **what**.

---

# Repository Responsibilities

Repositories are responsible for

- Reading data
- Writing data
- Updating data
- Deleting data
- Mapping database models
- Executing queries

Repositories are **not** responsible for

- Business rules
- Permissions
- Validation
- Authentication
- HTTP

---

# Example

Business operation

```
Doctor completes consultation.
```

Repository responsibilities

```
Create Consultation

↓

Create Prescription

↓

Update Appointment
```

Notice

The repository never asks

```
Can this doctor perform this action?
```

That question belongs to the service.

---

# Repository Is a Database Expert

Repositories understand

- Prisma
- SQL
- MongoDB
- Indexes
- Transactions
- Query optimization

Services should not.

Controllers should never know these details.

---

# Repository Should Return Business Data

Good

```
Find Patient

↓

Return Patient
```

Bad

```
Return HTTP Response
```

The repository should never know the client exists.

---

# Repository Does Not Decide Business Rules

Example

```
PatientRepository

↓

Find Patient
```

Correct.

Example

```
PatientRepository

↓

Reject if Doctor from another Clinic
```

Wrong.

Repositories should never enforce business policies.

---

# Engineering Note

Repositories should answer

> "Can I retrieve or persist this data?"

They should never answer

> "Should the business allow this operation?"

---

# One Repository Per Module

Our clinic application might contain

```
patient/

    patient.repository.ts

doctor/

    doctor.repository.ts

appointment/

    appointment.repository.ts

consultation/

    consultation.repository.ts
```

Each repository owns one business module.

---

# Should One Repository Call Another?

Generally,

**No.**

Example

```
PatientRepository

↓

AppointmentRepository
```

This creates unnecessary coupling.

Instead

```
PatientService

↓

PatientRepository

↓

AppointmentRepository
```

The service coordinates multiple repositories.

Repositories remain independent.

---

# Should One Service Call Multiple Repositories?

Absolutely.

Example

Doctor completes consultation.

```
ConsultationService

↓

AppointmentRepository

↓

PatientRepository

↓

ConsultationRepository

↓

PrescriptionRepository
```

The service owns the business workflow.

---

# Should One Service Call Another Service?

Sometimes.

Example

```
ConsultationService

↓

NotificationService

↓

Send Email
```

This is acceptable if responsibilities remain clear.

Avoid circular service dependencies.

If

```
PatientService

↓

AppointmentService

↓

PatientService
```

appears,

the design usually needs reconsideration.

---

# Engineering Note

A repository should never know another repository exists.

Repositories are isolated.

Services orchestrate.

---

# Repository Methods Should Describe Data Operations

Good

```
findById()

findByEmail()

findActivePatients()

create()

update()

delete()
```

Poor

```
doEverything()

executeLogic()

processPatient()
```

Method names should describe database intent.

---

# Repository Should Hide Database Details

Today

```
Prisma
```

Tomorrow

```
MongoDB
```

Ideally,

only the repository changes.

The service continues asking

```
Find Patient

Create Consultation

Update Appointment
```

The implementation changes.

The business does not.

---

# Query Optimization Lives Here

Suppose loading patient history requires

- Patient
- Consultations
- Prescriptions
- Lab Tests

The repository decides

- JOIN
- Include
- Aggregation
- Projection
- Pagination

The service should not optimize SQL.

---

# Engineering Note

Performance optimizations belong in repositories because they concern **how data is retrieved**, not **why it is needed**.

---

# Transactions

Repositories execute queries.

Services decide transactions.

Example

```
Service

↓

Begin Transaction

↓

Repository A

↓

Repository B

↓

Repository C

↓

Commit
```

Repositories should not silently start unrelated transactions.

The business operation defines the transaction boundary.

---

# Common Mistakes

## Business Logic Inside Repository

Wrong

```
Reject doctor

↓

Approve patient

↓

Calculate billing
```

Repositories should only manipulate data.

---

## Database Everywhere

Controllers

↓

Prisma

Services

↓

Prisma

Utilities

↓

Prisma

Soon,

database access becomes impossible to manage.

---

## Repository Returning HTTP Objects

Wrong

```
return res.json(...)
```

Repositories should never know Express exists.

---

## Massive Repository

Example

```
PatientRepository

↓

5000 Lines
```

Usually indicates

- too many responsibilities
- poor module boundaries
- unrelated queries grouped together

---

# Trade-offs

### Repository Pattern

Advantages

- Centralized database access
- Easier testing
- Easier refactoring
- Cleaner business layer

Disadvantages

- Additional abstraction
- Small CRUD projects may not need it

Architecture should match project complexity.

---

# Senior Engineer Review Questions

When reviewing a repository,

ask

- Does it only access data?
- Does it avoid business decisions?
- Does it hide ORM details?
- Can another database replace the current one with minimal service changes?
- Are queries reusable?
- Are methods named by data intent?

If the answer is "Yes",

the repository is likely well designed.

---

# Summary

Repositories isolate the application from the persistence layer.

They retrieve and store data, hide database implementation details, and provide reusable data-access methods for the Service Layer.

Repositories should remain focused on persistence and avoid business decisions.

Keeping this boundary clean allows the business layer to evolve independently from the database technology.

---

# Next Chapter

The next chapter covers **Database Schema Design**.

Now that we understand how the application is structured, we can design a database that reflects the business domain instead of driving it.
