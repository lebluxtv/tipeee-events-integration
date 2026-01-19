# Compatibility Notes – Tipeee Socket.IO / Engine.IO

**Document type:** Engineering / interoperability notes  
**Status:** Observational / Reverse-engineered  
**Last update:** 2026-01-19

---

## 1. Why This Document Exists

If you use the JavaScript `socket.io-client`, the library hides most transport-level
details and "just works".

If you implement a **native** client (C#, C++, Python, Go, etc.), you must handle
Engine.IO and Socket.IO framing correctly or you will encounter:

- Silent loss of events
- Replays visible in JS but not in native clients
- Random disconnects (keep-alive not implemented)
- Incorrect classification of frames as pings

This document exists to prevent those failures.

---

## 2. Protocol Stack (Critical)

Tipeee uses:

- **Socket.IO** (application events)
- over **Engine.IO** (transport framing)

This means that every message received over the WebSocket is not "raw JSON" but
a framed packet.

---

## 3. Engine.IO Version & Client Compatibility

### 3.1 Observed Version

Observed behavior is consistent with:

- **Engine.IO v3**
- **Socket.IO v2.x** client ecosystem

This implies:

- `socket.io-client v2.x` is compatible
- `socket.io-client v3+` is typically **not compatible** without downgrade / special handling

> If your connection succeeds but you never receive events, suspect a version mismatch.

---

## 4. Transports

Recommended transports:

- Primary: `websocket`
- Fallback: `polling` (if websocket is blocked)

Native implementations commonly use websocket-only; this is acceptable if you can connect,
but be aware that the official stack may fall back to polling.

---

## 5. Mandatory Keep-Alive (Ping / Pong)

### 5.1 Observed Frames

- Server sends Engine.IO **ping**: `"2"`
- Client must reply Engine.IO **pong**: `"3"`

If you do not reply to pings:
- the server will disconnect you
- the disconnect may appear silent depending on your client
- events may stop arriving

### 5.2 Minimal Rule

- On receive `"2"` → send `"3"`

---

## 6. Frame Prefixes (Most Common Failure)

### 6.1 Frame Types (Observed)

| Prefix | Meaning | Layer |
|---|---|---|
| `0{json}` | Handshake payload | Engine.IO |
| `2` | Ping | Engine.IO |
| `3` | Pong | Engine.IO |
| `40` | Namespace connect (usually `/`) | Socket.IO |
| `41` | Namespace disconnect | Socket.IO |
| `42[...]` | **Socket.IO EVENT** | Socket.IO |

---

## 7. The `42` Rule (Critical)

### 7.1 What `42` Means

`42` is not a random string.

It is a two-digit prefix meaning:

- `4` → Engine.IO message carrying Socket.IO data
- `2` → Socket.IO **EVENT** packet type

Therefore:

```
42["new-event", { ... }]
```

is a Socket.IO event named `"new-event"` with a JSON payload.

---

### 7.2 The Classic Bug: Stripping the Leading `4`

A frequent mistake in native clients is to remove the first `'4'` character, e.g.:

```
"42[...]"  → (strip '4') →  "2[...]"
```

This causes catastrophic behavior:

- Your code no longer sees `"42"` events
- The remaining `"2"` is interpreted as Engine.IO ping
- Valid events are silently dropped or mishandled
- You may never log anything, giving the false impression that "no events exist"

✅ **Correct handling**

- Treat `"42"` as an atomic marker for Socket.IO EVENT packets
- Do not mutate or strip the prefix blindly
- Parse based on the full prefix

Minimal safe logic:

- If payload is exactly `"2"` → pong `"3"`
- Else if payload starts with `"42"` → parse as Socket.IO EVENT
- Else → log / ignore safely

---

## 8. Socket.IO Event Parsing (Native Clients)

Given a frame:

```
42["new-event", { ...payload... }]
```

Parsing steps:

1. Remove the `42` prefix
2. Parse the remainder as a JSON array:
   - index 0: event name (string)
   - index 1: payload object (object)

Example remainder:

```json
["new-event", {"event": {...}}]
```

---

## 9. Subscription / Room Mechanics (General Note)

Some Socket.IO servers require an explicit subscribe or room join after connect.

Recommendations:
- Perform subscriptions **after** the namespace connect (`40`)
- Consider a short delay after `connect` before subscribing (tens to hundreds of ms)
- Implement reconnection logic and re-subscribe after reconnect

---

## 10. Recommended Operational Practices

- Automatic reconnection (with backoff)
- Raw packet logging behind a debug flag
- Store last raw payload for debugging
- Detect stalled connections (no events for N seconds → reconnect)

---

## 11. Troubleshooting Quick Checks

### Symptom: “Works in JS, nothing in native”
Likely causes:
- Incorrect framing logic (`42` mishandled)
- Ping/pong not implemented
- Socket.IO / Engine.IO version mismatch
- Subscription sent too early

### Symptom: “Disconnects randomly”
Likely causes:
- Missing pong reply to ping
- Network/proxy closing idle sockets

### Symptom: “Replays show in dashboard but not in native”
Likely causes:
- Same as above; replays arrive as normal events and are dropped by framing mistakes

---

## 12. Status

This document reflects **observed behavior as of 2026-01-19**.
If Engine.IO / Socket.IO versions change on Tipeee's side, this document must be revisited.
