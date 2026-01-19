# Glossary – Tipeee Live Events Documentation

**Document type:** Terminology reference  
**Audience:** Developers, integrators, maintainers  
**Status:** Canonical for this repository  
**Last update:** 2026-01-19

---

## Purpose

This glossary defines the **exact meaning of terms** as they are used throughout
this repository.

Its goal is to:
- Remove ambiguity
- Prevent incorrect assumptions
- Ensure consistent interpretation across documents

When in doubt, **this glossary takes precedence**.

---

## Term Definitions

### Event (Socket.IO Event)
A message emitted by the Socket.IO server and delivered to the client as an
application-level notification.

In this repository:
- All relevant events use the Socket.IO event name `"new-event"`
- Different behaviors are encoded in the payload, not the event name

---

### Payload
The JSON object carried inside a Socket.IO event.

In Tipeee’s case:
- The payload is an object containing an `event` property
- All meaningful data is nested under this property

---

### Raw Payload
The **exact payload received from Tipeee**, after transport decoding but before
any transformation, normalization, or filtering.

Properties:
- Must be preserved verbatim
- Used for debugging and forward compatibility
- Stored under the `raw` field in the normalized model

---

### Normalized Event
A consumer-facing representation of a Tipeee event, derived from the raw payload.

Characteristics:
- Stable field names
- Flattened structure
- Explicit semantics
- Includes the raw payload for traceability

Defined in: `docs/specification.md`

---

### Replay Event
An event that represents a **replayed alert**, manually triggered from the
Tipeee dashboard.

Characteristics:
- Uses the same Socket.IO event name as live events
- Identified by `event.is_event_replay = true`
- Structurally identical to live donation events

---

### Live Event
An event corresponding to an actual user action occurring in real time
(e.g., a donation during a stream).

Characteristics:
- `event.is_event_replay = false` or missing
- Otherwise identical in structure to replay events

---

### Donation
A financial contribution made by a user to a Tipeee project.

In this repository:
- Donation events are the primary observed event type
- Identified by `event.type = "donation"`

---

### Donation Type
A raw classification string provided by Tipeee, exposed as:

```
event.donation_type
```

Examples:
- `DIRECT`
- `DIRECT_MONTH`

Important:
- This value reflects **internal or contextual classification**
- It does NOT guarantee billing behavior

---

### Campaign
The configuration context of a Tipeee project under which a donation is made.

Examples:
- Monthly campaign
- One-time campaign

Campaign properties are found under:
```
event.project.parameters
```

---

### Campaign Cadence
The temporal structure suggested by a campaign configuration.

Examples:
- Monthly (`per_month`)
- Yearly (`per_year`)

Important:
- Campaign cadence does NOT imply recurring billing
- A one-time donation may still occur in a monthly campaign

---

### Recurrence
The concept of **automatic future billing** beyond the current donation.

In this repository:
- Recurrence is treated as **unknown by default**
- No reliable detection is currently possible from observed payloads

Allowed normalized values:
- `unknown`
- `one_shot`
- `recurring`

Defined in: `docs/specification.md`

---

### Cadence Hint
A derived, non-authoritative indicator suggesting a time-based context
(e.g., monthly or yearly).

Properties:
- Derived from `donation_type` or campaign configuration
- Informational only
- Must not drive billing logic

---

### Engine.IO
A lower-level transport protocol used by Socket.IO.

Responsibilities:
- Connection management
- Heartbeats (ping/pong)
- Framing of messages

Engine.IO frames are **not raw JSON**.

---

### Socket.IO
A real-time communication library layered on top of Engine.IO.

Responsibilities:
- Namespaces
- Events
- Event payload delivery

In this project:
- All application-level data is delivered via Socket.IO events

---

### Frame
A single transport-level message received over the WebSocket connection.

Examples:
- `"2"` → Engine.IO ping
- `"42[...]”` → Socket.IO event frame

Frames must be parsed before payload extraction.

---

### Ping / Pong
A heartbeat mechanism used by Engine.IO.

Behavior:
- Server sends ping (`"2"`)
- Client must reply with pong (`"3"`)

Failure to respond results in disconnects.

---

### Framing
The structure and prefixing applied to messages at the Engine.IO / Socket.IO level.

Example:
```
42["new-event", {...}]
```

Framing errors are the most common cause of silent failures in native clients.

---

### Native Client
Any client implementation that does NOT rely on the official
JavaScript `socket.io-client`.

Examples:
- C#
- C++
- Python
- Go
- Rust

Native clients must implement framing and keep-alive logic explicitly.

---

### Official Dashboard
The Tipeee web interface used by streamers to manage alerts and campaigns.

It uses the official JavaScript Socket.IO client and therefore hides transport complexity.

---

## Final Note

This glossary is intentionally strict.

If a concept is not defined here, it should be considered:
- ambiguous
- implementation-specific
- or currently unknown

Such cases should be documented explicitly if and when they arise.
