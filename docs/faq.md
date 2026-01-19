# FAQ – Tipeee Live Events Integration

**Document type:** Frequently Asked Questions  
**Audience:** Developers & integrators  
**Status:** Observational / Reverse-engineered  
**Last update:** 2026-01-19

---

## Why does it work in the official dashboard but not in my native client?

Because the official dashboard uses the JavaScript `socket.io-client`, which
automatically handles:

- Engine.IO framing
- Socket.IO packet prefixes
- Ping / pong keep-alive

Native clients must implement these explicitly.  
Most failures are caused by incorrect handling of the `42` Socket.IO event frame
or missing pong replies.

➡️ See: `docs/compatibility.md`

---

## Why do I receive events in JS but nothing in C# / Python / C++?

This is almost always caused by **framing errors**.

Common mistakes:
- Treating the incoming WebSocket message as raw JSON
- Stripping the leading `'4'` from `42[...]`
- Misinterpreting `"2"` as an event instead of an Engine.IO ping
- Failing to reply `"3"` to ping frames

The result is silent event loss.

➡️ See: `docs/compatibility.md`

---

## Why does a replayed alert look exactly like a live donation?

Because **it is the same Socket.IO event**.

- Event name: `"new-event"`
- Same payload structure
- Same nesting

Replay detection is done via a **payload flag**:

```
event.is_event_replay = true
```

There is no separate “replay event”.

---

## Can I safely ignore replay events?

Yes, but only **intentionally**.

Replay events:
- Are explicitly marked
- Are useful for testing, automation, and debugging
- May be triggered manually from the dashboard

If you ignore them, do so by checking `isReplay == true`, not by filtering
event names.

---

## Is there more than one Socket.IO event type?

No.

So far, all donation-related activity arrives as:

```
"new-event"
```

Different behaviors are encoded inside the payload.

Do not wait for other event names.

---

## Can I reliably detect recurring (subscription) donations?

**No – not at this time.**

Observed payloads do **not** contain:
- Subscription identifiers
- Next billing dates
- Explicit recurring objects

Fields such as:
- `donation_type`
- `campaign_type`

describe **campaign context**, not billing guarantees.

Therefore:
- Recurrence MUST default to `"unknown"`

➡️ See: `docs/specification.md`

---

## What does `DIRECT_MONTH` actually mean then?

It indicates that the donation occurred within a **monthly campaign context**.

It does **not** guarantee:
- automatic renewal
- future charges
- an active subscription

A one-time donation can still produce `DIRECT_MONTH`.

---

## Why does my client disconnect after a while?

Likely causes:
- Missing reply to Engine.IO ping (`"2"` → `"3"`)
- Network proxy closing idle sockets
- No reconnection logic

Long-lived clients **must** implement keep-alive handling.

---

## Should I store the raw payload?

Yes. Always.

Reasons:
- Debugging unexpected behavior
- Forward compatibility with new fields
- Verifying assumptions later

The normalized model should always include the raw payload as-is.

---

## Should I rely on project configuration fields?

No.

Fields under:
```
event.project.parameters
```

describe **project-level settings**, not event semantics.

They may change independently of donations and should not be used for
decision-making logic.

---

## Can this integration break in the future?

Yes.

This documentation is based on **observed behavior**, not an official contract.

Potential breaking changes:
- Engine.IO / Socket.IO version upgrades
- New framing behavior
- Payload structure changes
- New donation or subscription models

This is why defensive parsing and raw payload storage are mandatory.

---

## Where should I start if something breaks?

Recommended order:
1. Check Engine.IO framing (`42`, ping/pong)
2. Log raw packets
3. Compare with `appendix-payload.md`
4. Review `compatibility.md`
5. Only then inspect business logic

---

## Is this an official Tipeee API?

No.

This project:
- Is not affiliated with Tipeee
- Does not guarantee stability
- Exists for interoperability and research purposes

Use at your own risk.
