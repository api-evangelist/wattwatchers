# Wattwatchers (wattwatchers)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Wattwatchers is an Australian digital energy company (founded 2007, headquartered in Sydney, NSW) that designs and manufactures the Auditor family of DIN-rail electricity monitoring and switching devices, and operates the cloud platform behind them. It sits behind the meter rather than at it — Auditors clamp onto individual circuits and report real-time, circuit-level energy data over 4G or WiFi to Wattwatchers' hosted platform, which resells that data to solar installers, energy retailers, energy services companies, EV and DER programs, schools and research trials. Its API posture is genuinely open at the documentation layer and closed at the data layer: the full REST API v3 (Mercury) reference, a live Swagger UI and a downloadable OpenAPI 3.0 contract are all served anonymously from docs.wattwatchers.com.au, but no key is self-serve — Wattwatchers issues bearer tokens by hand and scopes each one to the specific devices you own or manage, so the API returns your fleet's data and nothing else. Consumer usage data is therefore available through a documented API, while no open grid or market data is published at all. Wattwatchers is not a designated Consumer Data Right energy data holder and does not appear as an accredited data recipient — the CDR energy mandate applies to retailers and AEMO, not to behind-the-meter hardware vendors, so this platform carries no CDR obligation and implements no CDR or Green Button data standard.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/wattwatchers/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/wattwatchers/refs/heads/main/apis.yml)

## Tags

- Energy
- Australia
- Utilities
- Electricity
- Smart Metering
- Energy Data
- IoT
- Solar
- DER
- Demand Response

## Timestamps

- **Created:** 2026-07-27
- **Modified:** 2026-07-27

## APIs

### Wattwatchers REST API v3 (Mercury)

The Wattwatchers REST API v3, code-named Mercury — 14 documented operations across 13 paths, covering device inventory and configuration (including switch control via PATCH), 30-second "short energy" interval data, 5-minute "long energy" interval data, and Modbus data read from downstream devices by 6M+One hardware. Authentication is a Wattwatchers-issued bearer API key scoped to the devices assigned to that key. Version 3.6.0 of the published OpenAPI contract.

- **Human URL:** [https://docs.wattwatchers.com.au/api/v3/index.html](https://docs.wattwatchers.com.au/api/v3/index.html)
- **Base URL:** `https://api-v3.wattwatchers.com.au`

#### Tags

- Devices
- Short Energy
- Long Energy
- Modbus
- Energy Data
- Smart Metering

#### Properties

- [OpenAPI](openapi/wattwatchers-rest-api-v3-openapi.json) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Documentation](https://docs.wattwatchers.com.au/api/v3/index.html)
- [API Reference](https://docs.wattwatchers.com.au/api/v3/endpoints.html)
- [Interactive Documentation](https://docs.wattwatchers.com.au/api/v3/openapi/index.html)
- [Authentication](https://docs.wattwatchers.com.au/api/v3/auth.html)
- [Rate Limits](https://docs.wattwatchers.com.au/api/v3/rate-limits.html)
- [Errors](https://docs.wattwatchers.com.au/api/v3/errors.html)
- [Conventions](https://docs.wattwatchers.com.au/api/v3/conventions.html)
- [Change Log](https://docs.wattwatchers.com.au/api/v3/release-notes.html)
- [Sample Code](https://github.com/wattwatchers/rest-api-notebooks)

#### Documented Operations

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/devices` | Device IDs accessible to the current API key |
| GET | `/devices/{device-id}` | Device status, comms, channels, phases, switches |
| PATCH | `/devices/{device-id}` | Update device config — labels, CT ratings, switch state |
| GET | `/devices/channel-categories` | Valid channel categories (e.g. Solar) |
| GET | `/devices/models` | Valid device models |
| GET | `/short-energy/{device-id}` | 30-second interval energy data |
| GET | `/short-energy/{device-id}/first` | First short-energy record |
| GET | `/short-energy/{device-id}/latest` | Latest short-energy record |
| GET | `/long-energy/{device-id}` | 5-minute interval energy data |
| GET | `/long-energy/{device-id}/first` | First long-energy record |
| GET | `/long-energy/{device-id}/latest` | Latest long-energy record |
| GET | `/modbus/{device-id}` | Modbus data over a timeframe |
| GET | `/modbus/{device-id}/first` | First Modbus record |
| GET | `/modbus/{device-id}/latest` | Latest Modbus record |

## Access, Standards and Mandate

- **Home market:** Australia
- **Mandate regime:** none — Wattwatchers is a behind-the-meter hardware and platform vendor, not an energy retailer or AEMO, so the Consumer Data Right energy regime does not designate it.
- **Mandate status:** not applicable — verified against the live CDR Register API on 2026-07-27: absent from the 84 designated energy data-holder brands and absent from the 38 accredited data recipients.
- **Data standard:** no standard reference found — proprietary JSON/REST. No Green Button/ESPI, no CDR Consumer Data Standards, no IEEE 2030.5, OpenADR, OCPP/OCPI or IEC CIM. Modbus appears only as a device-side fieldbus the hardware reads from, not as a data-sharing standard.
- **Consumer data API:** yes — per-circuit interval usage for a named site, through a documented API.
- **Open market data:** no — nothing is published anonymously; every endpoint requires a key.
- **Access gate:** customer-account-required — you must operate Wattwatchers hardware, get into the Fleet Management app, and have Wattwatchers issue a key scoped to your devices. No signup, no free tier, no sandbox.
- **Auth model:** HTTP Bearer token (`Authorization: Bearer key_...`), declared as `BearerAuth` in the OpenAPI. No OAuth2, no OIDC (`/.well-known/openid-configuration` returns 404), no scopes.
- **Transport:** REST only. No webhooks, MQTT or WebSocket; the docs describe a stream-based push API as something Wattwatchers is "exploring", so integrators poll.

## Common Properties

- [Website](https://wattwatchers.com.au/)
- [Documentation](https://docs.wattwatchers.com.au/)
- [Interactive Documentation](https://docs.wattwatchers.com.au/api/v3/openapi/index.html)
- [Authentication](https://docs.wattwatchers.com.au/api/v3/auth.html)
- [Rate Limits](https://docs.wattwatchers.com.au/api/v3/rate-limits.html)
- [Change Log](https://docs.wattwatchers.com.au/api/v3/release-notes.html)
- [GitHub Organization](https://github.com/wattwatchers)
- [Sample Code](https://github.com/wattwatchers/rest-api-notebooks)
- [LinkedIn](https://www.linkedin.com/company/wattwatchers-digital-energy)
- [Support](https://service.wattwatchers.com.au/)
- [Applications](https://docs.wattwatchers.com.au/apps.html)

## Maintainers

**FN:** Kin Lane
**Email:** kin@apievangelist.com
