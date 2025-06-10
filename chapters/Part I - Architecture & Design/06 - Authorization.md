# 06 - Authorization

> Authorization answers one question:
>
> **"Is the authenticated user allowed to perform this action?"**

---

# Objective

In the previous chapter, we learned how the backend identifies a user through authentication.

Once the user's identity is known, the backend must decide whether that user has permission to perform the requested action.

This process is called **Authorization**.

Authentication and authorization solve different problems and should always remain independent.

---

# Authentication vs Authorization

Authentication answers

```
Who is making this request?
```

Authorization answers

```
What is this user allowed to do?
```

Both are required.

Knowing who the user is does not automatically mean they can perform every action.

---

# Authorization in Our Clinic Application

Actors

- Admin
- Doctor
- Receptionist

Each actor has different responsibilities.

Example

Doctor

✓ View patient history

✓ Create consultation

✓ Prescribe medicine

Receptionist

✓ Register patient

✓ Book appointment

✗ Prescribe medicine

Admin

✓ Manage clinics

✓ Manage doctors

✓ View reports

Authorization protects these business rules.

---

# Why Authorization Exists

Suppose authentication was enough.

Any authenticated doctor could

- View every patient
- Delete appointments
- Create clinics
- Access another doctor's consultations

Clearly this is unacceptable.

Authentication identifies users.

Authorization restricts actions.

---

# Authorization Begins With Business Rules

Never start by writing middleware.

Start by writing the business rule.

Example

```
Only doctors can prescribe medicines.
```

Another example

```
Doctors may only access patients
assigned to their clinic.
```

Another

```
Only administrators can create clinics.
```

Every authorization rule should originate from a business requirement.

---

# Types of Authorization

There are multiple ways to authorize users.

The most common approaches are:

### Role-Based Access Control (RBAC)

Permissions are based on roles.

Example

```
Doctor

Receptionist

Admin
```

Each role has predefined permissions.

This is the approach we will use.

---

### Permission-Based Access Control

Instead of roles,

permissions are assigned individually.

Example

```
appointment:create

patient:view

consultation:update
```

Useful for larger enterprise systems.

---

### Ownership-Based Authorization

The user can only access data they own.

Example

Doctor

↓

Own Clinic

↓

Own Patients

↓

Own Consultations

Ownership is checked during business logic.

---

# Authorization Flow

Every protected request follows this sequence.

```
Request

↓

Authentication

↓

Identity Available

↓

Authorization

↓

Business Logic

↓

Database
```

Authorization should happen before business logic begins.

---

# Example

Doctor requests

```
GET /patients/123
```

Backend verifies

- JWT valid
- Doctor authenticated

Next,

authorization verifies

- Is this doctor active?
- Does this doctor belong to this clinic?
- Does this patient belong to the same clinic?

If any answer is "No",

access is denied.

The controller never executes business logic.

---

# Authorization Is a Business Decision

Authorization is often confused with authentication middleware.

Example

Business Rule

```
Doctor can only view patients
assigned to their clinic.
```

This is not an HTTP rule.

This is not a database rule.

It is a business rule.

The backend enforces it because the business requires it.

---

# Where Should Authorization Live?

Simple authorization

Example

```
Admin Only
```

can be handled using reusable middleware.

Example

```
Require Role

↓

Admin
```

Complex authorization

Example

```
Doctor belongs to clinic

AND

Patient belongs to clinic

AND

Consultation is active
```

should be verified inside the service.

The service has access to business data.

Middleware usually does not.

---

# Engineering Note

Not every authorization rule belongs inside middleware.

Middleware is useful for generic rules.

Business-specific authorization should remain inside the service layer because it often requires database lookups and business knowledge.

---

# Authorization Responsibilities

Authorization is responsible for

- Verifying permissions
- Restricting protected actions
- Enforcing business ownership
- Protecting sensitive resources

Authorization is not responsible for

- Logging users in
- Validating request data
- Processing business operations
- Reading HTTP parameters

---

# Authorization Examples

## Example 1

Doctor prescribes medicine.

Allowed

Doctor

Denied

Receptionist

Denied

Admin

---

## Example 2

Receptionist creates appointment.

Allowed

Receptionist

Allowed

Admin

Denied

Doctor

---

## Example 3

Doctor opens patient history.

Allowed

Doctor assigned to clinic

Denied

Doctor from another clinic

---

# Authorization Layers

Authorization can happen at different levels.

### Route Level

Protect entire endpoints.

Example

```
Admin routes
```

---

### Module Level

Protect an entire module.

Example

```
Reports Module

↓

Admin Only
```

---

### Business Level

Protect individual business rules.

Example

```
Doctor

↓

Patient

↓

Same Clinic?
```

↓

Allow

or

↓

Reject

Business-level authorization is usually the most important.

---

# Common Mistakes

### Assuming Authentication Is Enough

Wrong

```
User Logged In

↓

Allow Everything
```

Authentication proves identity.

It does not grant unlimited access.

---

### Putting Every Rule Into Middleware

Eventually middleware becomes responsible for

- permissions
- ownership
- validation
- business logic

This creates tightly coupled code.

---

### Trusting Client Roles

Never trust

```
role

clinicId

doctorId
```

received from the client.

Always use the authenticated identity generated by the backend.

---

### Hardcoding Permissions

Avoid

```
if(role === "doctor")
```

throughout the project.

Permission logic should remain centralized.

---

# Trade-offs

### Role-Based Authorization

Advantages

- Easy to understand
- Easy to implement
- Suitable for most applications

Disadvantages

- Less flexible as permissions grow

---

### Permission-Based Authorization

Advantages

- Highly flexible
- Fine-grained control

Disadvantages

- More complex
- Requires permission management

Choose the simplest solution that satisfies the business.

---

# Checklist

Before implementing authorization,

answer the following.

- Which actors exist?
- Which actions can each actor perform?
- Which resources require protection?
- Which rules depend on ownership?
- Which rules depend on roles?
- Which rules belong inside middleware?
- Which rules belong inside services?

If these questions are answered,

authorization design is complete.

---

# Summary

Authorization protects business rules.

It determines whether an authenticated user is permitted to perform a specific action.

The backend should always derive authorization rules from business requirements rather than technical implementation.

Keeping authorization separate from authentication produces a cleaner, more maintainable architecture.

---

# Next Chapter

The next chapter covers **Validation**.

Before business logic executes, the backend must ensure that incoming data is complete, correctly formatted, and safe to process.
