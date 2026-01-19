# Tipeee Live Events (Unofficial docs)

**Language:** EN | [FR](README.fr.md)

Practical documentation of **what we actually implemented and observed** to receive Tipeee live events:
- in a **web page (JS)** using `socket.io-client`
- in **C# inside Streamer.bot** (WebSocket + manual parsing)

No “official API” claims. Just working, reproducible stuff.

## Files
- [`payload.md`](payload.md) — observed payload examples (live + replay)
- [`docs/js_example.md`](docs/js_example.md) — JS example (web)
- [`docs/csharp_example.md`](docs/csharp_example.md) — C# example (**tested only in Streamer.bot**)

## TL;DR
- We listen to the Socket.IO event: **`new-event`**
- A replay (triggered from Tipeee dashboard) looks the same but has:
  - `event.is_event_replay = true`
