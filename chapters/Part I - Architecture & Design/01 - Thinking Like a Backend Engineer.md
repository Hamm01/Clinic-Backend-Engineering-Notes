# 01 - Thinking Like a Backend Engineer

> Before writing a single line of code, understand the business you are solving.

---

# Objective

This chapter explains the mindset required before building a backend application.

The purpose is **not** to design APIs, create database tables, or choose a framework.

The purpose is to understand the business problem so that every technical decision has a reason behind it.

---

# Backend Exists to Solve Business Problems

A backend application is not a collection of routes, controllers, or database queries.

It is a software system that implements business requirements.

Everything else—Express, PostgreSQL, Prisma, Redis—is only a tool used to implement those requirements.

Never begin by asking:

- Which database should we use?
- Which ORM should we use?
- Should we use Express or NestJS?

Begin by asking:

- What problem are we solving?
- Who will use the system?
- What actions can they perform?
- What rules must always be enforced?

Technology should always be selected after understanding the business.

---

# Example Business

Throughout this repository, we will build a Clinic Management System.

The system allows doctors to manage patients across multiple clinics.

Actors:

- Admin
- Doctor
- Receptionist

Core Resources:

- Clinic
- Doctor
- Patient
- Appointment
- Consultation
- Prescription
- Lab Test

---

# Step 1 — Understand the Business

Before opening your editor, write a short business description.

Example:

> A healthcare company owns multiple clinics. Doctors work in one or more clinics. Patients visit clinics, book appointments, receive consultations, and are prescribed medicines. Doctors should only access patients belonging to their clinics.

This single paragraph defines the entire project.

Everything implemented later should satisfy this business description.

---

# Step 2 — Identify Actors

Actors are people or systems interacting with the application.

For the clinic application:

| Actor        | Responsibility                    |
| ------------ | --------------------------------- |
| Admin        | Manages clinics and doctors       |
| Doctor       | Consults patients                 |
| Receptionist | Manages appointments and patients |

Each actor has different permissions.

Do not think about APIs yet.

Think about responsibilities.

---

# Step 3 — Identify Resources

Resources are the main entities managed by the business.

Example:

- Clinic
- Doctor
- Patient
- Appointment
- Consultation
- Prescription

These resources usually become database models later, but at this stage they represent business concepts—not tables.

---

# Step 4 — Identify Actions

Determine what each actor can do.

Doctor

- Login
- View appointments
- View patient history
- Create consultation
- Prescribe medicines
- Recommend lab tests

Receptionist

- Register patient
- Book appointment
- Reschedule appointment
- Cancel appointment

Admin

- Create clinic
- Add doctor
- Remove doctor
- Assign doctor to clinic

Actions describe business behavior.

Routes and controllers are implementation details.

---

# Step 5 — Identify Business Rules

Business rules are conditions that must always remain true.

Examples:

- A doctor can only access patients from assigned clinics.
- A consultation cannot exist without an appointment.
- A completed consultation cannot be edited.
- Only doctors can prescribe medicines.
- A cancelled appointment cannot be completed.

Business rules are the most important part of the backend.

If these rules are implemented correctly, changing frameworks or databases becomes much easier.

---

# Step 6 — Identify Modules

Group related functionality into modules.

Example:

Auth

- Login
- JWT
- Password Management

Clinic

- Create Clinic
- Update Clinic

Doctor

- Doctor Profile
- Doctor Assignment

Patient

- Registration
- Patient History

Appointment

- Booking
- Cancellation
- Rescheduling

Consultation

- Diagnosis
- Prescription
- Lab Tests

Modules represent business capabilities.

---

# Step 7 — Identify Relationships

Understand how resources relate to each other.

Example:

- A clinic has many doctors.
- A clinic has many patients.
- A patient has many appointments.
- An appointment has one consultation.
- A consultation contains prescriptions.
- A consultation contains lab tests.

These relationships help design both the database and the business logic.

---

# Step 8 — Think About Permissions

Every action should answer:

Who can perform this?

Example:

| Action               | Doctor | Receptionist | Admin |
| -------------------- | :----: | :----------: | :---: |
| Login                |   ✓    |      ✓       |   ✓   |
| Register Patient     |   ✗    |      ✓       |   ✓   |
| View Patient History |   ✓    |      ✗       |   ✓   |
| Prescribe Medicine   |   ✓    |      ✗       |   ✗   |
| Create Clinic        |   ✗    |      ✗       |   ✓   |

Permissions should be identified before writing APIs.

---

# Step 9 — Think About Data Ownership

Every piece of data belongs to someone.

Example:

Patient History

Owner:

- Clinic

Accessible By:

- Assigned Doctors
- Admin

Not Accessible By:

- Doctors from other clinics

Ownership determines authorization.

---

# Step 10 — Think About Future Growth

Before implementation, ask:

- Will there be multiple clinics?
- Can doctors work in multiple clinics?
- Will appointments require reminders?
- Will notifications be added later?
- Will lab systems integrate in future?
- Will reports be generated?

Thinking ahead prevents unnecessary redesign later.

Do not over-engineer.

Simply identify likely future requirements.

---

# What We Still Ignore

At this stage, we intentionally ignore:

- Express
- Routes
- Controllers
- Services
- Repositories
- Prisma
- PostgreSQL
- MongoDB
- Redis
- Folder Structure

Those decisions come after the business is understood.

---

# Common Mistakes

### Starting with Database Design

Wrong approach:

"I'll create all database tables first."

Without understanding business rules, database design often becomes incorrect.

---

### Starting with Routes

Wrong approach:

```
POST /patients

GET /patients
```

Routes describe implementation.

They do not describe the business.

---

### Choosing Technology Too Early

Avoid questions like:

- Express or Nest?
- MongoDB or PostgreSQL?
- Prisma or TypeORM?

Technology should support business—not define it.

---

# Senior Engineer Checklist

Before writing code, confirm that you can answer:

- What problem are we solving?
- Who are the actors?
- What resources exist?
- What actions are supported?
- What business rules must never be violated?
- How are resources related?
- Who owns the data?
- Who can access the data?
- What future requirements are already visible?

If any answer is unclear, implementation should wait.

---

# Summary

A backend engineer does not begin by writing code.

The first responsibility is understanding the business.

The implementation should always be a reflection of business requirements, not the other way around.

The remaining chapters will show how these business decisions gradually become folders, APIs, services, repositories, and database queries while keeping the business logic independent of implementation details.
