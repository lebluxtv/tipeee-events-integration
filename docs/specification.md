# Specification – Tipeee Live Events (Normalized Contract)

**Document type:** Functional specification  
**Status:** Observational / Reverse-engineered  
**Normativity:** This document defines the **expected consumer-facing contract**  
**Last update:** 2026-01-19

---

## 1. Scope

This specification defines the **logical contract** for consuming Tipeee live events
once they have been **successfully received and decoded** at the transport level.

This document intentionally **excludes**:
- Engine.IO / Socket.IO framing
- Transport-level concerns
- Client compatibility issues

➡️ Transport considerations are documented in  
`docs/compatibility.md`

---

## 2. Event Model Overview

### 2.1 Event Source

- **Source platform:** Tipeee
- **Transport:** Socket.IO (over Engine.IO)
- **Observed event name:** `"new-event"`

At the logical level, consumers should assume:
- A **single logical event type**
- Differentiated by payload fields, not by event name

---

## 3. Logical Event Types

Although only one Socket.IO event name is observed, the payload allows
distinguishing multiple **logical cases**.

### 3.1 Donation Event (Live)

A donation event represents a **live user donation** received during a stream.

Identification rules:
- `event.type == "donation"`
- `event.is_event_replay == false` (or missing)

---

### 3.2 Donation Event (Replay)

A replay event represents a **replayed alert** triggered from the Tipeee dashboard.

Identification rules:
- `event.type == "donation"`
- `event.is_event_replay == true`

⚠️ Replay events are **not** separate event types and must not be filtered
by event name.

---

## 4. Normalized Event Contract

Consumers are strongly encouraged to work with a **normalized event model**
rather than raw payloads.

### 4.1 Normalized Event Object (Canonical)

```json
{
  "source": "tipeee",
  "eventName": "new-event",

  "isReplay": false,

  "type": "donation",
  "username": "string",
  "amount": 0.0,
  "currencyCode": "EUR",
  "currencySymbol": "€",
  "message": "string",

  "donationTypeRaw": "string",
  "campaignTypeRaw": "string | null",

  "recurrence": "unknown | one_shot | recurring",
  "cadenceHint": "month | year | null",

  "receivedAt": "ISO-8601 timestamp",
  "raw": { "...full original payload..." }
}
```

---

## 5. Field Semantics

### 5.1 Core Fields

| Field | Type | Description |
|----|----|----|
| `source` | string | Always `"tipeee"` |
| `eventName` | string | Always `"new-event"` |
| `isReplay` | boolean | Replay flag derived from payload |
| `type` | string | Logical type (`"donation"`) |
| `username` | string | Donor display name |
| `amount` | number | Donation amount |
| `currencyCode` | string | ISO currency code |
| `currencySymbol` | string | Display symbol |
| `message` | string | User message (may be empty) |
| `receivedAt` | string | Local reception timestamp |
| `raw` | object | Full unmodified payload |

---

### 5.2 Donation Context Fields

| Field | Type | Meaning |
|----|----|----|
| `donationTypeRaw` | string | Raw `event.donation_type` |
| `campaignTypeRaw` | string | Raw campaign cadence (if present) |
| `cadenceHint` | enum | Derived hint (`month`, `year`, or `null`) |
| `recurrence` | enum | Billing certainty state |

---

## 6. Recurrence Classification (Strict Rules)

### 6.1 Observed Limitation

Current payloads **do not provide sufficient information** to reliably detect
true recurring subscriptions.

Therefore:

- `recurrence = "unknown"` MUST be the default
- `recurrence` MUST NOT be inferred from:
  - campaign configuration
  - donation type naming
  - UI context

### 6.2 Allowed Values

| Value | Meaning |
|----|----|
| `unknown` | No proof of future billing |
| `one_shot` | Explicit proof of single payment |
| `recurring` | Explicit proof of scheduled future billing |

Until subscription-level identifiers are observed, only `unknown` is valid.

---

## 7. Replay Semantics

Replay events:
- Must be processed identically to live events
- Must not be discarded by default
- Are explicitly flagged via `isReplay = true`

Consumers may optionally:
- Ignore replays
- Route them differently
- Use them for testing / automation

---

## 8. Required Consumer Behavior

A compliant consumer **MUST**:

- Accept `"new-event"` as the sole event name
- Inspect payload flags to determine replay status
- Preserve the raw payload
- Avoid assumptions about recurrence
- Tolerate missing or null fields

A compliant consumer **MUST NOT**:

- Infer billing behavior without evidence
- Assume field presence
- Couple logic to project configuration flags

---

## 9. Error Handling & Forward Compatibility

Consumers should assume:
- New fields may appear
- Existing fields may be missing
- Field types may vary (string vs number)

Best practices:
- Null-safe parsing
- Schema-agnostic raw storage
- Minimal hard dependencies

---

## 10. Relation to Other Documents

- Transport & framing rules → `docs/compatibility.md`
- Raw payload examples → `docs/appendix-payload.md`
- Terminology → `docs/glossary.md`
- Common questions → `docs/faq.md`

---

## 11. Status

This specification reflects **observed behavior as of 2026-01-19**  
and will evolve as new evidence (e.g. subscription payloads) becomes available.
