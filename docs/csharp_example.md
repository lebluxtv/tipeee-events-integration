# C# example (Streamer.bot ONLY)

⚠️ **Important**  
Ce qui suit a été **testé uniquement dans le contexte Streamer.bot** (script C#).  
On ne prétend pas fournir un client Socket.IO générique C#.

## Pourquoi c’est différent du JS
Dans Streamer.bot, tu n’utilises pas `socket.io-client`.  
Donc tu dois gérer ce que JS masque.

### Les 2 points qui nous ont bloqués avant “Eureka”
1) Engine.IO ping/pong : quand le serveur envoie `"2"`, il faut répondre `"3"`.
2) Les events Socket.IO arrivent en frames **`42[...]`** (et surtout pas `"2"`).

Si tu confonds `42[...]` avec `2`, tu perds tous les events sans le voir.

---

## Ce qu’on fait concrètement (résumé)
- Connexion WebSocket à l’endpoint Tipeee
- À chaque message reçu :
  - si message == `"2"` → envoyer `"3"` (pong)
  - si message commence par `"42"` → parser l’array JSON :
    - index 0 : nom d’event (ex: `"new-event"`)
    - index 1 : payload (ex: `{ "event": {...} }`)
- Stocker le raw (utile debug)
- Déduire `isReplay` depuis `event.is_event_replay`

---

## Parsing minimal (pseudo-code lisible)

```csharp
// msg = string reçu depuis le WebSocket

if (msg == "2") {
    // Engine.IO ping
    Send("3"); // pong
    return;
}

if (msg.StartsWith("42")) {
    // Socket.IO EVENT
    // msg = 42["new-event", { ...payload... }]
    var json = msg.Substring(2);

    // parse json as JArray
    var arr = JArray.Parse(json);

    var eventName = (string)arr[0]; // "new-event"
    var payload = (JObject)arr[1];  // { "event": {...} }

    if (eventName == "new-event") {
        var evt = payload["event"] as JObject;
        var isReplay = (bool?)evt?["is_event_replay"] == true;

        var username = (string?)evt?["parameters"]?["username"];
        var amount = (double?)evt?["parameters"]?["amount"] ?? 0.0;
        var currency = (string?)evt?["parameters"]?["currency"];
        var message = (string?)evt?["parameters"]?["message"];

        // logs + stockage raw
    }
}
```

---

## Replay
Un replay (relance depuis le dashboard) arrive comme un `new-event` normal, avec :
- `event.is_event_replay = true`

Donc : ne filtre pas “par type d’event”, filtre avec ce flag si tu veux les traiter différemment.
