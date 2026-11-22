# UniBus Live
### A real-time university transport system

**Role:** Final-year software project · **Year:** 2026 · **Institution:** NPI University of Bangladesh  
**Scope:** Transport administration, driver workflows, passenger tracking, mobile apps, API and real-time services  
**Status:** Developed and pilot-tested; the previous VPS-hosted backend is no longer online. This is a case study, **not** a functioning live demo.

[← Portfolio](../README.md)

---

## 01 / The problem

A timetable tells students *when a bus is scheduled*, not where it actually is. A university transport service also needs a consistent way to manage vehicles, route stops, trips and changes in operating conditions.

UniBus Live explores this as one connected system, rather than treating passenger maps, driver updates and administrative scheduling as separate applications.

## 02 / The product

| User | Workflow | Why it matters |
| --- | --- | --- |
| **Passenger** | View live trips, bus locations, route stops and estimated arrival information | Makes waiting and trip planning more informed |
| **Driver** | Work through trip states and submit location updates from the mobile experience | Connects field operations to the system's trip record |
| **Administrator** | Manage buses, routes, stops, schedules, users and operational activity | Keeps the transport model consistent across clients |

The web application includes administrative pages for fleet and route management, service schedules, operations, analytics and audit logs. The passenger side includes live tracking. Driver and passenger mobile workflows were also developed.

## 03 / Architecture and data flow

```text
Driver mobile / supported location source
            │ GPS observations
            ▼
     Express / TypeScript API
            │
            ├── Trip lifecycle and validation
            ├── Real-time state / buffering (Redis)
            ├── Persistent records (PostgreSQL + Prisma)
            └── Real-time events (Socket.IO)
                            │
               Passenger live map / ETA
                            │
                Admin operations view
```

This is a conceptual map of the implemented components, not a production infrastructure diagram.

**Primary technologies:** Next.js, React, TypeScript; Node.js and Express; PostgreSQL and Prisma; Redis; Socket.IO; React Native.

## 04 / Engineering decisions

**Trip state instead of raw coordinates alone.** A bus location without a valid trip is ambiguous. The backend models planned, pre-trip, running and ended states, with corresponding events and administrative actions.

**Real-time versus durable data.** Redis supports short-lived operational state and buffering, while PostgreSQL provides persistent records. Socket.IO distributes trip updates to subscribed clients. This reduces the temptation to treat every transient GPS position as a permanent business record.

**Estimated arrival rather than a static timetable.** The ETA service uses geographic distance, route-stop ordering and speed observations to estimate stops ahead. The implementation includes confidence levels and explicitly allows an unavailable estimate when the data cannot support one. It is an estimate, not a guarantee of arrival time.

**More than one tracking source.** The current data model distinguishes mobile location updates and GPS-device sources and records tracking health and selection reasons. This supports source-aware operational decisions without assuming one feed is always reliable.

**Operational resilience.** The backend includes approaches to idempotent trip requests, locking, audit history and authenticated sessions. These matter because duplicate requests and interrupted connectivity can otherwise distort trip state.

## 05 / Implementation evidence

The private development repositories contain:

- Web pages for buses, stops, routes, service schedules, operations, analytics, driver trips and passenger live tracking.
- A typed backend with controller/service separation, schema migrations and Socket.IO event documentation.
- ETA logic that works with ordered stops, map geometry and speed estimates.
- Trip lifecycle handling, real-time state services, authentication and automated checks.

These are observable repository features; no external performance benchmark or fleet-wide deployment result is claimed.

## 06 / Validation and limitations

Android builds were pilot-tested, and the project was presented at a university project fair. This establishes that the project was demonstrated, **not** that it is running as a university-wide transport service.

The former VPS-hosted backend is offline. An independent interactive demo could use synthetic routes, GPS movements and test accounts, but that demo has **not** yet been published. Old public repositories are not the current source; the latest source remains private.

## 07 / What this work demonstrates

The project required coordinating UI, mobile workflows, API design, real-time events, relational data, trip-state rules and location-driven calculations. The central engineering lesson is that **a useful live map depends on reliable state and data contracts, not merely a moving marker.**

*Public portfolio case study. Private repository contents, credentials and operational endpoints are intentionally excluded.*