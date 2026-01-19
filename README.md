# Tipeee Live Events (Unofficial docs)

Doc simple et pratique sur **ce qu’on a réellement observé** en recevant les events live Tipeee :
- dans une **page web JS** (socket.io-client)
- en **C# dans Streamer.bot** (script + WebSocket)

Aucune promesse “API officielle”. Juste du concret.

## Fichiers
- [`payload.md`](payload.md) : exemples de payloads reçus (live + replay)
- [`docs/js_example.md`](docs/js_example.md) : exemple JS (web)
- [`docs/csharp_example.md`](docs/csharp_example.md) : exemple C# **testé uniquement dans Streamer.bot**

## TL;DR
- On écoute l’event Socket.IO : **`new-event`**
- Un **replay** (relance depuis le dashboard) arrive pareil qu’un event live, mais avec :
  - `event.is_event_replay = true`
