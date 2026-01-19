# Tipeee Live Events (Docs non officielles)

**Langue :** [EN](README.md) | FR

Documentation **pratique** sur **ce qu’on a réellement fait et observé** pour recevoir les events live Tipeee :
- dans une **page web (JS)** via `socket.io-client`
- en **C# dans Streamer.bot** (WebSocket + parsing manuel)

Aucune prétention d’API officielle. Juste du concret, reproductible. 
Ce projet documente le comportement observé empiriquement des événements live Tipeee.
Il n’est ni affilié, ni approuvé, ni soutenu par Tipeee.

## Fichiers
- [`payload.fr.md`](payload.fr.md) — exemples de payload observés (live + replay)
- [`docs/js_example.fr.md`](docs/js_example.fr.md) — exemple JS (web)
- [`docs/csharp_example.fr.md`](docs/csharp_example.fr.md) — exemple C# (**testé uniquement dans Streamer.bot**)

## TL;DR
- On écoute l’event Socket.IO : **`new-event`**
- Un replay (relancé depuis le dashboard Tipeee) arrive pareil mais avec :
  - `event.is_event_replay = true`
