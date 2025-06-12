# 07 - Validation

> Validation answers one question:
>
> **"Is the incoming data safe and complete enough to execute the requested business operation?"**

---

# Objective

Before a request reaches the business layer, the backend must verify that the incoming data is structurally correct.

Validation ensures that requests contain the required fields, correct data types, and acceptable values before any business logic executes.

Validation is **not** responsible for deciding whether an operation is allowed.

Its only responsibility is to verify that the request itself is valid.

---

# Why Validation Exists

The backend should never assume that incoming requests are correct.

Every request originates from outside the application.

The client may be:

- Frontend
- Mobile Application
- Third-party API
- Internal Service
- Manual API Testing (Postman)

All external input should be considered untrusted until validated.

---

# Validation in Our Clinic Application

Example

Receptionist creates a patient.

```
POST /patients
```

Incoming Request

```json
{
  "name": "Rahul Sharma",
  "age": 28,
  "phone": "9876543210"
}
```

Before creating the patient, the backend should verify

- Name exists
- Name is a string
- Age is a number
- Age is positive
- Phone number format is valid

Only after validation succeeds should the request continue.

---

# Validation vs Business Logic

These concepts are often confused.

Validation asks

```
Is this request correctly formed?
```

Business Logic asks

```
Should the system allow this operation?
```

Example

Validation

```
Appointment Date

↓

Is it a valid date?
```

Business Logic

```
Doctor

↓

Available on that date?
```

The first is validation.

The second is business logic.

---

# Examples

## Patient Registration

Validation

- Name is required
- Phone number is required
- Age must be positive

Business Logic

- Patient already exists?
- Clinic reached patient limit?
- Duplicate registration?

---

## Doctor Login

Validation

- Email exists
- Password exists
- Email format is valid

Business Logic

- Doctor exists
- Password matches
- Account active

---

## Appointment Booking

Validation

- Doctor ID provided
- Patient ID provided
- Appointment date provided

Business Logic

- Doctor available?
- Patient already booked?
- Clinic open on selected day?

---

# Types of Validation

Validation usually happens in several forms.

---

## Required Fields

Example

```
Name

Required
```

```
Phone

Required
```

Missing required information should immediately reject the request.

---

## Type Validation

Example

Correct

```
Age

28
```

Incorrect

```
Age

"Twenty Eight"
```

The backend should verify data types before processing.

---

## Format Validation

Examples

- Email format
- Phone number
- UUID
- Date format

Correct structure does not necessarily mean valid business data.

---

## Value Validation

Examples

Age

```
0

Invalid
```

Medicine Quantity

```
-5

Invalid
```

Appointment Duration

```
30 Minutes

Valid
```

Validation ensures values fall within acceptable limits.

---

## Enum Validation

Example

Appointment Status

Allowed

- Scheduled
- Completed
- Cancelled

Rejected

```
Running
```

---

# Where Should Validation Happen?

Validation should happen before controllers begin business processing.

Typical request flow

```
Request

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
```

If validation fails,

the request should stop immediately.

---

# What Validation Should Check

Validation should verify

- Required fields
- Data types
- Formats
- Allowed values
- Minimum values
- Maximum values
- String length
- Array structure
- Object structure

Validation should not verify

- User permissions
- Business ownership
- Database records
- Business rules

---

# Engineering Note

A request can be perfectly valid but still fail.

Example

```
Appointment Date

Valid

↓

Doctor Already Booked

Rejected
```

The request structure is correct.

The business operation is not.

Never move business decisions into validation.

---

# Validation Responsibilities

Validation is responsible for

- Request shape
- Required fields
- Correct types
- Correct formats
- Safe input

Validation is not responsible for

- Authentication
- Authorization
- Database queries
- Business calculations
- Permission checks

Keep validation focused on the incoming request.

---

# Where Should Validation Live?

Typical module structure

```
modules/

    patient/

        patient.validator.ts
```

Example

```
patient.validator.ts

↓

Create Patient

Update Patient

Delete Patient
```

Each module owns its own validation rules.

Shared validation utilities can be placed inside

```
shared/

    validators/
```

Examples

- UUID validation
- Pagination validation
- Date validation

Only reusable validation belongs in shared.

---

# Validation Libraries

Most production applications use validation libraries instead of manually checking every field.

Common choices

- Zod
- Joi
- Yup
- class-validator

The library is a technical decision.

The validation rules come from the business.

---

# Validation Errors

Validation errors should be predictable.

Example

```
400 Bad Request
```

Response

```
Field

↓

Reason

↓

Expected Value
```

Clients should immediately understand what needs to be corrected.

Avoid returning vague messages such as

```
Something went wrong.
```

---

# Common Mistakes

### Validation Inside Controllers

Bad

```
Controller

↓

if (!name)

↓

if (!phone)

↓

if (!age)
```

Controllers become difficult to maintain.

---

### Validation Inside Services

Services should assume validated input.

Moving validation into services duplicates work.

---

### Business Validation Mixed With Request Validation

Wrong

```
Age > 18

↓

Doctor Exists

↓

Password Correct
```

These belong to different layers.

Keep responsibilities separated.

---

### Duplicating Validation

Avoid writing the same validation rules in multiple modules.

Shared validation should live in reusable utilities.

---

# Trade-offs

### Manual Validation

Advantages

- Complete control
- No external dependency

Disadvantages

- Repetitive
- Difficult to maintain
- Easy to introduce inconsistencies

---

### Validation Library

Advantages

- Reusable
- Consistent
- Easier to maintain
- Better developer experience

Disadvantages

- Additional dependency
- Learning curve

For production applications, a validation library is usually the better choice.

---

# Checklist

Before implementing validation,

verify the following.

- Which fields are required?
- Which fields are optional?
- Which data types are expected?
- Which formats are accepted?
- Which values are allowed?
- Which rules belong to validation?
- Which rules belong to business logic?

If these questions are answered,

validation is correctly designed.

---

# Summary

Validation protects the backend from malformed requests.

It verifies that incoming data is complete, correctly structured, and safe to process before any business logic executes.

A validated request does not guarantee that the business operation will succeed.

It only guarantees that the request is well-formed.

Keeping validation separate from business logic makes the backend easier to understand, maintain, and extend.

---

# Next Chapter

The next chapter covers **Controllers**.

Once a request has been authenticated, authorized, and validated, it reaches the first application layer responsible for coordinating the request—the Controller.

There we will understand why controllers should remain thin, what responsibilities they own, and what should never be implemented inside them.
