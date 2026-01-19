# Architecture – Tipeee Live Events Integration

**Document type:** System architecture overview  
**Audience:** Developers, integrators, maintainers  
**Status:** Observational / Reverse-engineered  
**Last update:** 2026-01-19

---

## 1. Purpose

This document provides a **high-level architectural view** of the Tipeee live
events integration described in this repository.

It explains:
- How data flows from Tipeee to consumers
- Where responsibilities are split
- Why certain layers exist
- Which concerns are intentionally isolated

This document contains **no code** and **no payload examples**.
Its goal is conceptual clarity.

---

## 2. Architectural Principles

The architecture follows these core principles:

1. **Separation of concerns**
2. **Transport isolation**
3. **Explicit normalization**
4. **Defensive boundaries**
5. **Forward compatibility**

Each layer has a single responsibility and must not leak concerns upward.

---

## 3. High-Level Data Flow

```
┌──────────────────┐
│ Tipeee Platform  │
│ (Dashboard & UI) │
└─────────┬────────┘
          │
          │ Socket.IO (Engine.IO framed)
          ▼
┌────────────────────────┐
│ Transport Layer        │
│ (WebSocket + framing)  │
└─────────┬──────────────┘
          │
          │ Decoded frames (42, ping/pong handled)
          ▼
┌────────────────────────┐
│ Event Decoder          │
│ (Socket.IO EVENT)      │
└─────────┬──────────────┘
          │
          │ Raw payload (JSON)
          ▼
┌────────────────────────┐
│ Normalization Layer    │
│ (Contract mapping)     │
└─────────┬──────────────┘
          │
          │ Normalized event
          ▼
┌────────────────────────┐
│ Consumer Layer         │
│ (Automation / Apps)    │
└────────────────────────┘
```

---

## 4. Layer Responsibilities

### 4.1 Transport Layer

**Responsibility:**
- Establish and maintain the connection
- Handle Engine.IO framing
- Respond to ping / pong
- Reconnect on failure

**Key constraints:**
- Messages are NOT raw JSON
- Incorrect framing silently breaks everything

Defined in:
- `docs/compatibility.md`

---

### 4.2 Event Decoder Layer

**Responsibility:**
- Identify Socket.IO EVENT frames
- Extract event name and payload
- Discard non-application frames

**Inputs:**
- Decoded transport frames

**Outputs:**
- Event name (string)
- Raw payload object

This layer must not:
- Interpret business meaning
- Filter replay vs live
- Modify payload content

---

### 4.3 Normalization Layer

**Responsibility:**
- Transform raw payloads into a stable internal model
- Flatten and rename fields
- Add explicit semantic flags
- Preserve raw payload

**Key outputs:**
- `isReplay`
- `recurrence` (tri-state)
- Canonical field names

Defined in:
- `docs/specification.md`

---

### 4.4 Consumer Layer

**Responsibility:**
- React to normalized events
- Trigger automation
- Display alerts
- Persist data

Consumers must:
- Trust the normalized contract
- Avoid inspecting raw payloads directly for logic
- Handle replay events intentionally

---

## 5. Replay Handling in Architecture

Replay events:
- Follow the same path as live events
- Are not special-cased at the transport level
- Are identified only during normalization

This ensures:
- Uniform handling
- Testability
- Predictable behavior

---

## 6. Recurrence Handling Strategy

Due to lack of subscription-level data:

- Recurrence is classified as `unknown` by default
- No downstream logic should assume recurring billing
- Campaign context is treated as informational only

This avoids:
- False positives
- Incorrect automation triggers
- Financial misinterpretation

---

## 7. Error Containment Strategy

Each layer contains errors locally:

| Layer | Error Type | Containment |
|----|----|----|
| Transport | Disconnects, framing | Reconnect, retry |
| Decoder | Malformed frames | Drop frame, log |
| Normalization | Missing fields | Default values |
| Consumer | Logic errors | App-specific |

No layer should crash the entire pipeline.

---

## 8. Extensibility & Future Changes

This architecture allows for:

- New event types
- New payload fields
- Subscription-related data
- Alternative transports

Without requiring:
- Changes to consumers
- Rewrites of transport logic

---

## 9. Anti-Patterns (Explicitly Avoided)

- Business logic in transport layer
- Payload interpretation in decoder
- Inferring billing behavior from UI context
- Tight coupling to project configuration flags

---

## 10. Relation to Other Documents

- Functional contract → `docs/specification.md`
- Transport details → `docs/compatibility.md`
- Raw payloads → `docs/appendix-payload.md`
- Terminology → `docs/glossary.md`

---

## 11. Summary

This architecture:
- Reflects real-world constraints
- Is resilient to uncertainty
- Separates unstable concerns from stable ones
- Enables safe automation and integration

It is intentionally conservative, favoring correctness and clarity
over assumptions or convenience.
