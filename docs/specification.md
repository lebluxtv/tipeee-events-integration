# Tipeee Live Events – Unofficial Technical Specification

## Scope
This document describes how to consume **real-time donation events from tipeee.com**
using the same event stream as the official Tipeee dashboard and widgets.

This specification is **unofficial**, **undocumented by Tipeee**, and provided on a
**best-effort basis**.

---

## Transport & Protocol
- **Technology**: Socket.IO
- **Engine.IO version**: v3 (`EIO=3`)
- **Client requirement**: `socket.io-client@2.x` (mandatory)
- **Endpoint**:
  ```
  https://sso.tipeee.com
  ```
- **Socket path**:
  ```
  /socket.io/
  ```
- **Transports**:
  ```
  websocket, polling (fallback)
  ```

> Socket.IO v3+ clients are **not compatible**.

---

## Authentication
Authentication is performed during the Socket.IO handshake.

- **Query parameter**:
  ```
  access_token = <TIPEEE_API_KEY>
  ```

The API key is provided by Tipeee at the account/widget level and must be treated as a
secret.

---

## Connection Model
- A **single persistent Socket.IO connection** is established.
- Once connected, the client **passively listens** for server-pushed events.
- **No polling** and **no periodic HTTP requests** are required.

This is a pure **event-driven (push)** model.

---

## Subscription (Required)
After the `connect` event, the client must subscribe to a specific project.

- **Event name**:
  ```
  statistic-user
  ```
- **Payload**:
  ```json
  {
    "user": { "username": "<project_slug>" },
    "usage": "DASHBOARD" | "ALERT" | "WIDGET"
  }
  ```

`usage = "DASHBOARD"` mirrors the behavior of the official Tipeee dashboard and is
recommended.

> In practice, emitting this event shortly after connection (small delay) improves
> reliability.

---

## Incoming Events
- **Primary event**:
  ```
  new-event
  ```

- The payload:
  - is JSON
  - is **not versioned**
  - may change structure
  - may be nested or partially wrapped

Consumers **must implement defensive parsing**.

---

## Recommended Normalized Event Model
For downstream systems (e.g. Streamer.bot), the following normalized structure is
recommended:

```json
{
  "source": "tipeee",
  "type": "donation",
  "username": "string",
  "amount": number,
  "currencyCode": "EUR",
  "currencySymbol": "€",
  "message": "string",
  "raw": { "...original payload..." }
}
```

The `raw` field should always be preserved for forward compatibility.

---

## Reliability Considerations
- Automatic reconnection is required.
- Silent disconnects may occur.
- Exponential backoff is recommended.
- Socket.IO heartbeat/ping-pong handles connection liveness.

---

## Security Considerations
- Never log or expose the API key.
- Treat all incoming payloads as untrusted input.
- Avoid assuming payload completeness or schema stability.

---

## Stability Disclaimer
- This API is **not officially supported** by Tipeee.
- No versioning or backward-compatibility guarantees exist.
- Breaking changes may occur without notice.

---

## Summary
- Real-time donation events are available via a **persistent Socket.IO connection**
- No REST polling or scraping is involved
- The integration is technically clean but **non-contractual**
- Suitable for best-effort integrations such as Streamer.bot connectors
