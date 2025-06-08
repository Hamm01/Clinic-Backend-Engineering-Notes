# 04 - Request Lifecycle

> Understanding how a request travels through the backend before understanding how data is stored.

---

# Objective

Every backend application receives requests, performs business operations, and returns responses.

Before writing controllers, services, or database queries, it is important to understand the complete lifecycle of a request.

This chapter explains how a single request moves through the application and why each layer exists.

Throughout this chapter, we will use the following business action:

> A doctor wants to open a patient's profile.

---

# The Request

A doctor clicks:

```
View Patient
```

The frontend sends a request.

```
GET /patients/:patientId
```

This request enters our backend application.

Everything that happens after this point is the backend request lifecycle.

---

# Complete Lifecycle

Every request follows the same high-level flow.

```
Client

↓

Route

↓

Middleware

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

↓

Repository

↓

Service

↓

Controller

↓

Response
```

Not every request requires every layer, but this is the standard flow for most protected endpoints.

---

# Step 1 — Route

The request first reaches a route.

Example:

```ts
GET /patients/:patientId
```

The route has one responsibility:

- Match the incoming URL.
- Execute configured middleware.
- Forward the request to the correct controller.

Routes should never contain business logic.

Think of a route as a traffic signal directing requests.

---

# Step 2 — Middleware

Middleware executes before the controller.

Middleware performs work that is common across multiple endpoints.

Examples:

- Authentication
- Logging
- Rate limiting
- Request ID generation
- CORS
- Compression

A middleware should solve one reusable concern.

Business rules should not be implemented here.

---

# Step 3 — Authentication

Authentication answers one question:

> Who is making this request?

Example:

The doctor sends a JWT.

The backend verifies the token.

If valid:

```
Doctor ID

Clinic ID

Role
```

become available for the rest of the request.

Authentication does not decide what the doctor can do.

It only identifies the requester.

---

# Step 4 — Authorization

Authorization answers a different question.

> Is this doctor allowed to perform this action?

Example:

Doctor belongs to

```
Clinic A
```

Patient belongs to

```
Clinic B
```

Result:

Access denied.

Authentication succeeded.

Authorization failed.

Never confuse these two concepts.

---

# Step 5 — Validation

Validation ensures incoming data is acceptable.

Example:

```
patientId

must be UUID
```

Example:

```
age

must be positive
```

Example:

```
appointmentDate

cannot be in the past
```

Validation protects the application from invalid input.

It does not contain business logic.

---

# Step 6 — Controller

Once the request is valid, the controller executes.

The controller has one purpose:

Translate an HTTP request into a business request.

Example responsibilities:

- Read parameters
- Read request body
- Read authenticated user
- Call service
- Return HTTP response

A controller should remain small.

It should not contain SQL.

It should not implement business rules.

---

# Step 7 — Service

The service is the heart of the backend.

Business logic lives here.

Example:

Doctor wants to view patient history.

The service decides:

- Does patient exist?
- Is doctor active?
- Does patient belong to doctor's clinic?
- Should archived records be hidden?
- Should sensitive information be masked?

These decisions belong to the business.

Not the controller.

Not the database.

---

# Step 8 — Repository

Once the service needs data, it calls a repository.

The repository performs database operations.

Example:

```
Find patient

Load appointments

Load consultations
```

The repository knows:

- Prisma
- SQL
- MongoDB
- Database indexes

The service should never know these implementation details.

---

# Step 9 — Database

The repository communicates with the database.

The database stores information.

It should not decide business rules.

Business rules belong to the service layer.

The database provides persistence.

---

# Step 10 — Returning the Response

After data is retrieved:

Repository

↓

Service

↓

Controller

↓

Client

The controller converts the result into an HTTP response.

Example:

```
200 OK
```

with patient information.

---

# Complete Example

Doctor opens patient profile.

```
Doctor clicks

↓

GET /patients/123

↓

Route matches

↓

Authenticate JWT

↓

Authorize doctor

↓

Validate patientId

↓

Controller executes

↓

PatientService.viewPatient()

↓

PatientRepository.findById()

↓

Database

↓

Repository returns patient

↓

Service applies business rules

↓

Controller returns response

↓

Doctor sees patient profile
```

Every request follows this general pattern.

---

# Responsibilities of Each Layer

| Layer          | Responsibility                   |
| -------------- | -------------------------------- |
| Route          | Match endpoint                   |
| Middleware     | Shared request processing        |
| Authentication | Identify requester               |
| Authorization  | Verify permissions               |
| Validation     | Validate input                   |
| Controller     | Handle HTTP request and response |
| Service        | Execute business logic           |
| Repository     | Access database                  |
| Database       | Store and retrieve data          |

Each layer has one responsibility.

Avoid mixing responsibilities.

---

# Where Does Business Logic Live?

Business logic belongs inside services.

Example:

Correct:

```
Doctor cannot access patients from another clinic.
```

Incorrect locations:

- Route
- Middleware
- Controller
- Repository
- Database Trigger

Correct location:

```
PatientService
```

Business rules should remain independent of HTTP and database technologies.

---

# Why Separate These Layers?

Each layer solves a different problem.

Routes understand URLs.

Controllers understand HTTP.

Services understand business.

Repositories understand databases.

Because responsibilities are separated:

- code becomes easier to maintain
- testing becomes simpler
- changing the database affects fewer files
- business rules stay centralized

---

# Common Mistakes

### Fat Controllers

Controllers performing validation, business logic, and SQL queries.

Result:

Large files that are difficult to maintain.

---

### SQL Inside Services

Services should request data.

Repositories should retrieve data.

---

### Business Logic Inside Middleware

Middleware should remain generic.

Business decisions belong to services.

---

### Database Deciding Business Rules

The database stores data.

The application decides business behavior.

---

# Checklist

For every request, ask:

- Which route receives it?
- Which middleware executes?
- Who is authenticated?
- Is authorization required?
- Is the request validated?
- Which controller handles it?
- Which service owns the business rule?
- Which repository accesses the database?
- What response should be returned?

If every question has a clear answer, the request lifecycle is well designed.

---

# Summary

A backend request passes through multiple layers before reaching the database.

Each layer exists for a specific reason.

Understanding these responsibilities is more important than memorizing framework syntax.

In the next chapter, we will begin implementing the first layer of this lifecycle by understanding **Authentication**—how users prove their identity before accessing protected resources.
