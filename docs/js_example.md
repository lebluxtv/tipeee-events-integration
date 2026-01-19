# JS example (page web)

Contexte : on est dans une page web (browser).  
Le client officiel `socket.io-client` fait tout le boulot (ping/pong, framing, etc.).

## Code minimal

```js
// Exemple minimal : écoute new-event et log le payload
// (À adapter à ton URL/namespace Tipeee)

import io from "socket.io-client";

const socket = io("https://<TIPEEE_ENDPOINT>", {
  transports: ["websocket", "polling"],
});

socket.on("connect", () => {
  console.log("[TIPEEE] connected", socket.id);
});

socket.on("new-event", (payload) => {
  // payload = { event: {...} }
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

socket.on("disconnect", (reason) => {
  console.warn("[TIPEEE] disconnected", reason);
});

socket.on("connect_error", (err) => {
  console.error("[TIPEEE] connect_error", err);
});
```

## Note
En JS tu ne gères pas :
- ping/pong Engine.IO
- framing `42[...]`
- reconnect bas niveau

Ça explique pourquoi “ça marche en JS” alors qu’en natif ça peut être silencieux.
