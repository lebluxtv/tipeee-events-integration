# Exemple JS (web)

**Langue :** [EN](js_example.md) | FR

Contexte : page web (navigateur).  
Le client officiel `socket.io-client` gère les trucs chiants (ping/pong, framing, etc.).

## Code minimal

```js
import io from "socket.io-client";

const socket = io("https://<TIPEEE_ENDPOINT>", {
  transports: ["websocket", "polling"],
});

socket.on("connect", () => {
  console.log("[TIPEEE] connected", socket.id);
});

socket.on("new-event", (payload) => {
  console.log("[TIPEEE] new-event", payload);

  const evt = payload?.event;
  const isReplay = !!evt?.is_event_replay;

  console.log("[TIPEEE] parsed", {
    isReplay,
    type: evt?.type,
    username: evt?.parameters?.username,
    amount: evt?.parameters?.amount,
    currency: evt?.parameters?.currency,
    message: evt?.parameters?.message,
  });
});

socket.on("disconnect", (reason) => console.warn("[TIPEEE] disconnected", reason));
socket.on("connect_error", (err) => console.error("[TIPEEE] connect_error", err));
```

## Ce que JS te masque (important)
En JS, tu n’implémentes généralement pas :
- ping/pong Engine.IO
- framing `42[...]` Socket.IO
- reconnect bas niveau
