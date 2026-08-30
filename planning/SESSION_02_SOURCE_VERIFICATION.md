# NQ Premarket Environmental Intelligence Scanner
## Session 2 — Source Verification
### Deliverables P, Q, R, S — scope: N.8, N.9, N.10, N.12

---

## SESSION SCOPE CONFIRMATION

**Session 1 read completely:** `planning/SESSION_01_ARCHITECTURE_FOUNDATION.md` was
read in full (all 780 lines — Deliverables A–E, unresolved-decision register
N.1–N.24, and implementation-readiness assessment O) before any verification began.

**Recorded Agent Reach commit SHA:** `93ae1d18c37b707dec053c7c4f9d91cd8ef8943d`
(2026-08-12 11:39:47 +0800, subject `fix(readme): update sponsor copy`).
Confirmed present in the local checkout via `git show -s`. This matches Session 1
§B.11 and §O exactly. **Minor discrepancy, no action required:** the Session 1
Git *commit subject* (`590bab7 "Session 1: architecture foundation (AR @ 93ea1d8)"`)
contains a transposition typo — the correct short ref is `93ae1d1`, not `93ea1d8`
(the latter is not a valid revision). The Session 1 document body is correct;
only the commit subject line is wrong. Flagged for the record.

**Session type:** source-verification only. This session produced **no scanner
code, no Agent Reach modification, no production scanner files, no schemas or
databases, and no owner decisions**. All network activity was read-only HTTP GET.
No source was *activated* into any registry (none exists). Recommendations below
are recommendations only; activation and terms sign-off remain owner decisions
(N.12, N.13).

**Verification method:** read-only HTTP GET via two paths —
1. **Direct** (`WebFetch` against the target host).
2. **Jina Reader** (`https://r.jina.ai/<url>`) — the exact retrieval path
   `agent_reach.channels.web.WebChannel.read()` uses (Session 1 §A.1), so a
   "works via Jina" result is directly meaningful for the scanner's real
   ingestion path.
Plus `WebSearch` for corroboration of endpoint-deprecation history.

**Environment caveat (applies to every result below):** all checks ran from
**Claude Code's WebFetch infrastructure, not the operator's production machine**.
Direct-path failures (`403`, timeout) are a datacenter/CDN-egress signal and
**must be re-tested from the actual production host** before being treated as
final — the operator's network may fare better *or* worse. Jina-route and
structured-API results are substantially host-independent and are the safer basis
for the recommendations.

**Confidence labels used below:** `VERIFIED-LIVE` (a GET was performed this
session and the outcome is quoted), `CORROBORATED` (a `WebSearch` or a second
independent request supports it), `ASSUMED` (stated explicitly, not tested).

---

## DELIVERABLE P — SOURCE REACHABILITY VERIFICATION

### P.1 — N.8: BLS reachability

#### P.1.1 What was tested

| Target | Route | Result | Label |
|---|---|---|---|
| `https://www.bls.gov/feed/bls_latest.rss` | direct | **HTTP 403 Forbidden**, body withheld | VERIFIED-LIVE |
| `https://www.bls.gov/feed/empsit.rss` | direct | **HTTP 403 Forbidden** | VERIFIED-LIVE |
| `https://www.bls.gov/schedule/news_release/2026_sched.htm` | direct | **HTTP 403 Forbidden** | VERIFIED-LIVE |
| `https://www.bls.gov/news.release/empsit.toc.htm` | direct | **HTTP 403 Forbidden** | VERIFIED-LIVE |
| `https://www.bls.gov/developers/api_signature_v2.htm` | direct | **HTTP 403 Forbidden** | VERIFIED-LIVE |
| `https://www.bls.gov/schedule/2026/home.htm` | **Jina** | **HTTP 200** — full 2026 "Schedule of Selected Releases": CPI and Employment Situation dates, all @ 8:30 AM ET | VERIFIED-LIVE |
| `https://www.bls.gov/news.release/empsit.nr0.htm` | **Jina** | **HTTP 200** — full Employment Situation release narrative (opening line: *"Both nonfarm payroll employment (-23,000) and the unemployment rate (4.1 percent) changed little in July…"*) | VERIFIED-LIVE |
| `https://api.bls.gov/publicAPI/v2/timeseries/data/CUUR0000SA0` (CPI-U, NSA) | direct | **HTTP 200, `"status":"REQUEST_SUCCEEDED"`** — JSON series; latest Jul 2026 = 333.918 | VERIFIED-LIVE |
| `.../CUUR0000SA0L1E` (Core CPI, all items less food & energy) | direct | **`REQUEST_SUCCEEDED`** — Jul 2026 = 337.133, Jun 2026 = 336.882 | VERIFIED-LIVE |
| `.../CES0000000001` (Total Nonfarm Payrolls, level) | direct | **`REQUEST_SUCCEEDED`** — monthly levels through Jul 2026 | VERIFIED-LIVE |
| `.../LNS14000000` (civilian unemployment rate) | direct | **`REQUEST_SUCCEEDED`** — Jul 2026 = 4.1%, Jun 2026 = 4.2% | VERIFIED-LIVE |

Correct BLS schedule URL pattern (the Session-1-era guess `/schedule/news_release/2026_sched.htm`
is a genuine 404): **`https://www.bls.gov/schedule/2026/home.htm`** (whole year) and
**`https://www.bls.gov/schedule/2026/MM_sched.htm`** (per month), retrieved via Jina.

#### P.1.2 Corroboration

- `CORROBORATED` — `api.bls.gov` v2 works unregistered at low volume; a **free**
  registration key (submit email + organization + CAPTCHA, key mailed from
  `labstat@bls.gov`) raises limits from the unregistered tier to **500 queries/day,
  50 series/request, 20-year ranges**, plus catalog metadata and net/percent
  calculations. Registration is a form, not a contract negotiation.
- `VERIFIED-LIVE` — the API reports government-shutdown gaps honestly: October 2025
  returns `"-"` with footnote *"Data unavailable due to the 2025 lapse in
  appropriations."* The scanner must treat missing recent periods as a normal,
  expected condition, not an error.
- `CORROBORATED` — `www.bls.gov` sits behind Akamai bot management; a bare
  `403 Forbidden` with no body is Akamai's standard automated-client response.
  This is the mechanism behind the Section 25 / N.8 "prior host-level BLS access
  restriction" note.

#### P.1.3 Finding

BLS is **reachable for every datum the scanner needs**, by two independent routes
that both worked from this (datacenter) environment:

1. **`api.bls.gov/publicAPI/v2/` — primary, for numbers.** Structured JSON series
   for CPI-U, Core CPI, Nonfarm Payrolls, unemployment rate, and any other series
   ID. The scanner computes MoM / YoY from the index/level itself. Most robust
   route: a real API, not a scrape; a separate host from the blocked `www.bls.gov`.
2. **Jina Reader (`r.jina.ai/https://www.bls.gov/…`) — secondary, for prose and
   schedule.** Retrieves the release calendar and the full release narrative,
   which the API does not carry. This is the scanner's normal `WebChannel.read()`
   path, so no adapter change is implied.

**Direct requests to `www.bls.gov` are bot-blocked (403)** — confirmed for RSS,
schedule, release, and even the developer-docs page. The scanner should model the
direct path as **expected-to-fail** and never depend on it.

#### P.1.4 Residual risk

- Jina egress could itself be blocked by BLS/Akamai in future. Mitigation: the API
  route carries the decision-critical numbers; Jina only adds narrative colour and
  the schedule (the schedule is also derivable a year ahead from a single annual
  fetch, or hand-maintained — BLS pre-announces all dates).
- Unregistered API quota is small and shared; a burst could 429. Mitigation:
  obtain the free key before any real run (owner follow-up, N.24-adjacent).
- All 403s were from datacenter infra. **Operator must re-test `www.bls.gov`
  directly from the production host** — a residential/office IP may not be blocked,
  which would restore the direct path as a fallback.

#### P.1.5 Remaining operator steps

1. Re-test `https://www.bls.gov/` directly from the production machine; record
   whether Akamai blocks that IP.
2. Register for a BLS Public Data API key (`https://data.bls.gov/registrationEngine/`).
3. Confirm the exact series IDs the scanner will pull (candidate list in Q.4) and
   that 50 series/request covers a single run within one query.

---

### P.2 — N.9: Reuters discovery and retrieval route

#### P.2.1 What was tested

| Target | Route | Result | Label |
|---|---|---|---|
| `https://feeds.reuters.com/reuters/businessNews` | direct | **infra-blocked** (`unable to fetch`) | VERIFIED-LIVE |
| `https://www.reuters.com/markets/` | direct | **infra-blocked** (`Claude Code is unable to fetch from www.reuters.com`) | VERIFIED-LIVE |
| `https://www.reuters.com/arc/outboundfeeds/rss/?outputType=xml` | direct | **infra-blocked** | VERIFIED-LIVE |
| `https://www.reuters.com/markets/` | **Jina** | **HTTP 403 Forbidden** | VERIFIED-LIVE |

#### P.2.2 Corroboration

- `CORROBORATED` — Reuters **discontinued official RSS in June 2020**;
  `feeds.reuters.com` has been dead since. No official Reuters RSS exists as of
  2025–2026. (Multiple independent write-ups; FiveFilters, HN, GitHub issue
  trackers all consistent.) This is exactly the situation Section 8 warns about:
  *"do not assume historical Reuters RSS endpoints still exist."* They do not.
- `ASSUMED` — Reuters' terms of use prohibit automated scraping/crawling of
  reuters.com for content reuse; the site is fronted by aggressive anti-bot
  (returns 401/403 to non-browser clients). Not independently re-read this session;
  treated as known.

#### P.2.3 Finding

Reuters has **no machine-readable feed** and **both retrieval paths available to
the scanner fail** — direct is blocked upstream and the Jina route returns 403.
There is no viable route to Reuters as a *fixed, polled* source.

**Recommendation: exclude Reuters from the V1 fixed-source set.** If Reuters
coverage is ever wanted, the only defensible pattern is:

- **Exa-discovery-only.** The Exa adapter (Session 1 item 8 / §A.9) may surface
  Reuters URLs in results; the scanner keeps the **Exa-provided title + snippet**
  as *low-authority corroboration only*.
- **Full-text retrieval modeled as expected-to-fail** (Jina returns 403 today).
- **Never Tier-1 / primary authority**, never the sole support for any evidence
  cluster, never quoted verbatim (retrieval failure means no verifiable source
  text).

#### P.2.4 Residual risk

- Low, given exclusion. The institutional universe the scanner actually needs
  (Fed, Treasury, BLS, BEA) is Tier-1 and does not depend on Reuters.
- If the owner later insists on wire coverage, the terms-review obligation (N.13)
  is non-trivial and should precede any Exa-mediated use.

#### P.2.5 Remaining operator steps

1. Owner confirms Reuters is **out of scope for V1** (recommended), or explicitly
   accepts the Exa-discovery-only, no-retrieval, low-authority pattern and
   commissions a terms review.
2. If wire-service breadth is desired, evaluate alternatives with permissible
   automated access as a separate future item (not in this session's scope).

---

### P.3 — N.10: Treasury official access routes and endpoints

#### P.3.1 What was tested

| Target | Route | Result | Label |
|---|---|---|---|
| `https://home.treasury.gov/` | direct | **timeout (>60s)** | VERIFIED-LIVE |
| `https://home.treasury.gov/news/press-releases` | direct | **timeout (>60s), 2 attempts** | VERIFIED-LIVE |
| `https://home.treasury.gov/rss/press.xml` | direct | **timeout (>60s)** | VERIFIED-LIVE |
| `https://home.treasury.gov/resource-center/data-chart-center/interest-rates/daily-treasury-rates.csv/2026/all?...` | direct | **timeout (>60s)** | VERIFIED-LIVE |
| `https://home.treasury.gov/news/press-releases` | **Jina** | **HTTP 200** — full press-release index: dated titles + `/news/press-releases/sbXXXX/` links (newest 2026-08-28) | VERIFIED-LIVE |
| `https://home.treasury.gov/policy-issues/financing-the-government/quarterly-refunding` | **Jina** | **HTTP 200** — Quarterly Refunding hub (refunding statements, TBAC, primary dealers, webcasts) | VERIFIED-LIVE |
| `https://api.fiscaldata.treasury.gov/services/api/fiscal_service/v2/accounting/od/debt_to_penny?sort=-record_date` | direct | **HTTP 200 JSON** — latest 2026-08-27, total public debt outstanding $40,077,529,831,942.94; **no API key** | VERIFIED-LIVE |
| `https://api.fiscaldata.treasury.gov/.../v2/accounting/od/avg_interest_rates?sort=-record_date` | direct | **HTTP 200 JSON** — latest record 2026-07-31 (e.g. T-Bills 3.758%, FRN 3.948%); 4,993 records; **no API key** | VERIFIED-LIVE |
| `https://www.treasurydirect.gov/TA_WS/securities/auctioned?type=Note&format=json` | direct | **HTTP 200 JSON** — 40 records; most recent auction 2026-08-27, CUSIP 91282CRJ2, full bid-to-cover / allotment / pricing stats; **no API key** | VERIFIED-LIVE |

#### P.3.2 Corroboration

- `VERIFIED-LIVE` — the Jina-rendered press-release index contains **no RSS/Atom
  link**. The only subscription mechanism offered is **GovDelivery email**
  (`https://public.govdelivery.com/accounts/USTREAS/subscriber/new?topic_id=USTREAS_49`).
  This confirms Section 8's warning against assuming a historical Treasury
  press-RSS path is still valid — there is no confirmed machine feed for Treasury
  press releases.
- `CORROBORATED` — the Fiscal Data API (`api.fiscaldata.treasury.gov`) is
  documented as fully public, **no API key, no rate limit stated** beyond fair use;
  it exposes dozens of datasets (debt, cash balance, average interest rates,
  auction summary, exchange rates). The TreasuryDirect `TA_WS` auction web service
  is likewise public and keyless.

#### P.3.3 Finding

Treasury is reachable, but **by route, not as one site**:

| Treasury need | Working route (this session) | Key? | Robustness |
|---|---|---|---|
| Debt / cash / **average interest rates on the public debt** | `api.fiscaldata.treasury.gov` (direct JSON) | No | High — real API, own host |
| **Auction results + upcoming auction schedule** (supply) | `www.treasurydirect.gov/TA_WS/` (direct JSON) | No | High — real API, own host |
| **Press releases** (policy, sanctions, personnel) | `r.jina.ai/https://home.treasury.gov/news/press-releases` | No | **Medium** — Jina only; direct times out; no native feed; must paginate/scrape the rendered index |
| **Quarterly Refunding Announcement (QRA)** / debt-management statements | `r.jina.ai/https://home.treasury.gov/policy-issues/financing-the-government/quarterly-refunding` | No | Medium — Jina only |
| Daily par yield curve (rates) | `home.treasury.gov` XML/CSV **times out**; use Fiscal Data `avg_interest_rates` **or** an operator re-test | — | Low direct; substitute exists |

**Direct access to `home.treasury.gov` is not viable from this environment** (four
distinct paths, all timed out at 60s). The scanner's Jina path works for the
narrative pages; the structured APIs are on *different, reachable hosts* and
should be preferred wherever the datum exists in both.

#### P.3.4 Residual risk

- **No native Treasury press feed** → the scanner must detect new press releases by
  polling the Jina-rendered index and diffing against seen `/sbXXXX/` slugs. More
  fragile than an RSS `<guid>` diff; needs its own dedup (Session 1 item 15).
- `home.treasury.gov` timeout may be environment-specific (CDN geo, egress IP).
  **Operator must re-test from the production host** — direct access there would
  materially de-risk the press/QRA path.
- GovDelivery email is a possible fallback for press-release *notification* but is
  email-plumbing, not a clean scanner input; note only, not recommended.

#### P.3.5 Remaining operator steps

1. Re-test `https://home.treasury.gov/news/press-releases` and the daily
   yield-curve XML endpoint directly from the production machine.
2. On the live press-releases page, inspect the HTML `<head>` for any
   `<link rel="alternate" type="application/rss+xml">` a rendered-markdown view
   would strip; record the URL if present.
3. Decide press-release ingestion mechanism: (a) Jina index poll + slug diff
   (recommended interim), (b) GovDelivery, (c) native feed if (2) finds one.
4. Pick the rates source: Fiscal Data `avg_interest_rates` (works now) vs. the
   `home.treasury.gov` par-yield-curve XML (needs a working direct route).

---

## DELIVERABLE Q — RECOMMENDED INITIAL CRITICAL-SOURCE SET (N.12)

**Framing:** this is a **recommendation of candidates**, not an activation. Every
entry still requires the Section 8 verification gate and the N.13 terms-of-use /
automation check before it may be marked active. No registry exists and none was
created. The set is deliberately small, all Tier-1 (primary official issuer),
needs **no credentials beyond one free BLS API key**, and is sufficient to build
and prove the first vertical slice (Session 1 §O).

### Q.1 — Candidate table

| # | Source (issuer) | What it covers | Retrieval route (verified) | Data shape | Cadence | Credential | Auth tier | Known failure mode |
|---|---|---|---|---|---|---|---|---|
| 1 | **Federal Reserve — press (monetary)** | FOMC statements, FOMC minutes, discount-rate minutes, policy implementation notes | `federalreserve.gov/feeds/press_monetary.xml` (**direct RSS 200**); full text per item via `r.jina.ai/…` (**200**) | RSS items → Jina prose | event-driven; poll ≤15 min around 14:00 ET on FOMC days, hourly otherwise | none | Tier-1 | feed lag of a few minutes vs. the wire; mitigate by also polling the FOMC calendar page around known times |
| 2 | **Federal Reserve — press (all)** | superset of #1 + enforcement, orders, Board actions, speeches index | `federalreserve.gov/feeds/press_all.xml` (**direct RSS 200**) | RSS items → Jina prose | hourly | none | Tier-1 | noisier; filter by release type |
| 3 | **Federal Reserve — FOMC calendar** | meeting dates, SEP flags, statement/minutes/press-conference schedule (through 2027) | `federalreserve.gov/monetarypolicy/fomccalendars.htm` (**direct 200**) | HTML → parse | weekly refresh; daily in a meeting week | none | Tier-1 | layout changes; parse defensively |
| 4 | **BLS — Public Data API** | CPI-U `CUUR0000SA0`, Core CPI `CUUR0000SA0L1E`, Total Nonfarm Payrolls `CES0000000001`, unemployment rate `LNS14000000` (+ others as needed) | `api.bls.gov/publicAPI/v2/timeseries/data/…` (**direct JSON 200**) | JSON series → compute MoM/YoY | check within minutes of the scheduled 8:30 ET release on CPI / Employment Situation days; else daily | **free API key** (recommended) | Tier-1 | unregistered quota is small → 429; shutdown gaps return `"-"` |
| 5 | **BLS — release schedule + narrative** | exact release datetimes; release prose (context the API lacks) | `r.jina.ai/https://www.bls.gov/schedule/2026/home.htm` and `…/news.release/<rel>.nr0.htm` (**Jina 200**; direct = 403) | HTML → prose | schedule: monthly; narrative: on release day | none | Tier-1 | Jina could be blocked upstream → schedule is also hand-maintainable a year out |
| 6 | **BEA — news RSS** | Personal Income & Outlays (**PCE**, the Fed's preferred gauge) and GDP releases | `apps.bea.gov/rss/rss.xml` (**direct RSS 200**) | RSS items → Jina prose | check around 8:30 ET on scheduled days; else daily | none | Tier-1 | single combined feed (62 items) → filter by title |
| 7 | **BEA — release schedule** | forward PCE + GDP release dates/times (through Dec 2026 verified) | `bea.gov/news/schedule` (**direct 200**) | HTML → parse | weekly | none | Tier-1 | layout changes |
| 8 | **TreasuryDirect — auction API** | auction results + **upcoming auction schedule** (bill/note/bond supply, bid-to-cover, tails) | `treasurydirect.gov/TA_WS/securities/{auctioned,upcoming}?format=json` (**direct JSON 200**) | JSON | daily; more often on auction days | none | Tier-1 | none observed |
| 9 | **Treasury — Fiscal Data API** | average interest rates on the public debt, debt to the penny, operating cash balance | `api.fiscaldata.treasury.gov/services/api/fiscal_service/…` (**direct JSON 200**) | JSON | daily | none | Tier-1 | endpoint path/version drift (`v1` vs `v2`); pin per dataset |
| 10 | **Treasury — press releases / QRA** | debt-management (Quarterly Refunding), sanctions, Secretary remarks | `r.jina.ai/https://home.treasury.gov/news/press-releases` and `…/quarterly-refunding` (**Jina 200**; direct = timeout) | HTML index → slug diff → Jina prose | hourly | none | Tier-1 | **no native feed**; direct host unreachable here → **mark DEGRADED** until operator re-test or a feed is found |
| — | **Reuters** | — | — | — | — | — | — | **EXCLUDED** — see P.2 |

### Q.2 — Minimal first-slice subset

For the single-source vertical slice in Session 1 §O, any one of #1, #4, or #6 is
sufficient. **Recommended: #1 (Fed `press_monetary.xml`)** — a real RSS feed with
stable `<guid>`s (clean dedup demo), Tier-1 authority, full text retrievable via
the already-verified `WebChannel.read()` path, and directly on-topic for the
scanner's "Monetary policy and rates" evidence category.

### Q.3 — Why these and not others

- **All-official, all-Tier-1.** Every candidate is the primary issuer of its data;
  no aggregators, no wires, no secondary reporting. Removes source-authority
  ambiguity (Session 1 item 18) from the first build.
- **Credential-light.** Only BLS asks for a (free) key. Nothing here needs the
  LLM provider (N.5), consensus provider (N.1), or market-data provider (N.16–19)
  — so N.12 does not block on those.
- **Covers the core premarket macro surface:** monetary policy (Fed), the two
  market-moving BLS prints (CPI, jobs), the Fed's preferred inflation gauge (BEA
  PCE), growth (BEA GDP), and rates supply/technicals (Treasury auctions + average
  rates). That is the minimum viable "environment" for an NQ premarket read.

### Q.4 — BLS series IDs proposed (for the N.13 gate to confirm)

`CUUR0000SA0` (CPI-U all items, NSA), `CUUR0000SA0L1E` (Core CPI, NSA),
`CES0000000001` (Total Nonfarm Payrolls, level), `LNS14000000` (unemployment
rate), and optionally `CES0500000003` (avg hourly earnings) and `LNS11300000`
(labor-force participation). All are single-series pulls; a full run fits well
under the registered 50-series/request ceiling.

---

## DELIVERABLE R — ROUTE-PATTERN FINDINGS (cross-cutting)

### R.1 — Jina Reader is the load-bearing retrieval path — and it has one hole

`r.jina.ai` (i.e. `WebChannel.read()`) succeeded this session for **federalreserve.gov,
bls.gov (bypassing the direct 403), and home.treasury.gov (bypassing the direct
timeout)** — three of the four target families. It **failed for Reuters (403)**.
Implication: the scanner's primary retrieval path is validated against the
institutional universe it actually needs, but it is a **single point of
dependency**. Where a structured API carries the same datum (BLS numbers,
Treasury rates/auctions), that API should be primary and Jina the fallback — not
the other way round.

### R.2 — Prefer structured government APIs over page scraping

Three keyless JSON APIs worked **directly**, from a datacenter IP, with no
anti-bot friction:

- `api.bls.gov/publicAPI/v2/` — labor & price series
- `api.fiscaldata.treasury.gov/services/api/fiscal_service/` — debt, cash, rates
- `www.treasurydirect.gov/TA_WS/` — auctions

These are the most robust inputs in the whole candidate set: real APIs, stable
schemas, own hostnames, honest about data gaps. Every datum available from both an
API and a web page should be taken from the API; the page is for narrative only.

### R.3 — Direct access to `www.bls.gov` and `home.treasury.gov` is unreliable from non-residential infra

`www.bls.gov` → uniform `403` (Akamai). `home.treasury.gov` → uniform timeout.
Both are consistent with the Section 25 / N.8 "datacenter restriction" note and
should be assumed hostile to the scanner's direct path **until proven otherwise on
the production host**. `federalreserve.gov`, `apps.bea.gov`, `bea.gov`,
`api.bls.gov`, `api.fiscaldata.treasury.gov`, and `treasurydirect.gov` had **no
such problem**.

### R.4 — Feed availability is uneven

| Issuer | Native machine feed? |
|---|---|
| Federal Reserve | **Yes** — `press_all.xml`, `press_monetary.xml`, others (RSS 2.0, stable guids) |
| BEA | **Yes** — `apps.bea.gov/rss/rss.xml` (one combined feed) |
| BLS | **No RSS reachable** — `/feed/*.rss` is 403; use the API + Jina schedule instead |
| Treasury | **No** — press releases have only GovDelivery email; APIs cover data but not press |

The scanner needs **two ingestion styles**: RSS-guid diffing (Fed, BEA) and
index-page slug diffing (Treasury press), plus scheduled API polling (BLS, Treasury
data) keyed off the verified release calendars.

---

## DELIVERABLE S — UPDATED STATUS OF N.8 / N.9 / N.10 / N.12

| Item | Session 1 status | **Session 2 status** | Precise remaining blocker |
|---|---|---|---|
| **N.8 — BLS reachability** | open; not tested; datacenter-restriction concern | **RESOLVED — routes identified.** `api.bls.gov` (direct JSON, primary) + Jina for schedule/narrative both verified working. Direct `www.bls.gov` = 403 (expected-to-fail). | (a) operator re-tests direct `www.bls.gov` from the production host; (b) operator registers the free API key. Neither blocks the first slice. |
| **N.9 — Reuters route** | open; Section 8 warns endpoints may be gone | **RESOLVED — exclude from V1.** No official RSS (dead since 2020); direct blocked; Jina route = 403. No viable fixed-source route. | Owner ratifies exclusion (recommended), or commissions a terms review for an Exa-discovery-only, no-retrieval, low-authority pattern. Does not block anything. |
| **N.10 — Treasury routes** | open; Section 8 warns press-RSS paths may be invalid | **PARTIALLY RESOLVED.** Data APIs (Fiscal Data, TreasuryDirect auctions) verified working **direct**, keyless. Press releases / QRA reachable **via Jina only**; direct `home.treasury.gov` times out; **no native press feed exists** (GovDelivery email only). | (a) operator re-tests `home.treasury.gov` direct from the production host; (b) confirm/deny a native press feed via live `<head>` inspection; (c) owner picks the press-ingestion mechanism (Jina index poll recommended interim). Data-side is unblocked now. |
| **N.12 — Initial critical-source set** | open; needs a small verified set chosen | **RESOLVED — recommended candidate set delivered** (Deliverable Q: Fed monetary + all press, FOMC calendar, BLS API + schedule, BEA RSS + schedule, TreasuryDirect + Fiscal Data APIs; Treasury press marked DEGRADED; Reuters excluded). First-slice pick: Fed `press_monetary.xml`. | Owner **activates** the set and runs the N.13 terms-of-use / automation check per source — an owner decision, deliberately not made here. |

---

## OPEN ITEMS HANDED BACK (owner / future sessions)

1. **N.13 terms-of-use / automation review** for each activated source. U.S.
   federal government works (Fed, BLS, BEA, Treasury) are generally public-domain
   with permissive automated-access posture, but the BLS API terms and the Fiscal
   Data / TreasuryDirect terms should each be read and recorded before activation.
   Not performed this session.
2. **Operator-machine re-test** of every direct-path failure in Deliverable P
   (`www.bls.gov` 403; `home.treasury.gov` timeout). Results may differ on the
   production network and would restore direct fallbacks.
3. **Native Treasury press feed** — confirm or rule out via live HTML `<head>`
   inspection of `home.treasury.gov/news/press-releases`.
4. **BLS API key provisioning** — free, but must exist before a real run
   (relates to N.24).
5. **Nothing here decides** N.1 (consensus provider), N.3/N.4 (OS / scheduler),
   N.5/N.6 (LLM provider/model), N.7 (LLM credential storage), N.14/N.15
   (Deep-run budget), N.16–N.21 (instrument scope), N.22 (storage root), N.23
   (report delivery). Those remain exactly as Session 1 left them.

---

## RESTRICTIONS HONORED

No scanner code was written. No Agent Reach source, config, or test was modified.
No production scanner files, directories, schemas, or databases were created. No
owner decision was made — source activation, terms sign-off, and set composition
remain with the owner. All verification was read-only HTTP GET. The only file
created by this session is this document, `planning/SESSION_02_SOURCE_VERIFICATION.md`.

---

**End of Session 2 deliverable package (P, Q, R, S).** Per the master prompt's
session discipline, this session stops here and awaits the next instruction — no
sources were activated, no code was produced, and no owner-sensitive decisions
were made.
