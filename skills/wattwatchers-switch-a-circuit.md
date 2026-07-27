---
generated: '2026-07-27'
method: generated
name: Switch a circuit on or off
description: Safely open or close a physical relay on +3SW Wattwatchers hardware and confirm the change actually converged on the device.
api: openapi/wattwatchers-rest-api-v3-openapi.json
operations: [getDevice, updateDevice]
source: >-
  Grounded in openapi/wattwatchers-rest-api-v3-openapi.json (operationIds
  verified verbatim) and
  https://docs.wattwatchers.com.au/api/tips/switching-example.html.
  Conventions from conventions/wattwatchers-conventions.yml.
---

# Switch a circuit on or off

`updateDevice` is the **only write operation in the entire Wattwatchers API**, and one of the things it writes is switch state — which energises or de-energises a real electrical circuit. Treat every call as a safety-relevant action.

## Before you start

- **Confirm with a human.** There is no undo, no dry-run, no sandbox and no test key. A `state: "open"` on the wrong device cuts power to a real load.
- Switching is only available on `+3SW` hardware variants (`6M+3SW`, `6W+3SW`, `3RM+3SW`). It is **not** available on 6M+One devices.
- The API key must have been issued with the configuration-write permission level (introduced in v3.5). Without it you get `403 FORBIDDEN`.

## Auth

- `Authorization: Bearer key_...`. See `authentication/wattwatchers-authentication.yml`.

## Steps

1. **Read the device** — `getDevice` (`GET /devices/{device-id}`). Confirm:
   - the `model` is a `+3SW` variant,
   - a `switches` collection is present (non-switching devices return no switches collection),
   - the current `state` of the switch you intend to change,
   - the `label` of the channel the switch controls, so you can state in plain language what is about to be turned off.
2. **Confirm the intent with the operator**, quoting the device ID, the switch ID (`D123456789012_S1` form, one-indexed) and the human label.
3. **Write the state** — `updateDevice` (`PATCH /devices/{device-id}`) with `Content-Type: application/json` and a body containing only what you intend to change:

   ```json
   {
     "switches": [
       { "id": "D123456789012_S1", "state": "closed" }
     ]
   }
   ```

   You may include other writable fields (`label`, `timezone`, `channels`, `phases`) in the same PATCH. Read-only properties present in the body are silently ignored rather than rejected, so a "retrieve, modify, PATCH" round-trip is safe without pruning.
4. **Read `pending`** — the response is the device record. A configuration change appears under `pending` until the physical device next connects and converges. A 200 means the request was accepted, **not** that the relay has moved.
5. **Verify convergence** — re-issue `getDevice` after the device's next check-in and confirm the live switch `state` matches what you set and that `pending` no longer holds it. Do not report success until this step passes.

## Method notes

- `PUT` is not supported anywhere in this API. Use `PATCH` with all values if you need whole-object semantics.
- There is **no idempotency key**. Repeating the same PATCH converges on the same state, but the vendor offers no dedupe guarantee — do not retry blindly on a network timeout; re-read with `getDevice` first.

## Errors

- `403 FORBIDDEN` — the key lacks the configuration-write permission level, or the device is not assigned to it.
- `404 NOT_FOUND` — unknown device, or a device your key cannot see.
- `400 BAD_REQUEST` — malformed body, wrong `Content-Type`, or a bad serial number ("Serial numbers must be 13 characters in length, starting with a B, D, E, F.").
- **Multi-error responses**: `PATCH /devices/{device-id}` is the one endpoint that returns `{"errors": [ ... ]}` — an array of error objects, always an array even for a single error. The HTTP status returned is the **lowest** status code among them. Parse the array, not just the status.
- Full catalog: `errors/wattwatchers-error-codes.yml`.
