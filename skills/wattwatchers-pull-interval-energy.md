---
generated: '2026-07-27'
method: generated
name: Pull interval energy data for a device
description: Discover a device, learn its channel layout, then pull Long Energy interval data for a bounded time window and label each series correctly.
api: openapi/wattwatchers-rest-api-v3-openapi.json
operations: [listDevices, getDevice, getFirstLongEnergyData, getLatestLongEnergyData, getLongEnergyData]
source: >-
  Grounded in openapi/wattwatchers-rest-api-v3-openapi.json; every operationId
  verified verbatim in that spec. Cross-cutting rules from
  conventions/wattwatchers-conventions.yml,
  errors/wattwatchers-error-codes.yml and
  rate-limits/wattwatchers-rate-limits.yml.
---

# Pull interval energy data for a device

The core read flow of the Wattwatchers API: find your devices, understand the channel layout, then retrieve 5-minute Long Energy intervals for a window.

## Auth

- `Authorization: Bearer key_...` on every request. The key is issued by hand by Wattwatchers and only returns the devices assigned to it. See `authentication/wattwatchers-authentication.yml`.
- There is no sandbox and no test key. Any call you make is against production hardware data.

## Steps

1. **List your devices** — `listDevices` (`GET /devices`). Returns a flat, unpaginated array of device ID strings (e.g. `["D123456789012", ...]`). These are the only devices your key can see.
2. **Read the device record** — `getDevice` (`GET /devices/{device-id}`). Capture `channels[]` in order, along with each channel's `label`, `ctRating` and `categoryId`. **This step is mandatory**: energy responses are positional arrays with no channel names, so `eReal[n]` only means something once you know `channels[n]`. Also note `phases` if the site is three-phase.
3. **Find the data boundaries** — `getFirstLongEnergyData` (`GET /long-energy/{device-id}/first`) and `getLatestLongEnergyData` (`GET /long-energy/{device-id}/latest`). Use `fields[energy]=timestamp` to get just the timestamp cheaply. These tell you the earliest and most recent intervals available, so you never request a window outside the device's life.
4. **Pull the window** — `getLongEnergyData` (`GET /long-energy/{device-id}`) with `fromTs` and `toTs` as integer Unix seconds. `fromTs` is inclusive, `toTs` is exclusive.
   - Optional `granularity`: `5m`, `15m`, `30m`, `hour`, `day`, `week`, `month`.
   - `timezone` is **required** when `granularity` is `hour` or coarser, and ignored below that.
   - Optional `convert[energy]=kWh` (energy) or `kW` (power) to normalise units.
   - Optional `fields[energy]=+pf` to append calculated power factor.
   - Optional `filter[group]=phases` to collapse channels per the device's phase grouping.
5. **Label the series** — zip each positional array (`eReal`, `eRealPositive`, `eRealNegative`, `eReactive`, `vRMSMin/Max`, `iRMSMin/Max`) against the channel order from step 2.

## Constraints

- Maximum window is **7 days** for Long Energy at default granularity. Exceeding it returns `422 UNPROCESSABLE_ENTITY` with the message "The requested time period is greater than 7 days" — the request is rejected, not truncated. Chunk longer backfills.
- `fields[energy]=+pf` cannot be combined with `filter[group]=phases`, nor with `fields[energy]=timestamp`.
- For 30-second resolution use the Short Energy operations instead (`getShortEnergyData`, `getFirstShortEnergyData`, `getLatestShortEnergyData`) — the same shape with a **12 hour** maximum window.

## Errors

- `204` — the device has **never** reported Long Energy (not yet installed or initialised). Not an error.
- `200` with `[]` — the device has data, but none inside your window. Different from 204; do not conflate.
- `400 BAD_REQUEST` — malformed parameter, or `fromTs` omitted where required.
- `422 UNPROCESSABLE_ENTITY` — well-formed but invalid timestamp, or window too long for the granularity.
- `404 NOT_FOUND` — unknown device **or** a device not assigned to your key; the two are deliberately indistinguishable.
- Full catalog: `errors/wattwatchers-error-codes.yml`.

## Rate limits

- Per-key TPS and TPD, auto-scaled by device count. Read `X-RateLimit-TpsRemaining` / `X-RateLimit-TpdRemaining` on every response and pace accordingly.
- On `429`, back off for the integer seconds in `Retry-After`. `TOO_MANY_REQUESTS_TPS` is the per-second limit, `TOO_MANY_REQUESTS_TPD` the per-day limit (resets 12AM UTC).
- See `rate-limits/wattwatchers-rate-limits.yml`.
