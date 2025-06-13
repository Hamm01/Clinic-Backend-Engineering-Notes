# 09 - Services

> The Service Layer is where the application implements the business.

---

# Objective

The Service Layer is responsible for implementing the business requirements of the application.

Every business operation should be understandable by reading the service.

When someone asks,

> "What happens when a doctor completes a consultation?"

the answer should be found inside the service layer.

The service is the heart of the backend.

Everything before the service prepares the request.

Everything after the service supports the request.

---

# Where Does the Service Fit?

The request lifecycle now looks like this.

```
Client

↓

Route

↓

Authentication

↓

Authorization

↓

Validation

↓

Controller

↓

Service

↓

Repository

↓

Database
```

Everything above the service understands HTTP.

Everything below the service understands infrastructure.

The service understands only the business.

---

# Why Does the Service Exist?

Imagine there is no service layer.

The controller now has to

- validate business rules
- load patients
- load doctors
- compare clinics
- create consultation
- create prescription
- send notifications
- update appointment

Controllers become responsible for everything.

As the project grows,

maintenance becomes difficult.

The service exists to isolate business decisions from technical implementation.

---

# Thinking in Business

Do not think

```
GET /patients
```

Think

```
Doctor wants to view patient history.
```

Do not think

```
POST /consultation
```

Think

```
Doctor completes a consultation.
```

Services should describe business operations,

not HTTP endpoints.

---

# Business Logic

Business logic is every rule that exists because the business requires it.

Example

```
Doctor can only consult patients
from assigned clinics.
```

This rule exists because of the business.

Not because of Express.

Not because of PostgreSQL.

Not because of Prisma.

---

# Examples of Business Logic

Appointment

- Doctor must exist.
- Patient must exist.
- Appointment must be scheduled.
- Appointment cannot be completed twice.

Consultation

- Consultation belongs to an appointment.
- Only doctors create consultations.
- Consultation becomes read-only after completion.

Prescription

- Prescription cannot exist without consultation.
- Medicine dosage must be recorded.
- Expired medicines cannot be prescribed.

Notice something.

These rules remain the same even if the database changes.

---

# What Belongs Inside Services?

Services coordinate the business.

Typical responsibilities

- Execute business rules
- Coordinate repositories
- Coordinate multiple modules
- Start transactions
- Publish events
- Call external services when required

The service decides

**what** should happen.

Repositories decide

**how** data is retrieved.

---

# Example

Business Requirement

```
Doctor completes consultation.
```

The service may perform

```
Load Appointment

↓

Verify Appointment Exists

↓

Verify Doctor Owns Appointment

↓

Verify Appointment Not Completed

↓

Create Consultation

↓

Create Prescription

↓

Recommend Lab Tests

↓

Update Appointment Status

↓

Publish Notification

↓

Return Result
```

One business operation.

Multiple technical operations.

The service coordinates all of them.

---

# Engineering Note

If a business operation requires multiple repositories,

the coordination almost always belongs inside the service.

Repositories should never coordinate each other.

---

# Services Coordinate Repositories

Example

Creating a consultation requires

```
AppointmentRepository

↓

PatientRepository

↓

ConsultationRepository

↓

PrescriptionRepository
```

The service calls each repository when needed.

Repositories should never call each other.

Doing so creates tightly coupled data access.

---

# Services Should Not Understand HTTP

Services should never know

- Express
- Request
- Response
- Status Codes
- Headers
- Cookies

These belong to the controller.

The service only understands business operations.

Example

Good

```
Complete Consultation
```

Bad

```
POST /consultation
```

---

# Services Should Not Understand SQL

A service should never care whether data comes from

- PostgreSQL
- MongoDB
- Redis

Instead,

the service asks

```
Find Patient

↓

Update Appointment

↓

Create Consultation
```

Repositories handle implementation.

---

# Services Should Not Perform Validation

Validation already happened.

The service assumes

```
Incoming Data

↓

Already Valid
```

The service focuses entirely on business decisions.

---

# Business Flow Example

Doctor opens patient history.

Service

```
Load Doctor

↓

Load Patient

↓

Verify Same Clinic

↓

Load Consultations

↓

Load Prescriptions

↓

Sort History

↓

Return Timeline
```

Notice

No HTTP.

No SQL.

Only business.

---

# Transactions Usually Begin Here

Suppose

Doctor completes consultation.

Operations

```
Create Consultation

↓

Create Prescription

↓

Create Lab Tests

↓

Update Appointment
```

If one operation fails,

everything should rollback.

The service decides

"This entire operation is one transaction."

The repository simply executes queries.

---

# Engineering Note

The service owns the transaction because the service understands the complete business operation.

The repository only understands individual database operations.

---

# Services May Call Other Services

Sometimes,

business operations depend on another module.

Example

```
Consultation Service

↓

Notification Service

↓

Send Consultation Summary
```

This is acceptable when responsibilities remain clear.

Avoid creating circular dependencies between services.

---

# When Should a Service Be Split?

Example

PatientService becomes responsible for

- registration
- appointments
- billing
- prescriptions
- reports
- notifications

The service now owns multiple business capabilities.

Split it into

```
PatientService

AppointmentService

BillingService

ReportService
```

Each service should own one business capability.

---

# What Does NOT Belong Inside Services?

Avoid placing

- HTTP responses
- Route handling
- JWT decoding
- Request validation
- SQL queries
- Prisma queries
- Express middleware

inside the service.

The service should remain independent of frameworks.

---

# Common Mistakes

### Fat Controllers

Controller performs business logic.

Service becomes almost empty.

This defeats the purpose of the architecture.

---

### Database Queries Everywhere

Controllers

↓

Repositories

↓

Utilities

↓

Services

All querying the database independently.

Business flow becomes impossible to understand.

---

### Business Rules Inside Repository

Example

```
Repository

↓

Verify Doctor Owns Clinic
```

Repositories should retrieve data.

Business decisions belong to services.

---

### One Giant Service

A service managing every business operation eventually becomes another monolith.

Split services by business capability.

---

# Trade-offs

### Thin Services

Advantages

- Easy to understand
- Easier testing
- Clear ownership

Disadvantages

- More service files

---

### Large Services

Advantages

- Fewer files initially

Disadvantages

- Difficult to maintain
- Difficult to test
- Business logic becomes mixed
- Hard to extend

Prefer business-focused services over large generic services.

---

# Service Checklist

Before writing a service,

ask

- Which business operation does this implement?
- Which business rules belong here?
- Which repositories are required?
- Does this operation require a transaction?
- Does the service understand HTTP?
- Does the service understand SQL?
- Can another developer understand this business flow by reading the service?

If every answer is clear,

the service is correctly designed.

---

# Summary

The Service Layer is the core of the backend.

It implements business rules, coordinates repositories, manages transactions, and orchestrates business operations.

A well-designed service should describe the business process, not the technology used to implement it.

If controllers become thin and repositories remain simple, the service naturally becomes the place where the application's behavior is defined.

---

# Next Chapter

The next chapter covers **Repositories**.

We will understand why repositories exist, what responsibilities they own, why they should remain independent of business logic, and how they isolate the application from database implementation.
