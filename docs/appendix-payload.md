## Appendix A – Observed Live Event Payload (Full Sanitized Example)

This appendix provides a **comprehensive, sanitized example** of a real live event
payload received from the Tipeee Socket.IO event stream.

The structure reflects **actual observed payloads** while anonymizing sensitive or
account-specific data.

⚠️ **Important**
- This payload is provided for **illustration purposes only**
- It does **not** represent a stable or versioned API contract
- Field presence, naming, and structure may change without notice
- Consumers must implement **defensive parsing**

---

### Full Payload Example (Sanitized)

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
      "name": "",

      "translations": {
        "fr": {
          "name": "ExampleProject"
        }
      },

      "avatar": {
        "id": "<media_id>",
        "name": "<filename>",
        "type": "image",
        "mime_type": "image/png",
        "original_filename": "<original_filename>",
        "filename": "<stored_filename>",
        "path": "<storage_path>",
        "size": 0,
        "updated_at": "YYYY-MM-DDTHH:MM:SS+TZ"
      },

      "currency": {
        "code": "EUR",
        "symbol": "€",
        "label": "Euro"
      },

      "parameters": {
        "notification_tip": "1",
        "hidedAmount": false,
        "tipperAmount": "1",
        "tipperNumber": "1",

        "campaign_name": "monthly",
        "campaign_type": "per_month",

        "adult_content": false,
        "enabled_streaming": "YYYY-MM-DD HH:MM:SS",

        "disableRecurring": false,
        "recurring_only": false,
        "direct_only": false,

        "banned": false,
        "moderationVerified": false,
        "moderationNsfw": false,
        "moderationReported": false,

        "module_countdown_activated": false,
        "module_countdown_title": false,
        "module_countdown_description": false,
        "module_countdown_ending_time": false,
        "module_countdown_icon_id": false,
        "module_countdown_image_id": false,
        "module_countdown_reward_id": false,
        "module_countdown_link": false,

        "migration_status": "2",
        "translations_enabled": false,
        "discussion_activated": false
      }
    }
  }
}
```

---

### Parsing Guidance

The payload contains **significantly more data than required** for most integrations.

Recommended extraction points:

| Field | Path |
|------|------|
| Username | `event.parameters.username` |
| Amount | `event.parameters.amount` |
| Currency code | `event.parameters.currency` |
| Message | `event.parameters.message` |
| Project slug | `event.project.slug` |
| Currency symbol | `event.project.currency.symbol` |

All other fields should be considered **contextual metadata**.

---

### Observations

- Payloads are **deeply nested**
- Many fields are configuration flags unrelated to the donation itself
- Boolean and string-encoded numeric values may coexist
- New fields may appear without notice
- Some fields may be absent depending on:
  - donation type
  - project configuration
  - replay vs live events

---

### Recommended Handling Strategy

- Never assume field presence
- Never assume fixed nesting depth
- Avoid hard dependencies on project configuration fields
- Preserve the **raw payload** for debugging and forward compatibility
- Map only the minimal required data to internal models

---

### Relation to Normalized Model

The above payload can be normalized into the following simplified structure:

```json
{
  "source": "tipeee",
  "type": "donation",
  "username": "AnonymousUser",
  "amount": 1,
  "currencyCode": "EUR",
  "currencySymbol": "€",
  "message": "example message",
  "raw": { "...full payload..." }
}
```

---

### Disclaimer

This appendix reflects **observed behavior of the Tipeee frontend** at a given point
in time. It does not imply any guarantee of stability, completeness, or long-term
availability of the described structure.
