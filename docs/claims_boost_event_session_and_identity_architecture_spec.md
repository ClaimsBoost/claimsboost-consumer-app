# ClaimsBoost Event, Session, and Identity Architecture Spec

## Purpose
This document defines the internal architecture for event tracking, session state, and user identity at ClaimsBoost. It is intentionally staged, allowing the system to start simple and evolve without rewrites.

The architecture is divided into two layers:
- **Part 1: Session-Based System (Identity-Free)** — required from day one
- **Part 2: Identity Resolution & User Profiles** — additive, enabled when the business needs it

At all times, the system prioritizes:
- Correct UX behavior
- Low latency in user-facing flows
- Clear ownership of responsibility between systems
- Future-proofing without premature complexity

---

## Core Principles

1. **Hot path vs background processing**
   - The application must never block on analytics or identity systems
   - All user-facing decisions must be served from a fast, local data store

2. **Separation of concerns**
   - Event history and analytics are separate from application state
   - Identity resolution is separate from UI logic

3. **Explicit over implicit identity**
   - Identity is only established via clear user actions (OTP, login, verification)
   - No probabilistic or heuristic merging

4. **Layered evolution**
   - Part 2 builds on Part 1 without changing its guarantees
   - All data collected in Part 1 remains valid and usable

---

## System Components (Always Present)

### Client Application (SvelteKit)

Responsibilities:
- Generate and persist an **anonymous ID** on first visit
- Attach the anonymous ID to:
  - Every Jitsu event
  - Every Supabase state read/write
- Call `jitsu.identify()` when explicit identity signals occur (Part 2 only)

The client:
- Never performs identity resolution
- Never merges data
- Never queries the data warehouse directly

---

### Supabase (Application State Store)

Supabase is the **hot path** for the product.

It stores:
- Current session state
- Current user state (Part 2)
- Only information required to render the UI and make routing decisions

Supabase does **not** store:
- Event history
- Funnel data
- Analytics aggregates

Supabase guarantees:
- Sub-100ms reads
- Deterministic state
- Simple debugging

---

### Jitsu (Event System of Record)

Jitsu is the **system of record for all events**.

It is responsible for:
- Capturing all meaningful product events
- Streaming events to the data warehouse in real-time
- Retroactive identity stitching (Part 2 only)

Jitsu is:
- Write-only from the application's perspective
- Never queried by the UI
- Never part of the hot path

Key characteristics:
- MIT licensed, fully open source
- Self-hostable or available as managed cloud
- Streams directly to warehouse (Postgres, ClickHouse, BigQuery, Snowflake, Redshift)

---

## Part 1: Session-Based Architecture (Identity-Free)

### Goal

Enable reliable analytics and correct UX behavior without any concept of a persistent user.

At this stage:
- Each session is independent
- Repeat visits are not unified
- Identity does not exist

This is intentional.

---

### Session Concept

A **session** represents a single visit context.

- Generated on first visit
- Stored client-side (Jitsu generates an `anonymous_id` automatically)
- Used as the primary key for state

The anonymous ID is the only identifier in Part 1.

---

### Session State (Supabase)

Supabase stores one row per session, representing what is currently true for that visit.

Examples of session state:
- Phone verified
- Consent given
- Eligibility status
- Settlement estimate
- Assigned law firm
- First-touch UTMs

State is mutable and overwriteable.

---

### Events (Jitsu)

Every meaningful interaction emits an event.

Events:
- Are append-only in the warehouse
- Always include the anonymous ID
- Include contextual data (UTMs, page context, timestamp)

Events answer:
> "What happened?"

State answers:
> "What is true right now?"

---

### Data Flow (Part 1)

1. User performs an action
2. The application:
   - Updates session state in Supabase if required
   - Emits an event to Jitsu via `jitsu.track()`
3. The UI reads state exclusively from Supabase

Jitsu failures do not affect UX.

---

### Geolocation Handling

- IP-to-geo is handled via MaxMind in server-side logic
- Derived geo fields are stored in Supabase
- Raw IPs are not stored long-term unless required
- Geo context may be attached to events before sending to Jitsu

Note: Unlike Snowplow, Jitsu does not have built-in geo enrichment. This is handled application-side via MaxMind, which aligns with our architecture.

---

### What Part 1 Explicitly Avoids

- Identity graphs
- Profile merging
- Stream processing beyond Jitsu's built-in streaming
- Background workers
- Kafka or Pub/Sub
- Cross-session attribution

This keeps the system fast, simple, and debuggable.

---

## Part 2: Identity Resolution & User Profiles (Additive Layer)

### Goal

Introduce a persistent concept of a user while preserving Part 1 guarantees.

Part 2 enables:
- Cross-session continuity
- Multi-device understanding
- Lifetime attribution and analytics

---

### User Concept

A **user** is a durable entity that may be associated with many sessions.

- Users are introduced only when an explicit identity signal occurs
- Sessions continue to exist independently

---

### Identity Signals

Identity is established only through explicit actions:
- OTP verification
- Account creation
- Login
- Verified phone or email

No probabilistic or inferred identity is used.

---

### Jitsu Identity Resolution

When a user identifies themselves, the client calls:

```javascript
jitsu.identify({
  userId: 'user_123',
  email: 'user@example.com',
  phone: '+1234567890'
});
```

Jitsu's retroactive user recognition:
- Automatically updates all previous anonymous events in the warehouse
- Links the `anonymous_id` to the `user_id`
- Happens in real-time (not batch)

This means historical events are amended with user identity as soon as identification occurs.

---

### Infrastructure Requirements (Part 2)

Jitsu's identity stitching requires:
- **Redis**: Caches anonymous events for retroactive updates
- **Warehouse with UPDATE support**: Postgres, Snowflake, Redshift, MySQL, or ClickHouse (ReplacingMergeTree)

Redis RAM consumption scales with anonymous event volume. Plan capacity accordingly.

---

### Profile Sync Process

A lightweight sync process updates Supabase when identity is established:

1. Client calls `jitsu.identify()` with user details
2. Client simultaneously updates Supabase session record with `user_id`
3. Supabase user profile is created or updated
4. Historical session records can be linked to user via `user_id`

This is simpler than a background worker approach because:
- Identity is known at the moment of identification
- The client can update both systems synchronously
- No need to poll or listen for external events

---

### Supabase in Part 2

Supabase now stores:
- Session state (unchanged, now includes optional `user_id` foreign key)
- User profiles (new)

The UI:
- Continues to read from Supabase
- Never queries the warehouse
- Remains unaware of event stitching mechanics

---

### Merge Philosophy

- Jitsu handles event-level identity stitching (anonymous → identified)
- Supabase stores the canonical user profile
- The UI consumes resolved state from Supabase

Merge rules for user profiles are explicit and deterministic:
- "Set once" fields are preserved (e.g., first_touch_utm)
- "Latest wins" fields are updated (e.g., last_seen_at)
- Append-only fields retain history (e.g., sessions array)

Note: Unlike batch-based systems, Jitsu does not maintain a separate merge history log. The warehouse events themselves serve as the audit trail.

---

### Failure Tolerance

If Jitsu or Redis fails:
- Sessions continue to work (Supabase is independent)
- New events may be lost until recovery
- Identity stitching for in-flight anonymous events may not complete

If identification fails:
- Session continues to function anonymously
- User can retry identification
- No data is lost, just not linked

State in Supabase remains authoritative for UX decisions.

---

### Trade-offs vs Snowplow

| Aspect | Jitsu Approach | Snowplow Approach |
|--------|----------------|-------------------|
| Identity timing | Real-time | Batch (dbt runs) |
| Merge history | No dedicated log | User Mapping table |
| Infrastructure | Redis + warehouse | dbt + warehouse |
| Schema enforcement | Flexible (auto-columns) | Strict (Iglu registry) |
| License | MIT (open source) | SLULA (restricted) |
| Enrichments | DIY (MaxMind, etc.) | 50+ built-in |

We accept these trade-offs because:
- Real-time identity resolution improves user experience
- We handle geo enrichment server-side anyway
- MIT license provides long-term flexibility
- Simpler infrastructure aligns with our team size

---

## Evolution Guarantees

This architecture guarantees:
- No rewrites when identity is introduced
- No loss of historical data
- No regression in UX performance
- Warehouse events serve as the audit trail

---

## Summary

- Supabase powers the live product
- Jitsu records event history and streams to the warehouse
- Anonymous IDs are primary in Part 1
- User identity is layered on in Part 2 via `jitsu.identify()`
- Identity never blocks UX
- Complexity is introduced only when it creates business value

This document defines the authoritative mental model for ClaimsBoost's event and identity architecture.
