# Tipeee Socket.IO Event Payload – Observed Specification
**Version:** 1.0  
**Status:** Observational / Reverse-engineered  
**Last update:** 2026-01-19

---

## 1. Scope & Intent

This document describes the **observed structure and behavior** of events received
from the **Tipeee Socket.IO live event stream**, as used by the Tipeee dashboard
and third-party integrations.

⚠️ **Important notice**

- This is **not an official API contract**
- The structure is **reverse-engineered from real traffic**
- Field presence, naming and nesting **may change without notice**
- All parsing **must be defensive**
- Any behavioral inference (recurring billing, replay, etc.) must be explicitly justified

---

## 2. Transport Layer (Socket.IO / Engine.IO)

### 2.1 Event Envelope

Observed Socket.IO frames carrying payloads are **EVENT frames**:

```
42["<eventName>", { ...payload... }]
```

Observed so far:

- `eventName = "new-event"`

No alternative event names have been observed for donations or replays.

---

## 3. Single Raw Payload Format (Important)

There is **ONE raw payload format**.

There are **NOT multiple raw JSON schemas**.

Differences such as:
- live vs replay
- monthly context vs one-shot payment

are expressed via **flags and contextual fields**, not via distinct event types.

---

## 4. Full Observed Payload (Sanitized Example)

```json
{
  "event": {
    "type": "donation",
    "donation_type": "DIRECT_MONTH",
    "display": true,
    "simulation": false,
    "is_event_replay": false,
    "private_tips": false,

    "id": "<event_id>",
    "created_at": "YYYY-MM-DDTHH:MM:SS+TZ",

    "parameters": {
      "amount": 1,
      "currency": "EUR",
      "message": "example message",
      "formattedMessage": "example message",
      "username": "AnonymousUser",
      "publicMessage": false
    },

    "project": {
      "id": "<project_id>",
      "slug": "<project_slug>",
      "status": "OPEN",

      "translations": {
        "fr": { "name": "ExampleProject" }
      },

      "currency": {
        "code": "EUR",
        "symbol": "€",
        "label": "Euro"
      },

      "parameters": {
        "campaign_type": "per_month",
        "disableRecurring": false,
        "recurring_only": false,
        "direct_only": false
      }
    }
  }
}
```

---

## 5. Live Event vs Replay Event

### 5.1 Key Observation

**Replay alerts DO NOT use a different event name.**

Both live events and replayed alerts arrive as:

```
eventName = "new-event"
```

### 5.2 Replay Detection (Authoritative)

Replay detection is based **solely** on the payload flag:

```
event.is_event_replay = true
```

| Case | is_event_replay |
|----|----|
| Live donation | `false` or missing |
| Replay (dashboard relaunch) | `true` |

---

## 6. Donation Cadence vs True Recurrence (Critical Section)

### 6.1 What the Payload CAN Tell Us

The following fields are observed:

- `event.donation_type` (e.g. `"DIRECT_MONTH"`)
- `event.project.parameters.campaign_type` (e.g. `"per_month"`)

These fields indicate the **campaign or UI context** in which the donation was made.

They **DO NOT prove** that:
- the payment will repeat
- a subscription exists
- a future charge is scheduled

A user can make a **one-time payment** inside a monthly campaign.

### 6.2 What the Payload DOES NOT Contain (So Far)

No observed payload contains:

- `subscription_id`
- `recurring_id`
- `next_payment_at`
- `next_charge_at`
- `subscription` / `recurring` object

Therefore:

> **True recurring billing CANNOT be determined with certainty from current payloads.**

---

## 7. Correct Conceptual Model

### 7.1 Distinguish These Three Concepts

| Concept | Meaning |
|------|------|
| Donation type | Technical classification used internally by Tipeee |
| Campaign cadence | Monthly / yearly context of the project |
| True recurrence | Automatic future billing |

Only the first two are currently observable.

---

## 8. Recommended Normalized Model (Safe & Honest)

Instead of a misleading boolean, use a **tri-state model**.

```json
{
  "source": "tipeee",
  "eventName": "new-event",

  "isReplay": false,

  "type": "donation",
  "username": "AnonymousUser",
  "amount": 1,
  "currencyCode": "EUR",
  "currencySymbol": "€",
  "message": "example message",

  "donationTypeRaw": "DIRECT_MONTH",
  "campaignTypeRaw": "per_month",

  "recurrence": "unknown",
  "cadenceHint": "month",

  "raw": { "...full payload..." }
}
```

---

## 9. Reliable Extraction Paths

| Data | JSON Path |
|----|----|
| Replay flag | `event.is_event_replay` |
| Event id | `event.id` |
| Created at | `event.created_at` |
| Username | `event.parameters.username` |
| Amount | `event.parameters.amount` |
| Currency | `event.parameters.currency` |
| Message | `event.parameters.message` |
| Donation type (raw) | `event.donation_type` |
| Campaign type | `event.project.parameters.campaign_type` |

---

## 10. Parsing Rules (Must-Follow)

- Never assume field presence
- Never assume string vs number consistency
- Never infer recurrence from campaign configuration alone
- Always preserve raw payload
- Treat replay and live events identically except for `is_event_replay`

---

## 11. Summary (TL;DR)

- One Socket.IO event: `"new-event"`
- Replay is a **flag**, not a different event
- Monthly ≠ recurring
- No subscription data observed yet
- Recurrence must remain **unknown** until proven
- Defensive parsing is mandatory

---

## 12. Status

This document reflects **observed real-world behavior as of 2026-01-19**  
and should be updated as soon as a **true recurring billing payload** is captured.
