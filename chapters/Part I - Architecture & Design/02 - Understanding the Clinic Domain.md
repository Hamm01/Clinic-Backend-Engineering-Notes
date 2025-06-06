# 02 - Understanding the Clinic Domain

> A backend should model the business domain before modelling the database.

---

# Objective

In the previous chapter, we identified the business problem and the actors involved in the system.

In this chapter, we define the **business domain** of our Clinic Management System.

The goal is to identify the core business objects, understand how they relate to each other, and establish clear ownership of responsibilities before designing APIs or database schemas.

At the end of this chapter, we should be able to answer:

- What objects exist in the system?
- Why do they exist?
- Who owns them?
- How are they connected?
- Which module should manage them?

---

# What is a Domain?

A domain is the real-world business that the software is built to solve.

For this project, the domain is:

> Managing multiple clinics where doctors consult patients, maintain medical records, prescribe medicines, recommend laboratory tests, and manage appointments.

The backend exists to represent this business digitally.

Everything we build later should directly support this domain.

---

# Identify Core Business Objects

The first step is identifying the major business objects.

For our clinic application, the core objects are:

| Object | Purpose |
|----------|---------|
| Clinic | Represents a physical healthcare center. |
| Doctor | Medical professional providing consultations. |
| Patient | Person receiving medical care. |
| Appointment | Scheduled meeting between doctor and patient. |
| Consultation | Doctor's examination during an appointment. |
| Prescription | Medicines prescribed during consultation. |
| Lab Test | Medical tests recommended during consultation. |

These are business concepts.

They are **not** database tables yet.

---

# Clinic

## Why does it exist?

A healthcare organization may own multiple clinics.

Every doctor and patient interaction belongs to a clinic.

Without a clinic, the system cannot determine where consultations or appointments occur.

## Responsibilities

- Store clinic information.
- Manage doctors assigned to the clinic.
- Manage patients visiting the clinic.
- Maintain clinic-specific records.

---

# Doctor

## Why does it exist?

Doctors perform consultations.

They diagnose illnesses, prescribe medicines, and recommend laboratory tests.

Doctors are responsible for medical decisions.

## Responsibilities

- Login to the system.
- View appointments.
- Access patient history.
- Create consultations.
- Prescribe medicines.
- Recommend lab tests.

Doctors should never manage system configuration or business administration.

---

# Patient

## Why does it exist?

Patients receive healthcare services.

A patient accumulates medical history over time through multiple visits.

## Responsibilities

- Maintain personal information.
- Maintain consultation history.
- Maintain prescription history.
- Maintain laboratory history.

Patients do not create consultations.

Doctors create consultations for patients.

---

# Appointment

## Why does it exist?

Appointments organize a doctor's schedule.

Every consultation begins with an appointment.

Without appointments:

- Scheduling becomes difficult.
- Doctor availability cannot be managed.
- Visit history becomes inconsistent.

## Responsibilities

- Store appointment date.
- Store appointment time.
- Store appointment status.
- Connect doctor and patient.

---

# Consultation

## Why does it exist?

A consultation represents the actual medical examination.

This is where business value is created.

Everything the doctor observes and decides is recorded here.

## Responsibilities

- Symptoms
- Diagnosis
- Doctor notes
- Follow-up advice

A consultation should never exist without an appointment.

---

# Prescription

## Why does it exist?

A consultation may result in medicines being prescribed.

A prescription belongs to one consultation.

## Responsibilities

- Medicine name
- Dosage
- Duration
- Frequency
- Additional instructions

---

# Lab Test

## Why does it exist?

Doctors often recommend investigations before confirming a diagnosis.

These recommendations should be stored separately.

Examples:

- Blood Test
- X-Ray
- MRI
- CT Scan

Lab tests belong to consultations.

---

# Understanding Ownership

Every business object has an owner.

Ownership determines who creates and manages the object.

| Object | Created By | Managed By |
|----------|------------|------------|
| Clinic | Admin | Admin |
| Doctor | Admin | Admin |
| Patient | Receptionist | Doctor / Receptionist |
| Appointment | Receptionist | Receptionist |
| Consultation | Doctor | Doctor |
| Prescription | Doctor | Doctor |
| Lab Test | Doctor | Doctor |

Ownership is a business concept.

It should be understood before implementing permissions.

---

# Relationships Between Objects

Each business object is connected to others.

| Relationship | Description |
|--------------|-------------|
| Clinic → Doctors | One clinic has many doctors. |
| Clinic → Patients | One clinic serves many patients. |
| Doctor → Appointments | One doctor has many appointments. |
| Patient → Appointments | One patient has many appointments. |
| Appointment → Consultation | One appointment produces one consultation. |
| Consultation → Prescription | One consultation may contain multiple medicines. |
| Consultation → Lab Test | One consultation may contain multiple recommended tests. |

These relationships describe the business.

Database relationships will be designed later.

---

# Business Journey of a Patient

Understanding the lifecycle of a patient helps identify missing business objects.

Typical journey:

1. Patient visits clinic.
2. Receptionist registers patient.
3. Appointment is scheduled.
4. Doctor opens appointment.
5. Doctor reviews previous history.
6. Consultation begins.
7. Diagnosis is recorded.
8. Medicines are prescribed.
9. Lab tests are recommended.
10. Follow-up appointment may be scheduled.

Every step in this journey becomes part of the backend.

---

# Business Boundaries

Each module should own its own responsibility.

## Clinic Module

Responsible for:

- Clinic information
- Doctor assignment

Not responsible for:

- Consultations
- Prescriptions

---

## Doctor Module

Responsible for:

- Doctor profile
- Doctor availability

Not responsible for:

- Appointment scheduling
- Patient registration

---

## Patient Module

Responsible for:

- Patient profile
- Medical history

Not responsible for:

- Authentication
- Clinic management

---

## Appointment Module

Responsible for:

- Booking
- Cancellation
- Rescheduling
- Status updates

Not responsible for:

- Diagnosis

---

## Consultation Module

Responsible for:

- Diagnosis
- Clinical notes
- Prescriptions
- Lab tests

Not responsible for:

- Scheduling appointments

---

# Business Rules Identified So Far

The domain already reveals several important business rules.

- A doctor can only consult patients assigned to their clinic.
- A consultation must belong to an appointment.
- A prescription cannot exist without a consultation.
- A lab test recommendation cannot exist without a consultation.
- A patient can have multiple appointments.
- A patient can have multiple consultations.
- A doctor can consult many patients.
- A completed consultation should become read-only.

Notice that these rules exist independently of any programming language or database.

---

# Questions Before Moving Forward

Before writing code, verify that these questions have clear answers.

- What business object am I working with?
- Who owns this object?
- Who is allowed to modify it?
- Which module should manage it?
- Which other objects depend on it?
- Which business rules protect it?

If these questions cannot be answered, the domain is still incomplete.

---

# Summary

The domain is the foundation of the backend.

By identifying business objects, responsibilities, ownership, and relationships first, we create a backend that reflects the real-world business instead of the database.

The next chapter will transform this domain into a practical project structure by deciding how these business objects become backend modules, folders, and application layers without mixing responsibilities.