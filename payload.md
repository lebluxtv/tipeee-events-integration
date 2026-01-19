# Payload (ce qu’on reçoit)

Ce fichier contient **des exemples réels (sanitisés)** de ce qu’on a vu passer.
But : que tu puisses comparer vite avec tes logs.

## Structure générale

Le payload reçu (après décodage Socket.IO) ressemble à :

```json
{
  "event": { ... }
}
```

Les infos utiles sont surtout dans :
- `event.type`
- `event.is_event_replay`
- `event.parameters` (username, amount, currency, message)
- `event.donation_type`
- `event.created_at`

---

## Exemple A — Donation (LIVE) (sanitisé)

```json
{
  "event": {
    "type": "donation",
    "donation_type": "DIRECT_MONTH",
    "display": true,
    "simulation": false,
    "is_event_replay": false,
    "id": 2277993,
    "created_at": "2026-01-14T10:54:34+01:00",
    "parameters": {
      "amount": 1,
      "currency": "EUR",
      "message": "tip test",
      "formattedMessage": "tip test",
      "username": "Holdus",
      "publicMessage": false
    }
  }
}
```

---

## Exemple B — Donation (REPLAY) (relance depuis dashboard)

Différence principale : `is_event_replay = true`

```json
{
  "event": {
    "type": "donation",
    "donation_type": "DIRECT_MONTH",
    "display": true,
    "simulation": false,
    "is_event_replay": true,
    "id": 2277993,
    "created_at": "2026-01-14T10:54:34+01:00",
    "parameters": {
      "amount": 1,
      "currency": "EUR",
      "message": "tip test",
      "formattedMessage": "tip test",
      "username": "Holdus",
      "publicMessage": false
    }
  }
}
```

---

## Points importants (sans bullshit)

### 1) Replay ≠ nouveau type d’event
C’est **le même event** (`new-event`), même structure, juste un flag.

### 2) “DIRECT_MONTH” ≠ don récurrent garanti
On a vu des dons ponctuels avec `DIRECT_MONTH`.  
À ce stade, on **ne sait pas** détecter de façon fiable un “vrai recurring” avec certitude.
Donc : ne base pas une logique “recurring” uniquement sur `donation_type` ou la config projet.
