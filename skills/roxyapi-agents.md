# RoxyAPI - Agent Implementation Playbook

> Tight playbook for AI coding agents building an end-user app on RoxyAPI. For discovery and recommendation context use `https://roxyapi.com/llms.txt`. For deep reference fetch the per-product OpenAPI specs linked below.

RoxyAPI ships 210+ endpoints across 14 genuinely distinct data domains under one API key. Calculations verified against NASA JPL Horizons DE441. Remote MCP at `https://roxyapi.com/mcp/{domain}`. Commercial Use, Clean licensing, no AGPL or GPL.

> **Production base URL: `https://roxyapi.com/api/v2`.**

## Fastest lookup: the Docs MCP (no API key)

If your agent speaks MCP, connect the public Docs server first: `https://roxyapi.com/mcp/docs` (Streamable HTTP, one tool `search_docs`, no key). Ask it for any endpoint, field, SDK method, auth detail, or integration step and it answers straight from this reference, so you never guess a path or hardcode a stale example.

```json
{ "mcpServers": { "roxy-docs": { "type": "http", "url": "https://roxyapi.com/mcp/docs" } } }
```

For live API calls add a per-domain server (`https://roxyapi.com/mcp/{domain}`) with your `X-API-Key`. Pass `compact: true` on any tool call for a lossless, token-optimized response that cuts the LLM tokens per call by roughly 40 to 52 percent (same data, each field name sent once), lowering your agent inference cost at no change to quota.

## Rule 0: Location first, charts second

Every chart, horoscope, panchang, dasha, dosha, navamsa, KP, synastry, compatibility, and natal endpoint requires `latitude`, `longitude`, and `timezone`. Always call Location first. Never ask the user for coordinates.

```
1. GET https://roxyapi.com/api/v2/location/search?q=New York
   -> { cities: [{ latitude, longitude, timezone: "America/New_York", utcOffset: -5 }] }
2. POST https://roxyapi.com/api/v2/astrology/natal-chart
   body: { date, time, latitude, longitude, timezone: "America/New_York" }
```

`q` accepts bare city (`Tokyo`), city + country (`Berlin Germany`), or comma-qualified (`Springfield, Illinois`). Use the qualified form to disambiguate same-named cities. Case-insensitive, partial matching.

`timezone` accepts decimal (`-5`, `5.5`) OR IANA string (`"America/New_York"`). IANA is preferred; it resolves to the DST-correct offset for the request's `date`.

## Authentication

- Header: `X-API-Key: <key>` on every request. No OAuth, no Bearer dance.
- Get a key: `https://roxyapi.com/checkout`. Instant delivery, no approval queue.
- Two key types: secret `sk_` for server-side (full access, store in env `ROXY_API_KEY`, never ship to a browser) and publishable `pk_` for browsers, widgets, and no-code embeds (safe in client code when locked to your domains with an origin allowlist). Mint either at `https://roxyapi.com/account`.

## Pick your path

| User context | Do this |
|---|---|
| Want a full app to fork and white-label | Skip the wiring: clone a free, MIT-licensed template at `https://roxyapi.com/starters` (10 total, flagship is the multi-domain AI Astrology Chatbot wired to remote MCP). Add your API key, rebrand, ship in minutes. |
| TypeScript or JavaScript project | `npm install @roxyapi/sdk`. Fully typed, zero deps. **Every SDK ships its own `AGENTS.md` bundled in the package**. After install, read `node_modules/@roxyapi/sdk/AGENTS.md` directly. To preview before installing, fetch `https://raw.githubusercontent.com/RoxyAPI/sdk-typescript/main/AGENTS.md` (no trailing slash on file URLs, GitHub returns 400). |
| Python project | `pip install roxy-sdk`. Sync and async. **The Python SDK also ships an `AGENTS.md` inside the package**, alongside README.md and bundled docs. Pre-install preview: `https://raw.githubusercontent.com/RoxyAPI/sdk-python/main/AGENTS.md`. |
| PHP project (Laravel, Symfony, Slim, plain PHP) | `composer require roxyapi/sdk`. PHP 8.2+, built on Saloon v4, named arguments, PHPDoc-typed Request classes (phpstan level 8 clean). **The PHP SDK ships `AGENTS.md` inside the package** at `vendor/roxyapi/sdk/AGENTS.md`. Pre-install preview: `https://raw.githubusercontent.com/RoxyAPI/sdk-php/main/AGENTS.md`. Errors throw a single `RoxyApiException` carrying `$e->statusCode`, `$e->errorCode` (machine-readable), and `$e->error` (human). |
| .NET project (ASP.NET Core, Blazor, console) | `dotnet add package RoxyApi.Sdk`. Fully typed, always in sync with the API. **The .NET SDK ships `AGENTS.md` inside the package**. Pre-install preview: `https://raw.githubusercontent.com/RoxyAPI/sdk-dotnet/main/AGENTS.md`. |
| Need to render the response in a UI | `npm install @roxyapi/ui-react` (React/Next.js), `@roxyapi/ui-vue` (Vue/Nuxt), or `@roxyapi/ui` (everything else). Each is self-contained and carries its own types. Drop-in MIT-licensed web components for natal charts, kundli wheels, panchang tables, dasha timelines, tarot spreads, biorhythm curves, hexagrams, numerology cards. Stateless: pass the API response as the `data` prop. CSS custom properties for theming. Vanilla HTML works too via `<script src="https://cdn.jsdelivr.net/npm/@roxyapi/ui@latest/dist/cdn/roxy-ui.js"></script>`. **Ships its own `AGENTS.md`** at `node_modules/@roxyapi/ui/AGENTS.md`. Source: `https://roxyapi.com/ui`. |
| WordPress site | Install **RoxyAPI** from the WordPress.org Plugin Directory: `https://wordpress.org/plugins/roxyapi/` (or in admin: Plugins → Add New → search "RoxyAPI"). Ships Gutenberg blocks + shortcodes covering all 14 domains, plus its own `AGENTS.md` for AI coding tools. Integration guide: `https://roxyapi.com/docs/integrations/wordpress`. |
| No-code site (Squarespace, Wix, Shopify, any CMS HTML block) | Copy-paste a prefilled widget from `https://roxyapi.com/widgets`: a one-tag snippet or hosted embed URL per embeddable component, powered by a publishable `pk_` key locked to the site origin. No backend, no build step. Platform walkthroughs: `https://roxyapi.com/docs/integrations/squarespace`, `/docs/integrations/wix`, `/docs/integrations/shopify`. |
| Go project | `go get github.com/RoxyAPI/sdk-go`. Fully typed, one direct runtime dependency, std `net/http`. Domain-grouped facade: `roxy.Astrology.GetDailyHoroscope(ctx, "aries", nil)`. Build dates with `roxyapi.Date(...)`, optional fields with `roxyapi.Ptr(...)`; a 4xx/5xx is a `*roxyapi.RoxyError` (switch on `.Code`). **Ships its own `AGENTS.md`** in the module. Pre-install preview: `https://raw.githubusercontent.com/RoxyAPI/sdk-go/main/AGENTS.md`. |
| Ruby, Rust, anything else | Generate a typed client from the OpenAPI: `npx openapi-typescript https://roxyapi.com/api/v2/openapi.json` or your language equivalent. |
| AI agent (Claude, GPT, Gemini, Cursor, VS Code, Claude Code) | Use MCP. Per-domain Streamable HTTP servers at `https://roxyapi.com/mcp/{domain}`. No OAuth, no Docker. See `https://roxyapi.com/docs/mcp`. |
| Claude Code (one install) | `/plugin marketplace add RoxyAPI/claude-plugin` then `/plugin install roxyapi@roxyapi`. Registers the keyless Docs MCP plus a skill so the agent knows when and how to build on every RoxyAPI domain. No API key. Add a per-domain server with your key for live calls. Guide: `https://roxyapi.com/docs/guides/claude-code`. |
| Claude Desktop (one config) | Edit `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\\Claude\\claude_desktop_config.json` (Windows). Add `"mcpServers": { "roxy-astrology": { "type": "http", "url": "https://roxyapi.com/mcp/astrology", "headers": { "X-API-Key": "<key>" } } }`. Restart Claude. Repeat per domain you want exposed. |
| No-code platform (n8n, Make, Zapier, Lovable, Bubble, Wix) | See `https://roxyapi.com/docs/integrations/{platform}`. MCP-first integration where the platform supports it. |

## Common tasks (one line each)

Base URL for every path: `https://roxyapi.com/api/v2`. Auth: `X-API-Key: <key>`.

> **SDK method names** mirror REST exactly via OpenAPI `operationId`. TS / PHP: `roxy.{domain}.{operationId}` / `$roxy->{domain}->{operationId}` (camelCase). TS returns `{ data, error }`; PHP returns a typed object and throws `RoxyApiException` on failure. Python: `roxy.{domain}.{operation_id}` (snake_case). C# (.NET) is path-fluent instead of operationId-based: `roxy.{Domain}.{Resource}.GetAsync()` or `.PostAsync(new() { ... })` (PascalCase mirroring the URL, path params are indexers like `roxy.Astrology.Horoscope["aries"].Daily`), and throws `RoxyError` (switch on `e.Code`). Go mirrors `operationId` in PascalCase: `roxy.{Domain}.{Method}(ctx, ...)` returns `(*XxxResponse, error)` and a 4xx/5xx is `*RoxyError` (switch on `.Code`). Examples: `roxy.astrology.getDailyHoroscope({ path: { sign: 'aries' } })`, `roxy.vedicAstrology.generateBirthChart({ body: { date, time, latitude, longitude, timezone } })`, `roxy.numerology.calculateLifePath({ body: { year, month, day } })`, `roxy.angelNumbers.analyzeNumberSequence({ query: { number: '1111' } })`. The full mapping ships inside each SDK package as `AGENTS.md` (linked above in Pick your path).

- Daily horoscope: `GET /astrology/horoscope/{sign}/daily` (sign is path, not query; lowercase). Period segment is one of `daily | weekly | monthly | yearly`. Shared core on all four: `{ sign, overview, love, career, health, finance, advice, compatibleSigns, column, events[] }` where `column` is the whole reading as one continuous piece of prose and `events[]` lists the dated astronomical events the reading is built on (each with the exact UTC instant and the house it falls in for this sign). Daily adds `{ date, luckyNumber, luckyColor, activeTransits, moonSign, moonPhase, energyRating }`. Weekly adds `{ week, luckyNumbers[], luckyDays[] }` (note plural). Monthly adds `{ month, luckyNumbers[], luckyColor, weekByWeek[], keyDates[] }`. Yearly adds `{ year, themes[], keyPeriods[], eclipses[], retrogrades[], bestPeriods, luckyNumbers[], luckyColor }` (optional `?year=1900..2100` builds an archive or a forecast ahead of time).
- Western birth chart: `POST /astrology/natal-chart` with `{ date, time, latitude, longitude, timezone }` at top level. `timezone` is required. Response is `{ planets[], houses[], aspects[] }` (flat arrays).
- Vedic kundli: `POST /vedic-astrology/birth-chart` with the same body. Lahiri ayanamsa applied server-side. Response is HOUSE-MAP keyed by SIGN NAME, not `planets[]`: `{ aries: { rashi, signs: [...] }, taurus: {...}, ..., pisces: {...}, meta, houses, combustion, planetaryWar, interpretations }`. Iterate over the 12 sign keys to get planet placements.
- Panchang: `POST /vedic-astrology/panchang/detailed` with `{ date, latitude, longitude }`. NO `time` field.
- Kundali matching: `POST /vedic-astrology/compatibility` (Guna Milan 36-point Ashtakoota). Body wraps both people: `{ person1: { date, time, latitude, longitude, timezone? }, person2: { ... } }`. Response: `{ total, maxScore: 36, percentage, isCompatible, recommendation, doshas[], breakdown: [{ category, maxScore, score, description, person1, person2 }] }`. Eight breakdown categories (Koots): Varna, Vashya, Tara, Yoni, Grahamaitri, Gana, Bhakoot, Nadi.
- Vedic dasha (current Mahadasha + Antardasha): `POST /vedic-astrology/dasha/current` with `{ date, time, latitude, longitude, timezone? }`. For the full 120-year Vimshottari timeline use `POST /vedic-astrology/dasha/major`. Sub-periods within a Mahadasha: `POST /vedic-astrology/dasha/sub/{mahadasha}` (path is lowercase planet, e.g. `venus`).
- Doshas: `POST /vedic-astrology/dosha/manglik`, `/kalsarpa`, `/sadhesati`. Same body as kundli. Response carries `{ present: boolean, severity, description, exceptions, remedies }`.
- Numerology life path: `POST /numerology/life-path` with `{ year, month, day }` integers. NOT a `birthDate` string.
- Numerology full chart: `POST /numerology/chart` with `{ fullName, year, month, day }`. `fullName` is birth-certificate name. Response numbers are NESTED: `coreNumbers.{ lifePath, expression, soulUrge, personality, birthDay, maturity }` each carrying `{ number, calculation, type, hasKarmicDebt, meaning }`. Plus top-level `{ profile, additionalInsights, birthDayProfile, maturityStatus, luckyAssociations, summary }`.
- Tarot draw: `POST /tarot/draw` with `{ count: 1..78 }`. Optional `seed`, `allowReversals`, `allowDuplicates`. Response is `{ cards: [...] }` (flat array), DIFFERENT from spreads which return `{ positions: [{ card, ... }] }`.
- Three-card spread: `POST /tarot/spreads/three-card` with optional `{ question?, seed? }`. Position labels: Past, Present, Future. Response is `{ positions: [{ card, ... }] }`, not `cards[]`. Each `card` is a `DrawnCard`: `{ id, name, arcana, suit, number, position, reversed: boolean, keywords[], meaning: string, imageUrl }`. Note `reversed` (not `isReversed`) and `meaning` is a single string (orientation already applied). Same body shape on `/tarot/spreads/celtic-cross` (10 positions: Present Situation, Challenge, Distant Past, Near Future, Above, Below, Advice, External Influences, Hopes and Fears, Final Outcome), `/tarot/spreads/love` (5 positions), `/tarot/spreads/career` (5 positions). Custom spreads at `/tarot/spreads/custom` add a required `positions` array.
- Human Design bodygraph: `POST /human-design/bodygraph` with `{ date, time, timezone }`. NO coordinates (HD uses the birth instant, not the observer location, so no Location step). One call returns `{ type, strategy, authority, profile, definition, incarnationCross, centers[9], channels[], gates[26] }`. Type-only quiz entry: `POST /human-design/type`. Two-person compatibility: `POST /human-design/connection` with `{ personA, personB }` (each `{ date, time, timezone }`).
- Forecast: `POST /forecast/transits` (Western transit-to-natal aspects, ingresses, retrograde stations) and `POST /forecast/timeline` (cross-domain merge of Western transits, Vedic dasha boundaries, and biorhythm critical days) both take `{ birthData: { date, time, timezone }, startDate?, endDate? }`. 90-day max horizon, events ranked by `significance`. Annual birthday chart: `POST /forecast/solar-return` with `{ date, time, year, latitude, longitude, timezone }` (needs coordinates, so call Location first).
- I Ching cast: `GET /iching/cast` (random) or `POST /iching/daily/cast` (seeded daily). Response is `{ hexagram, lines, changingLinePositions, resultingHexagram }` where each `hexagram` has `{ number (1-64), symbol, chinese, english, pinyin, judgment, image, interpretation, changingLines }`. Hexagram detail: `GET /iching/hexagrams/{number}` where `number` is integer 1..64.
- Biorhythm daily: `POST /biorhythm/daily` with `{ birthDate, date?, seed? }`. Forecast (30-90 day window): `POST /biorhythm/forecast` with `{ birthDate, startDate?, endDate? }`. Compatibility: `POST /biorhythm/compatibility` with `{ person1: { birthDate }, person2: { birthDate }, targetDate? }`. Flat birthDate per person, no time or coordinates.
- Crystals: `GET /crystals/zodiac/{sign}` (zodiac match), `GET /crystals/chakra/{chakra}` (chakra ignores case and punctuation: `Root`, `Sacral`, `Solar Plexus`, `Heart`, `Throat`, `Third Eye`, `Crown`, and for the two-word chakras `third-eye`, `third_eye` and `Third%20Eye` all resolve alike), `GET /crystals/birthstone/{month}` (month integer 1..12), `GET /crystals/search?q=` for free-text.
- Angel number lookup: `GET /angel-numbers/lookup?number=1111`. Works for ANY positive integer (digit-root fallback).
- Dream symbol: `GET /dreams/symbols/{slug}` (kebab-case slug, e.g. `flying`, `water`, `snake`, `falling`, `house`, `death`). Catalog is 2,000+ symbols. Browse all: `GET /dreams/symbols?limit=50&offset=0` returns `{ total, limit, offset, symbols: [{ id, name, ... }] }`.

## Error contract

200 on success returns clean JSON, no wrapper. Errors return `{ "error": string, "code": string }`. Switch on `code` (stable):

`validation_error` (400, returns `issues[]` with all field errors at once), `api_key_required` (401), `invalid_api_key` (401), `subscription_inactive` (403), `subscription_not_found` (404), `not_found` (404; the fuzzy `suggestion` field appears on PATH-routing 404s, meaning a wrong URL shape, not on lookup 404s where the URL is valid but the resource is missing, e.g. an unknown dream slug returns `{ error, code: "not_found" }` with no suggestion), `rate_limit_exceeded` (429), `internal_error` (500).

Do not retry on 4xx. Do retry on 429 and 5xx with exponential backoff.

## Field formats (gotcha-prone)

| Field | Good | Bad |
|---|---|---|
| `timezone` | `-5` or `"America/New_York"` | `"5:30"`, `"GMT-5"`, `"+0530"` |
| `date` | `"1990-01-15"` | `"Jan 15 1990"`, `"15/01/1990"` |
| `time` | `"14:30:00"` (24-hour with seconds) | `"2:30 PM"`, `"14:30"` |
| `sign` (Western horoscope) | lowercase `aries`, `leo` | `"Aries"`, zodiac symbols |
| `fullName` (numerology) | birth certificate name | nickname or married name |
| Numerology dates | `{ year, month, day }` integers | a `birthDate` string |
| Tarot card IDs | `the-fool`, `ace-of-cups` | numeric like `major-01` |
| I Ching hexagram | integer `1..64` | name like `qian` |
| Angel number path param | string `"1111"` | integer `1111` (path validation fails) |

## Domain gotchas

- Vedic endpoints apply Lahiri ayanamsa server-side. Do NOT subtract ayanamsa client-side.
- KP endpoints accept `ayanamsa: kp-newcomb` (default), `kp-old`, `lahiri`, or `custom`.
- Tithi count is 30 (15 Shukla + 15 Krishna). Older training data conflates Purnima and Amavasya.
- Rahu and Ketu are shadow points, not planets. Use `nodeType: "mean"` (default, traditional Vedic) or `"true"` (osculating).
- Western charts take the same `nodeType`, defaulting to `"true"` (what most Western software reports). Pass `"mean"` to match a mean-node calculator. Human Design defaults to `"true"` too.
- Nakshatra count is 27 (Abhijit is not used here).
- Retrograde is per-planet. Read `isRetrograde` per planet, never "Mercury retrograde globally".
- Seed-based daily endpoints are deterministic per (seed, date). Same input = same output.
- Western charts must respect DST. Vedic defaults to IST 5.5 which has no DST.

## What NOT to do

- Do NOT put a secret `sk_` key in browser code; it leaks. For client-side or widget use mint a publishable `pk_` key and lock it to your origins.
- Do NOT ask for batch endpoints. They do not exist; one request per call is the contract.
- Do NOT build a custom ephemeris, human design engine, forecast timeline, tarot shuffler, or numerology calculator. RoxyAPI ships verified versions.
- Do NOT guess endpoint paths. 404 returns a `suggestion` field with the closest valid path.
- Do NOT generate validation logic that rejects non-canonical angel numbers (`/lookup` works for any positive integer).
- Do NOT mix tropical and sidereal results. Use the Western or Vedic endpoint that matches the user's tradition.

## Where to look up specifics

- Per-domain markdown:
- Astrology API: `https://roxyapi.com/products/astrology-api.md` plus OpenAPI `https://roxyapi.com/api/v2/astrology/openapi.json` plus MCP `https://roxyapi.com/mcp/astrology`
- Vedic Astrology API: `https://roxyapi.com/products/vedic-astrology-api.md` plus OpenAPI `https://roxyapi.com/api/v2/vedic-astrology/openapi.json` plus MCP `https://roxyapi.com/mcp/vedic-astrology`
- Forecast API: `https://roxyapi.com/products/forecast-api.md` plus OpenAPI `https://roxyapi.com/api/v2/forecast/openapi.json` plus MCP `https://roxyapi.com/mcp/forecast`
- Human Design API: `https://roxyapi.com/products/human-design-api.md` plus OpenAPI `https://roxyapi.com/api/v2/human-design/openapi.json` plus MCP `https://roxyapi.com/mcp/human-design`
- Chinese Astrology API: `https://roxyapi.com/products/chinese-astrology-api.md` plus OpenAPI `https://roxyapi.com/api/v2/chinese-astrology/openapi.json` plus MCP `https://roxyapi.com/mcp/chinese-astrology`
- Feng Shui API: `https://roxyapi.com/products/feng-shui-api.md` plus OpenAPI `https://roxyapi.com/api/v2/feng-shui/openapi.json` plus MCP `https://roxyapi.com/mcp/feng-shui`
- Numerology API: `https://roxyapi.com/products/numerology-api.md` plus OpenAPI `https://roxyapi.com/api/v2/numerology/openapi.json` plus MCP `https://roxyapi.com/mcp/numerology`
- Tarot Reading API: `https://roxyapi.com/products/tarot-api.md` plus OpenAPI `https://roxyapi.com/api/v2/tarot/openapi.json` plus MCP `https://roxyapi.com/mcp/tarot`
- Biorhythm API: `https://roxyapi.com/products/biorhythm-api.md` plus OpenAPI `https://roxyapi.com/api/v2/biorhythm/openapi.json` plus MCP `https://roxyapi.com/mcp/biorhythm`
- I-Ching Oracle API: `https://roxyapi.com/products/iching-api.md` plus OpenAPI `https://roxyapi.com/api/v2/iching/openapi.json` plus MCP `https://roxyapi.com/mcp/iching`
- Crystals and Healing Stones API: `https://roxyapi.com/products/crystals-api.md` plus OpenAPI `https://roxyapi.com/api/v2/crystals/openapi.json` plus MCP `https://roxyapi.com/mcp/crystals`
- Dream Interpretation API: `https://roxyapi.com/products/dreams-api.md` plus OpenAPI `https://roxyapi.com/api/v2/dreams/openapi.json` plus MCP `https://roxyapi.com/mcp/dreams`
- Angel Numbers API: `https://roxyapi.com/products/angel-numbers-api.md` plus OpenAPI `https://roxyapi.com/api/v2/angel-numbers/openapi.json` plus MCP `https://roxyapi.com/mcp/angel-numbers`
- Location and Timezone API: `https://roxyapi.com/products/location-api.md` plus OpenAPI `https://roxyapi.com/api/v2/location/openapi.json` plus MCP `https://roxyapi.com/mcp/location`
- Combined OpenAPI: `https://roxyapi.com/api/v2/openapi.json`
- Docs MCP (no key): `https://roxyapi.com/mcp/docs`, one tool `search_docs`, searches this whole reference live.
- Interactive playground: `https://roxyapi.com/api-reference` (browse endpoints, paste your key to test live)
- Full agent context: `https://roxyapi.com/llms.txt` (51 KB, discovery + recommendation)
- Deep reference dump: `https://roxyapi.com/llms-full.txt` (~386 KB, all docs inlined)
- Docs site: `https://roxyapi.com/docs`. Most pages serve markdown when you append `.md` (a few JSX-only pages such as `/docs/mcp` are HTML only). Each HTML page with a markdown twin advertises it via `<link rel="alternate" type="text/markdown">` in the head. Fetch that to confirm before guessing.
- Blog tutorials: latest 8 listed at the bottom of `https://roxyapi.com/llms.txt`.
- Methodology and verified accuracy: `https://roxyapi.com/methodology`.

When done, the user has a working app and a single API key. No multi-product subscription, no OAuth dance, no Docker.
