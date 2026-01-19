# Payload (ce qu’on reçoit)

**Langue :** [EN](payload.md) | FR

But : garder une **référence terrain** de ce qu’on a vraiment vu passer.
À utiliser pour comparer vite avec tes logs.

## Structure générale

Après décodage Socket.IO, le payload ressemble à :

```json
{ "event": { ... } }
```

Champs utiles qu’on a vus de façon consistante :
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

## Notes (sans bullshit)

### Replay ≠ nouveau type d’event
C’est le même event Socket.IO (`new-event`), même structure, juste un flag.

### `DIRECT_MONTH` ≠ don récurrent garanti
On a déjà vu des dons ponctuels avec `DIRECT_MONTH`.
À ce stade, on **ne sait pas** déduire un “vrai recurring” de façon fiable.
Donc : ne base pas une logique “recurring” uniquement sur `donation_type` / campagne.
