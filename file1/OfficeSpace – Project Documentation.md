# OfficeSpace – Project Documentation

Nov 16, 2026 · @Ritesh Singh

## 1. Project overview

OfficeSpace is a web application that lets employees of an IT company check availability and book shared resources by date and time slot, with manager approval where required.

**Problem.** Manual booking causes scheduling conflicts, employees cannot see what is free, multiple bookings are hard to manage, and no central record of resource usage exists.

**Solution.** One platform for browsing resources, booking them, approving requests and tracking usage.

## 2. Requirements

The system supports five bookable resource types and three user roles.

**Resources:** meeting rooms, interview rooms, training rooms, projectors, testing devices. Example: Meeting Room MR-101 booked 2:00–3:00 PM for a team meeting.

| Capability | Employee | Manager | Admin |
| --- | --- | --- | --- |
| View available resources | Yes | Yes | Yes |
| Create, modify, cancel own bookings | Yes | Yes | Yes |
| Receive confirmations and reminders | Yes | Yes | Yes |
| View team bookings and utilization reports | No | Yes | Yes |
| Approve or reject booking requests | No | Yes | Yes |
| Manage users, roles, departments, teams | No | No | Yes |
| Add, update, remove resources and availability | No | No | Yes |
| Manage working hours, holidays, maintenance | No | No | Yes |
| View and manage all company bookings | No | No | Yes |
| Configure approval requirements | No | No | Yes |

## 3. Booking workflow and rules

A booking is created only if the resource is free for the requested slot, and goes to a manager only if approval is required.

1. Employee logs in.
2. Employee selects a resource, date and time.
3. System checks availability.
4. If available, a booking request is created.
5. If approval is required, the request goes to the manager, who approves or rejects it.
6. Employee receives the final status.

| Status | Meaning |
| --- | --- |
| PENDING | Waiting for manager approval |
| CONFIRMED | Approved |
| REJECTED | Request rejected |
| CANCELLED | Booking cancelled |

**Core business rule: no overlapping bookings for the same resource.** A slot ending at 11:00 does not conflict with one starting at 11:00.

| Existing booking (Room A) | New request | Result |
| --- | --- | --- |
| 10:00–11:00 | 10:30–11:30 | Not allowed (overlap) |
| 10:00–11:00 | 11:00–12:00 | Allowed (back to back) |

## 4. Architecture

The frontend calls the Spring Boot backend over REST, and only the Repository layer touches PostgreSQL.

&#91;embedded content: system architecture · 3 backend layers\]

Controllers receive requests and return responses, services hold the business logic, and repositories use Spring Data JPA. Entities map to tables, and DTOs carry data between frontend and backend so entities are never exposed directly. Spring Security with JWT enforces role-based access; for example, only an Admin can create or delete resources.

## 5. Database design (PostgreSQL)

Four tables: one user creates many bookings, one resource has many bookings, a booking has at most one approval record, and one manager approves many bookings.

&#91;image: image.png\]

**users**

| Column | Type | Notes |
| --- | --- | --- |
| id | BIGINT | Primary key |
| name | VARCHAR(100) |  |
| email | VARCHAR(150) | Unique |
| password | VARCHAR(255) | Hashed |
| role | VARCHAR(20) | EMPLOYEE / MANAGER / ADMIN |
| department | VARCHAR(100) | ENGINEER / HR / FINANCE |
| manager\_id | BIGINT | User ID of the manager |

**resources**

| Column | Type | Notes |
| --- | --- | --- |
| id | BIGINT | Primary key |
| name | VARCHAR(100) |  |
| type | VARCHAR(50) | MEETING ROOM / INTERVIEW ROOM etc |
| capacity | INT | Maximum capacity |
| status | VARCHAR(20) | ACTIVE / INACTIVE |

**bookings**

| Column | Type | Notes |
| --- | --- | --- |
| id | BIGINT | Primary key |
| user\_id | BIGINT | FK → users.id |
| resource\_id | BIGINT | FK → resources.id |
| title | VARCHAR(200) | Booking purpose |
| start\_time | TIMESTAMP |  |
| end\_time | TIMESTAMP |  |
| status | VARCHAR(20) | PENDING / CONFIRMED / REJECTED / CANCELLED |
| created\_at | TIMESTAMP |  |

**booking\_approvals**

| Column | Type | Notes |
| --- | --- | --- |
| id | BIGINT | Primary key |
| booking\_id | BIGINT | FK → bookings.id |
| manager\_id | BIGINT | FK → users.id |
| status | VARCHAR(20) | PENDING / APPROVED / REJECTED |
| comment | TEXT | Manager's comment |

## 6. Development plan and status

Work is split into five phases; update the Status column as each phase moves.

| Phase | Scope |
| --- | --- |
| 1. User management | Registration, login, JWT authentication, role-based access control |
| 2. Resource management | Add and update resources, view available resources, manage resource status |
| 3. Booking management | Create, modify and cancel bookings, availability check, overlap prevention |
| 4. Approval management | Manager approve/reject, booking status updates, team booking visibility |
| 5. Additional features | Email notifications, reminders, reports and analytics, maintenance management |

## 7. Work log

| Date | Work done |
| --- | --- |
| 2026-11-16 | Understand the project and wrote project documentation from the presentation |
