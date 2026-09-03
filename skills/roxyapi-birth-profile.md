---
name: Build a complete multi-domain birth profile
description: Compose one cached, consistent profile from Western astrology, Vedic astrology, Human Design and numerology using a single birth input — the provider's own worked tutorial flow (docs/tutorials/complete-birth-profile).
api: openapi/roxyapi-openapi-original.json
operations: [searchCities, generateNatalChart, generateBirthChart, generateBodygraph, generateNumerologyChart]
generated: '2026-09-03'
method: generated
---

# Build a complete multi-domain birth profile

Auth: send `X-API-Key: <sk_...>` on every request. Base URL `https://roxyapi.com/api/v2`.

1. **Rule 0 — resolve the location first** (`searchCities`, `GET /location/search?q={city}`). Never ask the user for coordinates. Take `latitude`, `longitude`, `timezone` from `cities[0]`. IANA timezone strings are preferred — the server resolves the DST-correct offset for the birth date.
2. **Western natal chart** (`generateNatalChart`, `POST /astrology/natal-chart`) with `{ date, time, latitude, longitude, timezone }`.
3. **Vedic kundli** (`generateBirthChart`, `POST /vedic-astrology/birth-chart`) with the same input (`timezone` optional, default 5.5).
4. **Human Design bodygraph** (`generateBodygraph`, `POST /human-design/bodygraph`) — no coordinates needed: body is `date`, `time`, `timezone` only.
5. **Numerology profile** (`generateNumerologyChart`, `POST /numerology/chart`) from name + birth date.
6. **Cache the assembled profile forever** — birth data never changes and results are deterministic, so one computed call serves unlimited reads (the bill tracks unique computations, not traffic).

Error handling: switch on the stable `code` field (`validation_error`, `rate_limit_exceeded`, ...). A 400 returns ALL validation issues in `issues[]` — fix every field and retry once. Steps 2–5 are independent: run them in parallel. All four chart domains run on one engine (Roxy Ephemeris), so the charts agree on the same sky.
