---
name: Generate a personal forecast timeline
description: One time-ordered, significance-scored forecast merging Western transits, Vedic dasha boundaries and biorhythm critical days for a single subject.
api: openapi/roxyapi-openapi-original.json
operations: [searchCities, generateTimeline, calculateTransits]
generated: '2026-09-03'
method: generated
---

# Generate a personal forecast timeline

Auth: `X-API-Key` header. Base URL `https://roxyapi.com/api/v2`.

1. **Resolve the location** (`searchCities`, `GET /location/search?q={city}`) if you have a city name; forecast accepts `latitude`/`longitude` as optional (default 0) inside `birthData`.
2. **Cross-domain timeline** (`generateTimeline`, `POST /forecast/timeline`): body wraps `birthData` (`date`, `time`, `timezone`) plus optional `startDate`, `endDate` (clamped to a 90-day horizon), `minSignificance`, and an optional `domains` subset. Events come back time-ordered with stable English `type`/`domain` codes; only `description` localizes with `?lang=`.
3. **Transit detail on demand** (`calculateTransits`, `POST /astrology/transits`): current sky, or pass the natal chart for personalized transit-to-natal aspects.

Rules: the horizon is capped at 90 days — do not request wider windows. Rate-limit state is in `X-RateLimit-Remaining`/`X-RateLimit-Reset`; on 429 stop until the calendar-month reset. Results are deterministic — cache per (birthData, window).
