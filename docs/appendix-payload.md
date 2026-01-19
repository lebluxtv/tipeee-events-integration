# Appendix – Observed Tipeee Payloads

**Document type:** Empirical appendix  
**Status:** Observational / Non-normative  
**Purpose:** Illustrate real payloads observed on the wire  
**Last update:** 2026-01-19

---

## 1. Purpose of This Appendix

This document contains **raw and sanitized examples** of payloads observed
from the Tipeee Socket.IO event stream.

It exists to:
- Provide transparency
- Support debugging and validation
- Illustrate real-world field presence and nesting

⚠️ **This document is NOT normative**

- Field presence is not guaranteed
- Examples do not define a contract
- Consumers must rely on `specification.md` for behavior

---

## 2. General Notes

- All examples originate from the same Socket.IO event:
  ```
  "new-event"
  ```
- Live events and replay events share the same structure
- Differences are expressed via payload flags
- Sensitive identifiers have been sanitized

---

## 3. Common High-Level Structure

```json
{
  "event": {
    "...": "..."
  }
}
```

All meaningful data is nested under the `event` key.

---

## 4. Example A – Live Donation Event (Sanitized)

```json
{
  "event": {
    "type": "donation",
    "donation_type": "DIRECT_MONTH",
    "display": true,
    "simulation": false,
    "is_event_replay": false,
    "private_tips": false,

    "id": 2277993,
    "created_at": "2026-01-14T10:54:34+01:00",

    "parameters": {
      "amount": 1,
      "currency": "EUR",
      "message": "tip test",
      "formattedMessage": "tip test",
      "username": "Holdus",
      "publicMessage": false
    },

    "project": {
      "id": 353397,
      "slug": "lebluxtv",
      "status": "OPEN",

      "translations": {
        "fr": { "name": "LeBluxTV" }
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

## 5. Example B – Replay Donation Event (Dashboard Relaunch)

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

## 6. Key Observations (Empirical)

- Replay events:
  - Use the same event name
  - Use the same payload structure
  - Differ only by `is_event_replay = true`

- Donation cadence indicators:
  - `donation_type = DIRECT_MONTH`
  - `project.parameters.campaign_type = per_month`

These fields **do not guarantee recurring billing**.

---

## 7. Field Volatility

Observed variability:
- Some boolean flags may be missing
- `project.parameters` may contain dozens of unrelated config flags
- Numeric values may appear as strings in some cases
- Additional fields may appear without notice

---

## 8. Sanitization Policy

The following have been sanitized or removed:
- API keys
- Media paths
- User-identifying metadata beyond display name
- Internal configuration fields not relevant to event semantics

---

## 9. How to Use This Appendix

Use this document to:
- Compare your received payloads
- Validate parsing logic
- Debug unexpected field absence or nesting

Do **not**:
- Hard-code logic based on appendix examples
- Assume fields are always present
- Infer billing behavior from these samples

---

## 10. Relation to Other Documents

- Contract & behavior → `specification.md`
- Transport framing → `compatibility.md`
- Terminology → `glossary.md`
