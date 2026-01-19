# Tipeee Live Events – Unofficial Integration Documentation

> **Status:** Reverse-engineered / Observational  
> **Audience:** Developers implementing custom or native clients  
> **Scope:** Transport, payload, and behavioral documentation for Tipeee live events

---

## Purpose

This repository documents the **observed behavior and structure** of live events
emitted by **Tipeee** through its **Socket.IO-based event stream**.

It is intended for developers who need to:
- Consume Tipeee live events outside of the official dashboard
- Implement **native clients** (C#, C++, Python, etc.)
- Build automation pipelines (e.g. Streamer.bot, OBS tooling, custom backends)

This is **not** an official API and does **not** claim long-term stability.

---

## Important Notice (READ FIRST)

⚠️ **Tipeee uses Socket.IO over Engine.IO.**

If you are **not** using the official JavaScript `socket.io-client`,
you **must** correctly handle:
- Engine.IO framing
- Socket.IO event prefixes
- Ping / pong keep-alive

Failure to do so will result in:
- Silent loss of events
- Replays working in JS but not in native clients
- Random disconnects with no explicit error

➡️ **Before implementing a native client, read:**  
[`docs/compatibility.md`](docs/compatibility.md)

---

## What This Repository Is (and Is Not)

### This repository **IS**:
- A **technical reference** based on real observed traffic
- A clear separation between:
  - transport-level constraints
  - payload structure
  - logical event semantics
- A living documentation meant to evolve as new cases are observed

### This repository **IS NOT**:
- An official Tipeee API
- A guarantee of backward compatibility
- A replacement for the official dashboard
- A billing or subscription authority

---

## Documentation Structure

All documentation lives under the `docs/` directory.

```
docs/
├── specification.md      # Functional contract (what to consume)
├── compatibility.md      # Transport & client compatibility (CRITICAL)
├── appendix-payload.md   # Raw observed payloads (empirical)
├── architecture.md       # High-level system overview
├── glossary.md           # Terminology definitions
└── faq.md                # Common pitfalls & questions
```

### Recommended Reading Order

1. **README.md** (this file)
2. [`docs/compatibility.md`](docs/compatibility.md) — Transport & client compatibility
3. [`docs/specification.md`](docs/specification.md) — Functional contract
4. [`docs/appendix-payload.md`](docs/appendix-payload.md) — Raw observed payloads
5. [`docs/architecture.md`](docs/architecture.md) — System overview
6. [`docs/faq.md`](docs/faq.md) — Common questions`
7. [`docs/glossary.md`](docs/glossary.md) — Terminology

---

## Key Observations (Summary)

- All donation-related events arrive as **Socket.IO event `"new-event"`**
- Replay alerts are **not separate events**
  - They are identified via a payload flag
- Monthly campaign context **does not imply recurring billing**
- True subscription / recurring billing **cannot currently be inferred reliably**
- Engine.IO framing mistakes can silently break everything

---

## Design Philosophy

This documentation follows these principles:

- **Strict separation of concerns**
- **Defensive parsing**
- **Explicit uncertainty**
- **No assumptions without evidence**
- **Transparency over convenience**

Where behavior is uncertain, it is documented as such.

---

## Who Should Read This

- Developers building **native Socket.IO clients**
- Tool authors integrating Tipeee into automation systems
- Engineers debugging “works in JS, fails elsewhere” scenarios
- Anyone needing a **clear, honest picture** of how Tipeee events behave

---

## Legal & Disclaimer

This project is provided for **educational and interoperability purposes**.

- No affiliation with Tipeee
- All trademarks belong to their respective owners
- Use at your own risk

---

## Contributing

If you observe:
- new event types
- subscription-related payloads
- breaking changes
- undocumented fields

please open an issue or submit a pull request with:
- sanitized payload
- context of observation
- date and client used

---

## Status

This documentation reflects **observed behavior as of 2026-01-19**  
and will be updated as new evidence emerges.
