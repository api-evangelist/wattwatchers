---
generated: '2026-07-27'
method: generated
name: Continuously sync a fleet's energy data
description: Run a robust, watermark-based sync of Long Energy data for an entire device fleet that survives device comms outages and back-fill.
api: openapi/wattwatchers-rest-api-v3-openapi.json
operations: [listDevices, getDevice, getLatestLongEnergyData, getLongEnergyData]
source: >-
  Grounded in openapi/wattwatchers-rest-api-v3-openapi.json (operationIds
  verified verbatim) and the vendor's own polling guidance at
  https://docs.wattwatchers.com.au/api/tips/polling-data.html and
  https://docs.wattwatchers.com.au/api/tips/device-catch-up.html
---

# Continuously sync a fleet's energy data

Wattwatchers ships **no webhooks, no MQTT, no WebSocket and no server push** — a stream API is described in the docs as something they are "exploring". Every integration polls. This skill implements the pattern the vendor themselves recommend, which is the only one that does not silently lose data.

## Why the naive approach fails

Auditor devices are IoT hardware on 4G or home WiFi. When connectivity drops, a device keeps recording locally and back-fills when it reconnects — catch-up after a long outage can take **up to about 6 hours**. A job that asks for "the last hour, every hour" will therefore return incomplete data with no error, and you will never know. Rolling a wider window (24 hours) reduces but does not remove the problem.

## The pattern: per-device watermarks

Maintain, per device, the timestamp of the last Long Energy interval you have durably stored. Sync forward from that watermark rather than from wall-clock time.

## Auth

- `Authorization: Bearer key_...`. See `authentication/wattwatchers-authentication.yml`.

## Steps

1. **Enumerate the fleet** — `listDevices` (`GET /devices`). Unpaginated array of every device ID on the key. Refresh this periodically; devices are added and removed by Wattwatchers, not by you.
2. **Cache the channel layout** — `getDevice` (`GET /devices/{device-id}`) per device. Store the ordered `channels[]` with labels, CT ratings and categories. Energy arrays are positional against this order, so a stale cache silently mislabels data. Re-fetch whenever a device's configuration may have changed (a `pending` value you observed has since cleared).
3. **Find the head** — `getLatestLongEnergyData` (`GET /long-energy/{device-id}/latest`) with `fields[energy]=timestamp`. This is a cached, cheap call and tells you whether the device has anything newer than your watermark. Skip devices with nothing new.
4. **Fetch forward in chunks** — `getLongEnergyData` (`GET /long-energy/{device-id}`) with `fromTs` = your watermark and `toTs` = min(watermark + 7 days, head + 1). The 7-day cap is hard: a longer window returns `422`, not a partial result. Loop until you reach the head.
5. **Re-read the trailing window** — do not advance the watermark all the way to the head immediately. Keep re-requesting the most recent period (the vendor notes that data for an already-reported interval can *change* when a device catches up, particularly at granularities coarser than 5m). Advance the durable watermark only for intervals older than your chosen settle period.
6. **Reconcile gaps** — record which intervals arrived. A gap that persists past the settle period is either a genuine device outage or an incomplete back-fill; re-query that specific range rather than widening every poll.

## Pacing against rate limits

- TPS and TPD both **auto-scale with the number of devices on your key**: TPD ≈ devices × (288 + 2880 + 288) plus a buffer, TPS ≈ devices ÷ 3. A 100-device fleet gets roughly 345,600 requests/day and 33 requests/second.
- Read `X-RateLimit-TpsRemaining` and `X-RateLimit-TpdRemaining` on every response and throttle from the live values, not from an assumption.
- Estimate safe concurrency as TPS × average response time — at TPS 33 and 100ms average, roughly 3 concurrent workers polling 10×/second.
- On `429`, sleep for the integer `Retry-After` seconds. `TOO_MANY_REQUESTS_TPD` means you are done for the UTC day; `TOO_MANY_REQUESTS_TPS` is transient.
- A `429` **without** `X-RateLimit-*` or `Retry-After` headers is platform-wide edge throttling, not your key's limit — back off harder and expect a vendor-side incident.
- See `rate-limits/wattwatchers-rate-limits.yml`.

## Errors to handle as normal operation

- `204` — device has never reported Long Energy. Skip it; retry on a slow cadence in case it is commissioned later.
- `200` with `[]` — no data in this window. Advance past it only if the window is older than the settle period.
- `422` — your window exceeded 7 days, or a timestamp was invalid. Fix the chunking; do not retry unchanged.
- `404` — the device left your key's assignment. Drop it from the fleet on the next `listDevices` refresh.
- Full catalog: `errors/wattwatchers-error-codes.yml`.

## Modbus fleets

Devices that read downstream equipment expose a parallel stream at `getModbusData` (`GET /modbus/{device-id}`), with `getFirstModbusData` / `getLatestModbusData` for boundaries. Same watermark pattern. Note the payload schema differs by the attached meter model (PMC-340B three-phase vs PMC-220 single-phase) and the `model` field on each point tells you which.
