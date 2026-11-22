# UniBus Live — A Real-Time University Transport Platform

**Project type:** Final-year software engineering project · **Institution:** NPI University of Bangladesh · **Year:** 2026  
**Role:** Application development across web, API and mobile workflows  
**Stack:** Next.js · React · React Native · Node.js · Express · TypeScript · PostgreSQL · Prisma · Redis · Socket.IO  
**Availability:** Previously VPS-hosted; the backend is currently offline. **No verified public interactive demo.**

[← Return to portfolio](../README.md)

---

## Executive overview

A conventional bus schedule describes the *plan*; it does not answer the question a waiting passenger actually has: **Where is my bus, which trip is active, and how reliable is the arrival estimate?**

UniBus Live was developed as a coordinated transport system rather than a standalone map. It connects the passenger experience, driver trip activity and transport administration around a common set of trip, route and location records.

The application combines three distinct interfaces:

| Audience | Core work | What the interface must explain |
| --- | --- | --- |
| Passengers | Live trips, bus locations, stop information and ETA | Which location and trip data can be trusted |
| Drivers | Start/manage trip activity and send mobile location data | What trip is active and whether updates are accepted |
| Administrators | Vehicles, routes, stops, schedules, accounts and operations | Whether live operations match planned service |

**Evidence basis:** Features and architecture described here have been cross-checked against the private frontend/backend repository structure, Prisma schema, backend service modules, test filenames and the documented Socket.IO protocol. This does **not** imply every feature was independently exercised against a running production system.

## 01 / Product and engineering challenge

The difficult part of bus tracking is not displaying a marker. A real location is only meaningful when it is tied to the correct vehicle, trip, source and time.

Several linked problems follow:

1. **Trip identity:** a bus can have a schedule and still not have an active, passenger-trackable trip.
2. **Location reliability:** driver-phone GPS and installed GPS-device feeds can be missing, stale or contradictory.
3. **ETA uncertainty:** geographic distance alone cannot guarantee an arrival time.
4. **Concurrent actions:** duplicated trip-start requests and delayed network responses can corrupt operational state without safeguards.
5. **Different audiences:** an administrator needs operational control; a passenger needs a simple, understandable live status.

These constraints shaped the application's backend more strongly than the appearance of the map itself.

## 02 / Application surfaces

### Passenger experience

The repository includes a passenger live-tracking page and API controllers for passenger trips, tracking, arrivals and trip details. The interface is intended to connect the currently active trip to its route stops, reported position and ETA information.

### Driver experience

Driver trip functionality connects start/end actions and location updates with the trip's lifecycle. The code also includes pre-trip phases, reflecting the distinction between a bus preparing for service and one already carrying out a running trip.

### Administration

The web dashboard contains dedicated surfaces for buses, routes, stops, route-stop ordering, service schedules, operations, analytics, users, GPS devices, delay reporting and audit logs.

This separation is intentional: transport staff must manage the underlying records, while passengers should not need to understand the administrative model.

## 03 / Architecture

```text
     Driver phone                GPS-device integration
          │                             │
          └──────── location observations ────────┐
                                                  ▼
             Next.js admin / passenger UI   Express + TypeScript API
                           │                      │
                           │               Authentication / roles
                           │               Trip lifecycle / checks
                           │               Tracking source selection
                           │                      │
                           │             ┌────────┴────────┐
                           │             ▼                 ▼
                           │           Redis         PostgreSQL
                           │      transient state   persistent records
                           │             │              Prisma
                           │             └────────┬────────┘
                           │                      ▼
                           └──────────── Socket.IO events
                                                  │
                                       Live trip + ETA views
```

*Conceptual diagram assembled from repository modules; it does not reveal deployment topology or internal service addresses.*

**Why these components?** PostgreSQL stores structured persistent entities; Prisma makes those entities accessible through typed queries. Redis handles short-lived tracking state and coordination. Socket.IO distributes live trip events. Separating durable business records from rapidly changing positions prevents the persistent data model from becoming merely a stream of GPS coordinates.

## 04 / Key implementation decisions

### A. Trip state is a first-class domain concept

The schema defines planned, pre-trip, running and ended trip states. The backend includes trip-start logic, lifecycle services, pre-trip promotion, operational audit records and event emission.

This allows a bus to be physically moving without incorrectly claiming that a passenger service is running.

**Engineering consequence:** live tracking should read a canonical active-trip state, not infer service availability from the latest coordinate alone.

### B. Location-source arbitration

The implementation distinguishes **driver-mobile** and **GPS-device** tracking inputs. The data model records source status and reasons for selection, including preference and fallback scenarios.

The repository has a dedicated `sourceArbitration.service.ts` alongside `gpsIngest.service.ts`, `driverLocation.service.ts`, `trackingFilter.service.ts` and real-time state services.

This matters because two feeds do not automatically mean better data. The system needs rules for choosing a usable location and recognizing when neither feed is healthy. A recorded “no healthy source” state is better than silently displaying a misleading position.

### C. ETA estimation with explicit uncertainty

The ETA implementation works with ordered stops and geographic calculations. Its result model includes nearest stop, next stop, per-stop estimates, observed/used speed, confidence level and a final-stop indicator.

It can return **no ETA** rather than inventing certainty when the observations are inadequate.

This is a deliberate product decision: an uncertain prediction should be represented as uncertain to the passenger.

### D. Real-time delivery versus persistence

The backend's documented Socket.IO protocol includes authenticated connections and trip-specific rooms. Trip events are scoped to the relevant session/trip context.

Transient position state and persistent trip history have different responsibilities. Real-time delivery improves freshness; durable records support administration and accountability.

### E. Duplicate requests and session security

The source contains request-idempotency and trip-lock services, session revocation logic, authorization checks, OTP-related flows and API-rate-limit tests.

These are especially relevant to mobile connectivity, where retries are ordinary rather than exceptional.

## 05 / Concrete source evidence

The following paths are **evidence labels from the private development repositories**, not public source links:

| Concern | Repository evidence |
| --- | --- |
| Trip lifecycle | `src/services/trip.service.ts`, `tripLifecycle.service.ts`, `preTripPromotion.service.ts` |
| Source selection | `sourceArbitration.service.ts`, `trackingFilter.service.ts`, `gpsIngest.service.ts` |
| ETA | `eta.service.ts`, `etaPredictionLog.service.ts` |
| Real-time contracts | `docs/socket-events.md`, `src/sockets/` |
| Data model | `prisma/schema.prisma`, migration history |
| Operational UI | Frontend pages for admin operations, routes, schedules and passenger live tracking |
| Automated checks | Tests for authentication, sessions, GPS-device keys, trip flows, sockets and passenger privacy |

The evidence proves that the implementation and tests exist in the inspected repository. It is not a substitute for publishing CI results or executing an end-to-end demo.

## 06 / Verification and demonstration status

The project had Android builds pilot-tested and was presented at a university project fair. Those events support a claim of development and demonstration, **not** a claim of continuous institutional deployment or measured operational success.

The earlier VPS has expired. Consequently, this portfolio deliberately does not show an obsolete URL as a “Live Demo.”

A safe public demo would be a **separate environment** with synthetic buses, routes, trips and moving positions. It would also need demo-only credentials, independent data storage, protection against abuse and an explicit label distinguishing simulation from real operations. This is a plan, not a deployed feature.

## 07 / What I learned / trade-offs

- A credible transport interface begins with correctly modeled trips and source validity, not map animation.
- Mobile retry behavior makes idempotency and lock handling important.
- ETA needs an honest “not enough information” state.
- Real-time state and long-term business records require different storage and delivery patterns.
- A convincing portfolio demo must preserve the actual interface while isolating all operational dependencies.

## 08 / What a recruiter can evaluate

This work demonstrates domain modeling, full-stack coordination, API boundaries, event-driven delivery, relational data design and engineering under unreliable connectivity.

**What is not claimed:** published performance benchmarks, university-wide production adoption, a currently functioning live link, or public access to the latest private source.

---

*Portfolio case study derived from repository evidence. No private credentials, user records, deployment addresses or operational secrets are published.*