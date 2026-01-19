# Exemple C# (Streamer.bot UNIQUEMENT)

**Langue :** [EN](csharp_example.md) | FR

⚠️ **Périmètre**  
Cette approche a été **testée uniquement dans le contexte Streamer.bot** (script C#).  
On ne prétend pas fournir un client Socket.IO C# générique.

## Pourquoi c’est différent du JS
Dans Streamer.bot, tu n’utilises pas `socket.io-client`.  
Donc tu dois gérer ce que JS masque.

### Deux blocages concrets avant “Eureka”
1) Heartbeat Engine.IO : quand le serveur envoie `"2"`, le client doit répondre `"3"`.
2) Les events Socket.IO arrivent dans des frames qui commencent par **`42`** (pas `"2"`).

Si tu ne traites pas `42[...]` correctement, tu peux être “connecté” mais ne rien recevoir.

---

## Ce qu’on fait (résumé pratique)
- Connexion WebSocket à l’endpoint Tipeee
- Pour chaque message reçu :
  - si message == `"2"` → envoyer `"3"` (pong)
  - si message commence par `"42"` → parser l’array JSON :
    - index 0 : nom d’event (ex: `"new-event"`)
    - index 1 : payload (ex: `{ "event": {...} }`)
- Stocker le raw pour debug
- Détecter replay via `event.is_event_replay`
