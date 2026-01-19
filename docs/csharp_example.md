# C# example (Streamer.bot ONLY)

**Language:** EN | [FR](csharp_example.fr.md)

⚠️ **Scope note**  
This approach was **tested only inside the Streamer.bot scripting environment**.  
No claim is made that this is a generic C# Socket.IO client.

## Why it differs from JS
In Streamer.bot, you’re not using `socket.io-client`.  
So you must handle what JS normally hides.

### Two concrete blockers we had before “Eureka”
1) Engine.IO heartbeat: when the server sends `"2"`, the client must reply `"3"`.
2) Socket.IO events come in frames starting with **`42`** (not `"2"`).

If you don’t treat `42[...]` correctly, you can stay “connected” but receive no events.

---

## What we do (practical summary)
- Connect a WebSocket to the Tipeee endpoint
- For each received message:
  - if message == `"2"` → send `"3"` (pong)
  - if message starts with `"42"` → parse the JSON array:
    - index 0: event name (e.g. `"new-event"`)
    - index 1: payload (e.g. `{ "event": {...} }`)
- Store raw payload for debugging
- Detect replay via `event.is_event_replay`
