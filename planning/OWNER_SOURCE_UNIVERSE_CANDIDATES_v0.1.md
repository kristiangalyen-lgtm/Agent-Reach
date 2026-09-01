# NQ Premarket Environmental Intelligence Scanner
## Owner Source Universe Intake — Candidate Document v0.1

**This is not Session 6.** This document is an owner-controlled *intake* of
candidate information sources for later classification, verification, terms
review, prioritization, and possible activation. It was prepared after
reading Sessions 1–5 in full (`SESSION_01_ARCHITECTURE_FOUNDATION.md` through
`SESSION_05_CANONICAL_STATE_AND_PUBLICATION_INTEGRITY.md`).

---

## 1. Purpose and Non-Activation Warning

The purpose of this document is to record, in one place, every information
source and market observation the owner wants *considered* for the scanner's
eventual source registry — nothing more.

**Listing a source in this document does not mean:**

- it is reachable,
- automated retrieval is permitted,
- its terms are cleared,
- it is approved,
- it is active,
- it is production-ready,
- or its information qualifies as evidence.

No source in this document is activated. No source's formal Session 3
lifecycle state is assigned or changed by this document (Section 3.2). No
source's terms-review status is marked `CLEARED` by this document (Section
3.3). Every entry remains a **candidate** until a future classification,
verification, terms-review, and owner-approval session says otherwise.

---

## 2. Owner Workflow Context

The owner's current morning process typically uses:

- a small number of Yahoo Finance articles,
- selected posts on X,
- Bloomberg Television on YouTube,
- and an AI tool to organize the morning environment.

The target scanner is meant to preserve the useful context from this workflow
while replacing unsupported assumptions with a structured hierarchy. The
owner specified seven target principles; each is annotated below with the
prior-session vocabulary it maps onto, so this document does not invent
parallel rules where one already exists.

| # | Owner principle | Maps onto |
|---|---|---|
| 1 | Official system-of-record sources establish facts | Session 3 Tier 1 (primary official issuer); Session 5 §0.1 item 8 — a Tier-1 source may establish its own official datum without artificial cross-issuer corroboration (`FACT_ESTABLISHED_BY_SYSTEM_OF_RECORD`) |
| 2 | Structured data preferred over page scraping | Session 2 Deliverable R.2 — "prefer structured government APIs over page scraping"; Session 3 `robustness_class` ranking (`STRUCTURED_API` > `NATIVE_FEED` > `INDEX_PAGE` > `PROXY_OF_BLOCKED`) |
| 3 | Market observations measure reaction and confirmation | Session 3 `MARKET_OBSERVATION` role; Session 4 `MEASURED_MARKET_OBSERVATION` evidence type |
| 4 | Bloomberg TV and Yahoo Finance provide attributed context and discovery | Session 3 Tier 3 (independent reputable reporting); Session 4 §G.1 eligibility gate — Tier 3 counts as coverage/corroboration, never sole support, never feeds directional/regime alone |
| 5 | Exa and future X sources provide discovery and contradiction leads only | Session 3 Tier 4 / `OPEN_WEB`; Session 4 §G.1 — Tier 4 never counts as coverage, may only seed contradiction search; Session 3 `CONTRADICTION_SEEKING` role |
| 6 | AI synthesizes validated evidence but is never an original source | Session 4 §A.4 — "raw model output that is not a well-formed typed `ExtractedClaim`" is dropped, never becomes evidence; §E.1 — the LLM may propose, never decide |
| 7 | Deterministic validation constrains AI extraction and synthesis | Session 4 §A.5 — extraction (Boundary 4) is the *only* non-deterministic stage; everything downstream is a pure function |

---

## 3. Source-Selection Philosophy

### 3.1 Independent axes

Per prior-session architecture, this document tracks **seven independent
axes** per candidate. None of these collapse into another — a `CORE`
priority does not imply Tier 1, a Wave 1 assignment does not imply `ACTIVE`,
and a `VERIFIED_LIVE_SESSION_2_INFRA` result does not imply owner approval.

| Axis | Vocabulary used here | Status |
|---|---|---|
| Owner priority | `CORE` / `USEFUL` / `OPTIONAL` / `LATER` | New in this document — not present in Sessions 1–5; proposed for ratification |
| Source authority | Tier 1 / Tier 2 / Tier 3 / Tier 4 | From Session 3 §U.2 — **proposed, unratified**, assigned per §U.7's rule that tier is an owner/architect judgment, never runtime-inferred |
| Source function / role | `PRIMARY`, `CALENDAR`, `MARKET_OBSERVATION`, `CORROBORATING`, `CONTRADICTION_SEEKING` (closed Session 3 vocabulary) plus document-level `candidate_function = DISCOVERY` (see §3.4) | Reused from Session 3 where possible; `DISCOVERY` is explicitly *not* a registry role (see §3.4) |
| Rollout wave | Wave 1–5 | New in this document — a sequencing axis, distinct from lifecycle state |
| Technical verification confidence | `VERIFIED_LIVE_SESSION_2_INFRA` / `CORROBORATED` / `CANDIDATE_UNTESTED` / `ASSUMED` / `PRODUCTION_HOST_RETEST_REQUIRED` | Extends Session 2's `VERIFIED-LIVE` / `CORROBORATED` / `ASSUMED` labels with two additions needed for honesty about infra provenance (see §3.5) |
| Terms-review status | `NOT_YET_REVIEWED` / `PENDING` / `FAILED` (never `CLEARED` in this document) | Session 3 `terms_review_status` enum, restricted per owner instruction (see §3.3) |
| Future lifecycle state | **Not assigned in this document** | Session 3 §V.1 (`DISCOVERED → … → ACTIVE`) is explicitly out of scope here — see §3.2 |

### 3.2 Lifecycle handling

This document does **not** assign or change any source's formal Session 3
lifecycle state (`DISCOVERED`, `UNDER_VERIFICATION`, `REQUIRES_TERMS_REVIEW`,
`VERIFIED_READY`, `ACTIVE`, `ACTIVE_DEGRADED`, `SUSPENDED`, `DISABLED`,
`RETIRED`). In particular, sources that Session 2 already live-verified are
**not** regressed to `DISCOVERED` — this document simply does not speak in
that vocabulary at all. Instead, each candidate below carries these
document-local fields:

- **candidate status** — is this source new to the universe, or does it
  carry forward prior-session findings?
- **inherited verification evidence** — what, specifically, did a prior
  session already establish about this source (if anything)?
- **verification confidence** — see §3.5.
- **production-host re-test status** — has this route been confirmed from
  the owner's actual production host, or only from Session 2's non-production
  infra?
- **terms-review status** — see §3.3.
- **remaining blockers** — what has to happen before this candidate could
  even enter formal Session 3 lifecycle tracking.

Formal lifecycle assignment remains for a later classification,
verification, terms-review, and owner-approval process — not this document.

### 3.3 Terms-review handling

No candidate in this document is marked `NOT_REQUIRED` merely because it is
public-domain, official, government-hosted, or unauthenticated. Public-domain
status and automated-access clearance are separate questions (Session 4 §C4:
"public-domain status does not automatically clear automated-access
requirements"). Unless a prior session recorded a *completed*,
source-specific terms determination — and none did, per Session 2's own
statement that N.13 review "was not performed this session" for any
candidate, including BLS, Fiscal Data, and TreasuryDirect — every source here
is marked `NOT_YET_REVIEWED`. No candidate is marked `PENDING` unless a
review is already known to be in progress (none are, so `NOT_YET_REVIEWED` is
the uniform default throughout this document). No candidate is marked
`CLEARED` by this document, under any circumstance.

### 3.4 Discovery handling

`DISCOVERY` is **not** treated as an approved addition to Session 3's closed
`source_roles.yaml` vocabulary (`PRIMARY`, `CORROBORATING`,
`CONTRADICTION_SEEKING`, `CALENDAR`, `MARKET_OBSERVATION`). Where a candidate
serves a discovery function (Exa, and any future owner-curated X sources),
this document records it as:

- `candidate_function = DISCOVERY` — a document-level descriptive label only,
- **possible future registry role** — `CONTRADICTION_SEEKING` or
  `CORROBORATING`, whichever a later session determines fits once the
  candidate is actually evaluated.

Whether to add a formal `DISCOVERY` role to Session 3's registry vocabulary
is left as an **unresolved owner decision** (see §8).

### 3.5 Verification confidence labels

| Label | Meaning |
|---|---|
| `VERIFIED_LIVE_SESSION_2_INFRA` | A live GET (or equivalent) was actually performed against this route during Session 2, from Claude Code's datacenter infrastructure — **not** the operator's eventual production host |
| `CORROBORATED` | Supported by a WebSearch or independent secondary check, not a direct live retrieval |
| `CANDIDATE_UNTESTED` | Named in the owner's candidate list or known-route inventory; no session has attempted retrieval against it |
| `ASSUMED` | A property is stated (e.g., a terms-of-service posture, a feed's stability) without any test having been run to confirm it |
| `PRODUCTION_HOST_RETEST_REQUIRED` | Applies alongside `VERIFIED_LIVE_SESSION_2_INFRA` on every such entry — Session 2 explicitly flagged its own results as needing re-verification from the operator's real network before being treated as final |

A Session 2 infrastructure result is never represented as a verification
performed from the operator's production host.

### 3.6 Evidence-eligibility reminder (Session 4 §G.1, unratified)

For reference only — no candidate below is being declared evidence-eligible
by this document:

- Tier 1–2 sources may be sole support and may feed deterministic
  directional/regime inputs.
- Tier 3 sources (e.g., Bloomberg TV, Yahoo Finance) may count as coverage,
  corroboration, and concentration, but **never** as sole support and
  **never** feed directional/regime assessment alone.
- Tier 4 sources (open web, Exa results, any future X material) **never**
  count as coverage; they may only seed contradiction search.

---

## 4. Candidate Source Tree by Function

Column legend used in every table below: **Priority** = owner priority
(§3.1); **Fn/Role** = source function/role (§3.1, §3.4); **Tier** = proposed
authority tier, unratified; **Wave** = proposed rollout wave (§6); **Cadence**
= `EVENT_DRIVEN` / `SCHEDULED_RELEASE` / `DAILY` / `INTRADAY`; **Route Kind**
= `STRUCTURED_API` / `NATIVE_RSS` / `NATIVE_ATOM` / `INDEX_PAGE` /
`JINA_READER` / `FILE_DROP`; **Confidence** = §3.5 label(s); **Terms** =
§3.3 label (uniformly `NOT_YET_REVIEWED` unless noted).

### 4.1 Calendar and Event Backbone (cross-reference index)

Each calendar candidate below has exactly one canonical entry, located under
its issuing family's own subsection (§4.2–§4.9). This subsection is an index
only — it does not duplicate the full row.

| Calendar | Canonical entry |
|---|---|
| Federal Reserve FOMC calendar | §4.2 (Federal Reserve Board) |
| BLS release calendar | §4.4 (Inflation and Labor) |
| BEA release schedule | §4.5 (Growth, Spending, and PCE) |
| Treasury auction calendar | §4.6 (Treasury Supply and Fiscal Conditions) |
| CME economic-release calendar | §4.4 (listed as a corroborating aggregator of the above; not an issuer in its own right) |
| Earnings calendar | §4.7 (Corporate and Nasdaq-100 Concentration) |
| Official IR calendars for important Nasdaq-100 companies | §4.7 (Corporate and Nasdaq-100 Concentration) |

**Candidate status:** new framing (the cross-reference structure); the
underlying calendars themselves are not new — see each canonical entry for
inherited verification evidence.

### 4.2 Federal Reserve Board

**Purpose:** Tier-1 system-of-record for U.S. monetary policy — decisions,
minutes, projections, speeches, testimony, and balance-sheet/rate releases.

**Information expected:** FOMC statements and minutes, Summary of Economic
Projections, press-conference occurrence and materials, speech/testimony
text, Beige Book, Monetary Policy Report, H.4.1 (balance sheet), H.15
(interest rates), meeting calendar.

| Candidate | Route(s) | Priority | Fn/Role | Tier | Wave | Cadence | Route Kind | Confidence | Terms |
|---|---|---|---|---|---|---|---|---|---|
| Monetary-policy RSS | `federalreserve.gov/feeds/press_monetary.xml` | CORE | PRIMARY | 1 | 1 | EVENT_DRIVEN | NATIVE_RSS | `VERIFIED_LIVE_SESSION_2_INFRA` + `PRODUCTION_HOST_RETEST_REQUIRED` | NOT_YET_REVIEWED |
| All-press-release RSS | `federalreserve.gov/feeds/press_all.xml` | USEFUL | PRIMARY | 1 | 1 | EVENT_DRIVEN | NATIVE_RSS | `VERIFIED_LIVE_SESSION_2_INFRA` + `PRODUCTION_HOST_RETEST_REQUIRED` | NOT_YET_REVIEWED |
| Feed index page | `federalreserve.gov/feeds/feeds.htm` | OPTIONAL | CALENDAR | 1 | 1 | DAILY | INDEX_PAGE | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| FOMC calendar | `federalreserve.gov/monetarypolicy/fomccalendars.htm` | CORE | CALENDAR | 1 | 1 | SCHEDULED_RELEASE | INDEX_PAGE | `VERIFIED_LIVE_SESSION_2_INFRA` + `PRODUCTION_HOST_RETEST_REQUIRED` | NOT_YET_REVIEWED |
| Monetary policy hub | `federalreserve.gov/monetarypolicy.htm` | USEFUL | CALENDAR | 1 | 1 | DAILY | INDEX_PAGE | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| Speeches | `federalreserve.gov/newsevents/speeches.htm` | USEFUL | PRIMARY | 1 | 3 | EVENT_DRIVEN | INDEX_PAGE | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| Testimony | `federalreserve.gov/newsevents/testimony.htm` | USEFUL | PRIMARY | 1 | 3 | EVENT_DRIVEN | INDEX_PAGE | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| Beige Book | (via monetary policy hub) | OPTIONAL | PRIMARY | 1 | 3 | SCHEDULED_RELEASE | INDEX_PAGE | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| Monetary Policy Report | (via monetary policy hub) | OPTIONAL | PRIMARY | 1 | 3 | SCHEDULED_RELEASE | INDEX_PAGE | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| H.4.1 balance sheet | `federalreserve.gov/releases/h41/` | USEFUL | PRIMARY | 1 | 3 | SCHEDULED_RELEASE | INDEX_PAGE | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| H.15 interest rates | `federalreserve.gov/releases/h15/` | USEFUL | PRIMARY | 1 | 3 | SCHEDULED_RELEASE | INDEX_PAGE | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| FOMC press conferences (video/transcript) | (via monetary policy hub) | OPTIONAL | PRIMARY | 1 | 3 | EVENT_DRIVEN | INDEX_PAGE | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| Official YouTube channel | `youtube.com/federalreserve` | LATER | PRIMARY | 1 | 3 | EVENT_DRIVEN | FILE_DROP (metadata/caption retrieval, not continuous stream) | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |

**Inherited verification evidence:** Session 2 performed live GETs against
`press_monetary.xml`, `press_all.xml`, and the FOMC calendar page (all
HTTP 200, direct, from datacenter infra), and confirmed full item text is
retrievable via Jina. Session 2 recommended `press_monetary.xml` as the
single first-vertical-slice source (clean `<guid>` dedup, Tier-1 authority,
already-verified full-text retrieval path). No other row in this table was
tested.

**Known failure modes / concerns:** none observed for the two tested feeds
beyond ordinary feed lag of a few minutes. Untested rows (speeches,
testimony, Beige Book, H.4.1/H.15, video) carry no failure-mode evidence
either way — simply unverified.

**Overlap/duplication notes:** `press_all.xml` is a superset of
`press_monetary.xml` (also includes enforcement actions, orders, and
speeches) — noisier; the two are not independent corroborators of the same
fact, they are the same issuer's overlapping feeds (Session 3 §U.6 issuer
concentration logic would treat them as one `FEDERAL_RESERVE` concentration
unit).

**Remaining blockers:** terms review (N.13) not yet performed for any Fed
route; production-host re-test not yet performed for the two verified feeds
and the calendar page; every other row needs a first live-retrieval attempt.

### 4.3 New York Federal Reserve

**Purpose:** Tier-1 system-of-record for U.S. money-market and
balance-sheet-operations data — rates, repo activity, SOMA holdings,
dealer statistics.

**Information expected:** SOFR, EFFR, repo/reverse-repo operations, SOMA
holdings, primary-dealer statistics, funding-market conditions, securities
lending, central-bank liquidity arrangements.

| Candidate | Route(s) | Priority | Fn/Role | Tier | Wave | Cadence | Route Kind | Confidence | Terms |
|---|---|---|---|---|---|---|---|---|---|
| Markets Data Hub | `newyorkfed.org/markets/data-hub/` | USEFUL | PRIMARY | 1 | 2 | DAILY | INDEX_PAGE | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| Markets API docs | `markets.newyorkfed.org/static/docs/markets-api.html` | USEFUL | PRIMARY | 1 | 2 | DAILY | INDEX_PAGE | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| SOMA RSS | `markets.newyorkfed.org/rss/feeds/soma` | OPTIONAL | PRIMARY | 1 | 2 | SCHEDULED_RELEASE | NATIVE_RSS | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| SOMA summary CSV | `markets.newyorkfed.org/api/soma/summary.csv` | USEFUL | PRIMARY | 1 | 2 | SCHEDULED_RELEASE | STRUCTURED_API | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| SOFR reference rate | `newyorkfed.org/markets/reference-rates/sofr` | CORE | MARKET_OBSERVATION | 1 | 2 | DAILY | INDEX_PAGE | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| EFFR reference rate | `newyorkfed.org/markets/reference-rates/effr` | CORE | MARKET_OBSERVATION | 1 | 2 | DAILY | INDEX_PAGE | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| SOMA holdings page | `newyorkfed.org/markets/soma-holdings` | USEFUL | PRIMARY | 1 | 2 | SCHEDULED_RELEASE | INDEX_PAGE | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |

**Inherited verification evidence:** none. Session 2 tested only Federal
Reserve *Board*-level feeds (§4.2); the NY Fed was never separately
investigated. Every route above is an untested candidate at strictly lower
confidence than the Board-level entries, despite both being "the Federal
Reserve" in a colloquial sense — do not treat them as equally verified.

**Known failure modes / concerns:** unknown — no attempt made yet to
determine whether `markets.newyorkfed.org` is reachable from non-residential
infra, whether its API requires registration, or whether SOFR/EFFR data
carries same-day or next-day publication lag.

**Overlap/duplication notes:** SOFR and EFFR are both reference-rate outputs
of the same NY Fed markets desk — same `issuer_family` as every other NY Fed
route; also observationally related to (but not the same series as) any
market-data-provider SOFR/EFFR quote the scanner might separately ingest
under §5 — these two representations should not be double-counted as
independent corroborators.

**Remaining blockers:** full route verification (nothing tested), terms
review, and a decision on whether NY Fed enters Wave 2 alongside SEC/Census
or stays with Fed Board in Wave 1 — recorded as Wave 2 here provisionally
(see §6), owner may prefer Wave 1.

### 4.4 Inflation and Labor

**Purpose:** Tier-1 system-of-record for U.S. price and labor-market
statistics.

**Information expected:** headline/core CPI, PPI, nonfarm payrolls,
unemployment rate, labor-force participation, average hourly earnings,
JOLTS, Employment Cost Index, productivity, unit labor costs, import/export
prices, official release schedule and narrative, weekly unemployment claims.

| Candidate | Route(s) | Priority | Fn/Role | Tier | Wave | Cadence | Route Kind | Confidence | Terms |
|---|---|---|---|---|---|---|---|---|---|
| BLS structured API | `api.bls.gov/publicAPI/v2/` | CORE | PRIMARY | 1 | 1 | SCHEDULED_RELEASE | STRUCTURED_API | `VERIFIED_LIVE_SESSION_2_INFRA` + `PRODUCTION_HOST_RETEST_REQUIRED` (4 of 6 series only — see below) | NOT_YET_REVIEWED |
| — series: headline CPI (`CUUR0000SA0`) | via API above | CORE | PRIMARY | 1 | 1 | SCHEDULED_RELEASE | STRUCTURED_API | `VERIFIED_LIVE_SESSION_2_INFRA` + `PRODUCTION_HOST_RETEST_REQUIRED` | NOT_YET_REVIEWED |
| — series: core CPI (`CUUR0000SA0L1E`) | via API above | CORE | PRIMARY | 1 | 1 | SCHEDULED_RELEASE | STRUCTURED_API | `VERIFIED_LIVE_SESSION_2_INFRA` + `PRODUCTION_HOST_RETEST_REQUIRED` | NOT_YET_REVIEWED |
| — series: nonfarm payrolls (`CES0000000001`) | via API above | CORE | PRIMARY | 1 | 1 | SCHEDULED_RELEASE | STRUCTURED_API | `VERIFIED_LIVE_SESSION_2_INFRA` + `PRODUCTION_HOST_RETEST_REQUIRED` | NOT_YET_REVIEWED |
| — series: unemployment rate (`LNS14000000`) | via API above | CORE | PRIMARY | 1 | 1 | SCHEDULED_RELEASE | STRUCTURED_API | `VERIFIED_LIVE_SESSION_2_INFRA` + `PRODUCTION_HOST_RETEST_REQUIRED` | NOT_YET_REVIEWED |
| — series: labor-force participation (`LNS11300000`) | via API above | USEFUL | PRIMARY | 1 | 1 | SCHEDULED_RELEASE | STRUCTURED_API | `CANDIDATE_UNTESTED` — requested by owner, **not** among the four series Session 2 actually confirmed | NOT_YET_REVIEWED |
| — series: average hourly earnings (`CES0500000003`) | via API above | USEFUL | PRIMARY | 1 | 1 | SCHEDULED_RELEASE | STRUCTURED_API | `CANDIDATE_UNTESTED` — requested by owner, **not** among the four series Session 2 actually confirmed | NOT_YET_REVIEWED |
| — JOLTS | via API above (different series IDs, not yet identified) | USEFUL | PRIMARY | 1 | 2 | SCHEDULED_RELEASE | STRUCTURED_API | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| — Employment Cost Index | via API above (different series IDs, not yet identified) | OPTIONAL | PRIMARY | 1 | 2 | SCHEDULED_RELEASE | STRUCTURED_API | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| — Productivity / unit labor costs | via API above (different series IDs, not yet identified) | OPTIONAL | PRIMARY | 1 | 3 | SCHEDULED_RELEASE | STRUCTURED_API | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| — Import/export prices | via API above (different series IDs, not yet identified) | OPTIONAL | PRIMARY | 1 | 3 | SCHEDULED_RELEASE | STRUCTURED_API | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| BLS release schedule | `bls.gov/schedule/news_release/` | CORE | CALENDAR | 1 | 1 | SCHEDULED_RELEASE | JINA_READER (direct blocked — see below) | `VERIFIED_LIVE_SESSION_2_INFRA` + `PRODUCTION_HOST_RETEST_REQUIRED` | NOT_YET_REVIEWED |
| BLS release narratives | `bls.gov/schedule/2026/home.htm` and per-release `.nr0.htm` pages | USEFUL | PRIMARY | 1 | 2 | SCHEDULED_RELEASE | JINA_READER (direct blocked) | `VERIFIED_LIVE_SESSION_2_INFRA` + `PRODUCTION_HOST_RETEST_REQUIRED` | NOT_YET_REVIEWED |
| DOL weekly unemployment claims | route not yet identified | USEFUL | PRIMARY | 1 | 2 | SCHEDULED_RELEASE | (unknown) | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| CME economic-release calendar (corroborating aggregator) | route not yet identified | OPTIONAL | CORROBORATING | 3 | 3 | SCHEDULED_RELEASE | (unknown) | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |

**Inherited verification evidence:** Session 2 confirmed direct
`www.bls.gov` access (RSS, schedule pages, even developer docs) returns a
uniform HTTP 403, identified as an Akamai bot-management block. The
structured `api.bls.gov` endpoint was separately confirmed reachable
directly, unregistered, HTTP 200 — but only for the 4 series listed above.
The 2 additional series the owner has requested here
(`LNS11300000`, `CES0500000003`) were never queried in Session 2 and carry no
live-verification evidence — they are presumed likely to work given the same
API mechanism, but that is an inference, not a test result, and is recorded
as `CANDIDATE_UNTESTED` accordingly. Schedule/narrative pages were confirmed
reachable via Jina only.

**Known failure modes / concerns:** `www.bls.gov` direct access is
structurally blocked (Akamai), independent of registration status — model
this route as expected-to-fail, not as an error condition. The API itself
returns `"-"` with a footnote during government-shutdown gaps rather than
erroring — must be modeled as a normal data state, not a failure. Unregistered
API use is rate-limited; a free registration key raises the ceiling (500
queries/day, 50 series/request, 20-year ranges) but requires an email/org/
CAPTCHA registration step not yet performed.

**Overlap/duplication notes:** the BLS release schedule and BLS release
narratives are both BLS-issued and form a single `BLS` concentration unit
alongside every series pulled from the same API; the CME economic-release
calendar is a secondary aggregator of official releases, not an independent
issuer, and cannot corroborate a BLS/BEA/Fed fact on its own authority.

**Remaining blockers:** BLS API registration decision (owner-facing —
whether to register for the higher quota); live-test the 2 unconfirmed
series; identify JOLTS/ECI/productivity/import-export series IDs; identify a
DOL route for weekly claims; terms review for all BLS/DOL routes; production-
host re-test for every `VERIFIED_LIVE_SESSION_2_INFRA` row.

### 4.5 Growth, Spending, and PCE

**Purpose:** Tier-1 system-of-record for U.S. growth, income/spending, and
trade statistics (BEA), and Tier-1 system-of-record for goods/construction/
trade activity indicators (Census).

**Information expected — BEA:** headline/core PCE, personal income/spending,
GDP and revisions, corporate profits, business investment, international
trade. **Information expected — Census:** retail sales, durable goods,
factory orders, manufacturing shipments/inventories, housing starts/permits,
new-home sales, construction spending, business inventories, wholesale
trade, international trade in goods.

| Candidate | Route(s) | Priority | Fn/Role | Tier | Wave | Cadence | Route Kind | Confidence | Terms |
|---|---|---|---|---|---|---|---|---|---|
| BEA release schedule | `bea.gov/news/schedule` | CORE | CALENDAR | 1 | 1 | SCHEDULED_RELEASE | INDEX_PAGE | `VERIFIED_LIVE_SESSION_2_INFRA` + `PRODUCTION_HOST_RETEST_REQUIRED` | NOT_YET_REVIEWED |
| BEA current releases | `bea.gov/news/current-releases` | USEFUL | PRIMARY | 1 | 1 | SCHEDULED_RELEASE | INDEX_PAGE | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| BEA combined RSS | `apps.bea.gov/rss/rss.xml` | CORE | PRIMARY | 1 | 1 | EVENT_DRIVEN | NATIVE_RSS | `VERIFIED_LIVE_SESSION_2_INFRA` + `PRODUCTION_HOST_RETEST_REQUIRED` | NOT_YET_REVIEWED |
| BEA structured API | `apps.bea.gov/api/data` | USEFUL | PRIMARY | 1 | 2 | SCHEDULED_RELEASE | STRUCTURED_API | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| Census economic-indicators hub | `census.gov/data/developers/data-sets/economic-indicators.html` | USEFUL | PRIMARY | 1 | 2 | DAILY | INDEX_PAGE | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| Census timeseries EITS API | `api.census.gov/data/timeseries/eits.html` | CORE | PRIMARY | 1 | 2 | SCHEDULED_RELEASE | STRUCTURED_API | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |

**Inherited verification evidence:** Session 2 confirmed `apps.bea.gov/
rss/rss.xml` (direct, 200, 62 items, combined feed requiring title-based
filtering — includes both PCE and GDP items) and `bea.gov/news/schedule`
(direct, 200, forward dates verified through December 2026). Census was
never investigated in Session 2 — every Census row above is untested.

**Known failure modes / concerns:** the BEA RSS feed is a single combined
feed across all BEA release types — the scanner will need title/category
filtering to isolate PCE vs. GDP vs. trade items, not a defect but an
integration requirement. Census reachability, rate limits, and whether a
registration key is required are all unknown.

**Overlap/duplication notes:** BEA's own schedule page and RSS feed form one
`BEA` concentration unit; Census indicators are a *different* issuer family
(`US_CENSUS_BUREAU`) and should not be merged with BEA even though both feed
"growth and spending" narratively.

**Remaining blockers:** Census route verification (nothing tested), BEA API
key/registration determination, RSS filtering-logic design (deferred to
implementation, out of scope here), terms review for both issuers,
production-host re-test for BEA's two verified routes.

### 4.6 Treasury Supply and Fiscal Conditions

**Purpose:** Tier-1 system-of-record for U.S. government debt issuance,
cash-balance conditions, and fiscal announcements.

**Information expected:** Treasury securities auctions (announcements,
results, offering amounts, high yield, bid-to-cover, indirect/direct/dealer
participation, SOMA add-ons), Treasury Fiscal Data API, Daily Treasury
Statement, operating cash balance, Debt to the Penny, average interest
rates, Monthly Treasury Statement, Quarterly Refunding Announcement, TBAC
materials, borrowing estimates, Treasury press releases.

| Candidate | Route(s) | Priority | Fn/Role | Tier | Wave | Cadence | Route Kind | Confidence | Terms |
|---|---|---|---|---|---|---|---|---|---|
| Fiscal Data API (general) | `api.fiscaldata.treasury.gov/services/api/fiscal_service/` | CORE | PRIMARY | 1 | 1 | DAILY | STRUCTURED_API | `VERIFIED_LIVE_SESSION_2_INFRA` + `PRODUCTION_HOST_RETEST_REQUIRED` | NOT_YET_REVIEWED |
| Fiscal Data — Debt to the Penny / avg interest rates / operating cash balance | same host as above | CORE | PRIMARY | 1 | 1 | DAILY | STRUCTURED_API | `VERIFIED_LIVE_SESSION_2_INFRA` + `PRODUCTION_HOST_RETEST_REQUIRED` | NOT_YET_REVIEWED |
| Treasury Securities Auctions dataset | `fiscaldata.treasury.gov/datasets/treasury-securities-auctions-data/` | CORE | PRIMARY | 1 | 1 | EVENT_DRIVEN | STRUCTURED_API | `CANDIDATE_UNTESTED` (dataset page itself not directly tested; sibling TreasuryDirect API was) | NOT_YET_REVIEWED |
| TreasuryDirect auction API | `treasurydirect.gov/TA_WS/securities/{auctioned,upcoming}?format=json` | CORE | PRIMARY | 1 | 1 | EVENT_DRIVEN | STRUCTURED_API | `VERIFIED_LIVE_SESSION_2_INFRA` + `PRODUCTION_HOST_RETEST_REQUIRED` | NOT_YET_REVIEWED |
| Quarterly Refunding hub | `home.treasury.gov/policy-issues/financing-the-government/quarterly-refunding` | USEFUL | PRIMARY | 1 | 2 | SCHEDULED_RELEASE | JINA_READER (direct times out) | `CANDIDATE_UNTESTED` — hub page itself not directly tested, though sibling press-release path was found Jina-only | NOT_YET_REVIEWED |
| Treasury press releases | `home.treasury.gov/news/press-releases` | USEFUL | PRIMARY | 1 | 2 | EVENT_DRIVEN | JINA_READER (no native feed exists) | `VERIFIED_LIVE_SESSION_2_INFRA` + `PRODUCTION_HOST_RETEST_REQUIRED` | NOT_YET_REVIEWED |
| TBAC materials | via Quarterly Refunding hub | OPTIONAL | PRIMARY | 1 | 3 | SCHEDULED_RELEASE | JINA_READER | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| Monthly Treasury Statement | route not yet identified (likely Fiscal Data API) | OPTIONAL | PRIMARY | 1 | 2 | SCHEDULED_RELEASE | STRUCTURED_API (assumed) | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| Daily Treasury Statement | route not yet identified (likely Fiscal Data API) | USEFUL | PRIMARY | 1 | 1 | DAILY | STRUCTURED_API (assumed) | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |

**Inherited verification evidence:** Session 2 confirmed
`api.fiscaldata.treasury.gov` and `treasurydirect.gov/TA_WS` are both
direct, keyless, HTTP 200. Session 2 separately confirmed Treasury press
releases have **no native RSS/Atom feed** (confirmed by inspecting the
Jina-rendered index page for an RSS link and finding none — GovDelivery
email is the only subscription option) and are reachable only via Jina;
direct `home.treasury.gov` requests (press releases, a guessed RSS path, and
a daily-rates CSV) all timed out (>60s) across multiple attempts. This is
preserved here exactly as Session 2 found it: **Treasury press retrieval is
currently degraded / Jina-dependent**, while Treasury's structured data APIs
(Fiscal Data, TreasuryDirect) are the preferred, currently-working route for
numeric data.

**Known failure modes / concerns:** `home.treasury.gov` direct access is
unreliable from non-residential infrastructure (timeout, not a clean
block/403 — mechanism not fully diagnosed). No feed exists at all for press
releases, so any monitoring approach must use index-page diffing via Jina, not
feed polling. API version drift is a stated risk (v1 vs. v2) requiring
per-dataset version pinning once implemented.

**Overlap/duplication notes:** the Fiscal Data API and TreasuryDirect API are
sibling Treasury-issued structured sources (one `US_TREASURY` concentration
unit); the Quarterly Refunding hub, TBAC materials, and Treasury press
releases are all reached through the same degraded `home.treasury.gov` host
and share the same known failure mode.

**Remaining blockers:** re-test `home.treasury.gov` direct access from the
production host (Session 2's timeout result may not reflect the operator's
real network); identify Monthly/Daily Treasury Statement's actual route;
terms review for both the Fiscal Data and TreasuryDirect APIs and for the
Jina-mediated press-release path.

### 4.7 Corporate and Nasdaq-100 Concentration

**Purpose:** Tier-1 system-of-record for individual company disclosures
(SEC) and official corporate communications (IR pages, earnings materials),
scoped to the owner-approved corporate universe (top 10 current Nasdaq-100
constituents by weight; any additional constituent reporting earnings that
day; any constituent with a material SEC filing, guidance change,
acquisition, regulatory event, or official announcement).

**Information expected:** SEC filings (8-K, 10-Q, 10-K, 6-K, and other
material forms), XBRL structured financial data, company earnings releases,
presentations, prepared remarks, current Nasdaq-100 weights, and the current
day's earnings reporters.

| Candidate | Route(s) | Priority | Fn/Role | Tier | Wave | Cadence | Route Kind | Confidence | Terms |
|---|---|---|---|---|---|---|---|---|---|
| SEC EDGAR submissions API | `data.sec.gov/submissions/CIK##########.json` | CORE | PRIMARY | 1 | 2 | EVENT_DRIVEN | STRUCTURED_API | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| SEC EDGAR XBRL APIs | via `sec.gov/about/developer-resources` | USEFUL | PRIMARY | 1 | 2 | EVENT_DRIVEN | STRUCTURED_API | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| SEC latest-filings feed(s) | via `sec.gov/search-filings/edgar-application-programming-interfaces` | CORE | PRIMARY | 1 | 2 | EVENT_DRIVEN | NATIVE_ATOM (assumed — EDGAR historically exposes Atom full-text/filing feeds; not yet confirmed) | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| Company IR pages (per constituent) | company-specific, not yet enumerated | CORE | PRIMARY | 1 | 2 | EVENT_DRIVEN | INDEX_PAGE (varies) | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| Company IR calendars (per constituent) | company-specific, not yet enumerated | CORE | CALENDAR | 1 | 2 | SCHEDULED_RELEASE | INDEX_PAGE (varies) | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| Official earnings releases/presentations/webcasts | company-specific, not yet enumerated | CORE | PRIMARY | 1 | 2 | EVENT_DRIVEN | INDEX_PAGE (varies) | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| Dynamic Nasdaq-100 weights source | not yet identified — no provider selected | CORE | PRIMARY | 2 (pending source identification) | 2 | DAILY | (unknown) | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| Earnings-calendar aggregator (which constituent reports today) | not yet identified — no provider selected | CORE | CALENDAR | 3 (pending source identification — likely a Tier-3 aggregator, not an issuer) | 2 | DAILY | (unknown) | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |

**Inherited verification evidence:** none. Session 1/2 discuss SEC EDGAR
only as a known route inventory item, never tested live.

**Known failure modes / concerns:** SEC's fair-access policy requires an
identifiable `User-Agent` header and a conservative request rate — this is a
hard constraint on any future implementation, not optional. Per-company IR
pages will vary enormously in structure (no single parser will work across
all ~100 Nasdaq-100 constituents), so the owner's instruction that not every
company be monitored equally at full depth (only the top-10-by-weight plus
same-day earnings reporters plus material-event constituents) is essential
to keeping this tractable.

**Overlap/duplication notes:** SEC EDGAR (submissions API, XBRL, filings
feed) is one `US_SEC` concentration unit regardless of which of the three
routes is used for a given filing; company IR sources are a *separate*
concentration unit per company (`issuer_family` should likely be keyed per
ticker, not collapsed into one "corporate" bucket, so that one company's
three IR pages don't masquerade as three independent corroborators of
another company's news — recorded here as an open question for the registry
design, not resolved by this document).

**Remaining blockers:** no dynamic Nasdaq-100 weights source has been
selected; no earnings-calendar-aggregator source has been selected; SEC
EDGAR routes are entirely untested; per-company IR page inventory has not
been started (100 constituents, not yet enumerated); terms review needed for
SEC EDGAR and for any earnings-calendar aggregator chosen.

### 4.8 Positioning and Slower Context

**Purpose:** weekly, lagged positioning context — explicitly not an intraday
signal.

**Information expected:** CFTC Commitments of Traders and Traders in
Financial Futures — Nasdaq-100 futures, S&P 500 futures, Treasury-futures,
and dollar positioning; dealer/intermediary, asset-manager, and
leveraged-fund positions; weekly changes.

| Candidate | Route(s) | Priority | Fn/Role | Tier | Wave | Cadence | Route Kind | Confidence | Terms |
|---|---|---|---|---|---|---|---|---|---|
| CFTC COT index | `cftc.gov/MarketReports/CommitmentsofTraders/index.htm` | OPTIONAL | CORROBORATING | 1 | 5 | SCHEDULED_RELEASE | INDEX_PAGE | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| CFTC TFF public-reporting dataset | `publicreporting.cftc.gov/Commitments-of-Traders/TFF-Futures-Only/gpe5-46if` | USEFUL | CORROBORATING | 1 | 5 | SCHEDULED_RELEASE | STRUCTURED_API | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |

**Inherited verification evidence:** none.

**Known failure modes / concerns:** none observed (untested); the primary
documented constraint is architectural, not technical — CFTC data is weekly
and lagged, and must never be modeled or presented as an intraday signal.

**Overlap/duplication notes:** both routes are the same issuer (`CFTC`) —
one concentration unit.

**Remaining blockers:** full route verification, terms review, and a
decision on whether this belongs earlier than Wave 5 given its low
implementation complexity (recorded here at Wave 5 per the owner's proposed
waves in §6, but flagged as a candidate for earlier promotion since it is
weekly/low-frequency and technically simple).

### 4.9 International Monetary Policy and Liquidity

**Purpose:** Tier-1/Tier-2 system-of-record for non-U.S. monetary policy and
cross-border liquidity conditions, scoped to ECB and BIS for V1 per the
owner's approved default scope; all other central banks retained as later
candidates only.

**Information expected — ECB:** policy decisions, press releases,
press-conference statements/transcripts, speeches/interviews, statistical
releases, market operations, exchange rates, meeting calendar.
**Information expected — BIS:** central-bank speeches, statistical releases,
global liquidity, international credit, policy rates, central-bank balance
sheets, international banking, debt securities, exchange rates.

| Candidate | Route(s) | Priority | Fn/Role | Tier | Wave | Cadence | Route Kind | Confidence | Terms |
|---|---|---|---|---|---|---|---|---|---|
| ECB RSS hub | `ecb.europa.eu/home/html/rss.en.html` | CORE | CALENDAR | 1 | 3 | DAILY | INDEX_PAGE | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| ECB Governing Council monetary-policy decisions | `ecb.europa.eu/press/govcdec/mopo/html/index.en.html` | CORE | PRIMARY | 1 | 3 | EVENT_DRIVEN | INDEX_PAGE | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| ECB press conferences | `ecb.europa.eu/press/press_conference/html/index.en.html` | USEFUL | PRIMARY | 1 | 3 | EVENT_DRIVEN | INDEX_PAGE | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| ECB reference exchange rates (mid) RSS | `mid.ecb.europa.eu/rss/mid.xml` | USEFUL | MARKET_OBSERVATION | 1 | 3 | DAILY | NATIVE_RSS | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| BIS RSS hub | `bis.org/rss/index.htm` | USEFUL | CALENDAR | 1 | 3 | DAILY | INDEX_PAGE | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| BIS central-bank speeches feed | `bis.org/doclist/cbspeeches.rss` | USEFUL | PRIMARY | 2 (BIS restates/aggregates central-bank speeches rather than being their originating issuer) | 3 | EVENT_DRIVEN | NATIVE_RSS | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| BIS statistics feed | `bis.org/doclist/all_statistics.rss` | USEFUL | PRIMARY | 1 | 3 | SCHEDULED_RELEASE | NATIVE_RSS | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| BIS statistics data portal | `data.bis.org/` | OPTIONAL | PRIMARY | 1 | 3 | SCHEDULED_RELEASE | STRUCTURED_API | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |

**Later international candidates (retained, not V1):** Bank of England,
Bank of Japan, People's Bank of China, Bank of Canada, Reserve Bank of
Australia, Swiss National Bank, IMF, OECD. No routes identified for any of
these yet; priority `LATER` for all; Wave 5.

**Inherited verification evidence:** none — Sessions 1/2 never investigated
ECB or BIS.

**Known failure modes / concerns:** entirely unknown; these routes have not
been checked for reachability, robots/anti-bot posture, or feed stability
from any infrastructure, production or otherwise.

**Overlap/duplication notes:** ECB's own RSS/decision/press-conference/
exchange-rate routes are one `ECB` concentration unit; BIS's speech feed is
a restatement of *other* central banks' speeches, not BIS's own primary
output — its authority tier is recorded as proposed Tier 2 for that reason,
distinct from BIS's own statistical releases (proposed Tier 1, BIS is the
system-of-record for its own aggregated statistics).

**Remaining blockers:** full route verification (nothing tested), terms
review, and owner confirmation that V1 scope truly stays limited to ECB/BIS
per the approved default scope.

### 4.10 Professional Context

**Purpose:** Tier-3 attributed context and discovery only. Per owner rule,
neither Bloomberg Television nor Yahoo Finance may replace an available
official source.

| Candidate | Route(s) | Priority | Fn/Role | Tier | Wave | Cadence | Route Kind | Confidence | Terms |
|---|---|---|---|---|---|---|---|---|---|
| Bloomberg Television (YouTube) | `youtube.com/@markets` | USEFUL | candidate_function = CONTEXT + DISCOVERY; possible future registry role `CORROBORATING` | 3 | 4 | EVENT_DRIVEN | FILE_DROP (metadata/captions/selective transcript only — never continuous stream) | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| Yahoo Finance homepage | `finance.yahoo.com/` | OPTIONAL | candidate_function = CONTEXT + DISCOVERY; possible future registry role `CORROBORATING` | 3 | 4 | DAILY | INDEX_PAGE | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| Yahoo Finance news RSS | `finance.yahoo.com/news/rss` | USEFUL | candidate_function = CONTEXT + DISCOVERY; possible future registry role `CORROBORATING` | 3 | 4 | EVENT_DRIVEN | NATIVE_RSS | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |
| Yahoo Finance markets page | `finance.yahoo.com/markets/` | OPTIONAL | candidate_function = CONTEXT + DISCOVERY | 3 | 4 | DAILY | INDEX_PAGE | `CANDIDATE_UNTESTED` | NOT_YET_REVIEWED |

**Inherited verification evidence:** none — neither Bloomberg nor Yahoo was
investigated in any prior session.

**Known failure modes / concerns:** Bloomberg — continuous live-stream
transcription is explicitly excluded by owner instruction; only
metadata/captions/selective-segment transcript retrieval is in scope, and
only once a segment-selection mechanism exists (not designed here). Yahoo —
feed stability and automated-use terms are both unverified; Yahoo Finance is
explicitly not to be treated as the canonical market-data provider even if
convenient.

**Overlap/duplication notes:** per Session 4's eligibility gate, both
sources may produce `REPORTED_STATEMENT`-type evidence and may count toward
coverage/corroboration once activated, but neither may ever be sole support
for a claim, and the underlying official release, filing, or dataset should
be sought first wherever one exists — these two are commentary/aggregation
layers on top of Tier-1/2 facts, not independent originating issuers.

**Remaining blockers:** route/reachability verification (nothing tested),
Bloomberg segment-selection design (out of scope for this document), Yahoo
automated-use terms review, and — per owner instruction — confirmation that
neither source is ever substituted for an available official route.

### 4.11 Discovery and Contradiction

**Purpose:** discovery and contradiction-search leads only — never itself
evidence.

| Candidate | Route(s) | Priority | Fn/Role | Tier | Wave | Cadence | Route Kind | Confidence | Terms |
|---|---|---|---|---|---|---|---|---|---|
| Exa web search | via `mcporter call exa.web_search_exa` (hosted MCP endpoint `mcp.exa.ai/mcp`) | USEFUL | candidate_function = DISCOVERY; possible future registry role `CONTRADICTION_SEEKING` | 4 | 4 | INTRADAY (query-driven, not a scheduled release) | (CLI-mediated, not one of the six route kinds — a fixed-template external command per Session 1 §A.9, not a direct HTTP route) | `CANDIDATE_UNTESTED` — Session 1 confirms the adapter *pattern* exists (`ExaSearchChannel`/`mcporter`) but Session 2 never issued a live query and returned results; no live Exa search has actually been performed by any session | NOT_YET_REVIEWED |

**Inherited verification evidence:** Session 1 confirmed, by source
inspection only, that Agent Reach's `ExaSearchChannel` never itself searches
— it only checks whether `mcporter` is configured. Session 1 documented the
free, no-API-key, no-registration nature of the Exa/mcporter bridge per
`guides/setup-exa.md`, and recommended a scanner-owned adapter using a fixed,
argument-array `mcporter call exa.web_search_exa query="..."` invocation.
**No session has ever actually issued this call and inspected real results**
— this remains explicitly untested, not merely unverified-from-production.

**Known failure modes / concerns:** `check()` in Agent Reach deliberately
caps its own health signal at `"warn"`, never `"ok"`, because it never
starts the remote service to verify connectivity — so even a healthy-looking
`doctor` output would not confirm Exa actually works end-to-end.

**Overlap/duplication notes:** none — Exa is the only source of its kind in
this document.

**Remaining blockers:** an actual end-to-end test call has never been made;
this is a larger gap than most other untested candidates (which at least
have a known, inspectable route) since Exa's behavior is entirely mediated
through a CLI bridge whose output shape has not been observed.

### 4.12 X and Social Sources

**Purpose:** recorded for completeness only. Excluded from active V1 per
owner instruction.

| Candidate | Route(s) | Priority | Fn/Role | Tier | Wave | Cadence | Route Kind | Confidence | Terms |
|---|---|---|---|---|---|---|---|---|---|
| X (future owner-curated list) | none defined | LATER | candidate_function = DISCOVERY; possible future registry role `CONTRADICTION_SEEKING` or `CORROBORATING` | 4 | 5 | (not applicable — excluded from V1) | (not applicable) | `CANDIDATE_UNTESTED` — no attempt made, none authorized | NOT_YET_REVIEWED |

**Future account categories (owner-named, not yet curated):** official
agencies, official central banks, official companies, named professional
reporters, selected researchers.

**Explicit constraints, restated from owner instruction:** no authentication,
no browser-cookie use, no broad automated keyword scraping, no
social-channel diagnostics, no post may establish a fact by itself. Future
consideration requires an owner-curated list and a separate terms and route
review — this document performs neither.

**Inherited verification evidence:** none, and none was sought — Session 4/5
already treat any X-type material as Tier-4/open-web by default (never sole
support, never counted as coverage, may only seed contradiction search)
unless a future ratification says otherwise.

**Remaining blockers:** everything — this category has zero technical
investment beyond the categorical placeholder above, by design.

---

## 5. Market and Cross-Asset Observations

Per owner instruction, this section defines *desired observations only*. It
does not select a provider and does not accept any ticker/symbol mapping.
Every row below carries `candidate_function = MARKET_OBSERVATION`,
`terms_review_status = NOT_YET_REVIEWED`, and
`verification_confidence = CANDIDATE_UNTESTED` (or, for the instrument
concept itself, "not applicable pending provider selection") — no provider
exists yet against which to verify anything.

### 5.1 V1 core candidates

NQ futures; ES futures; RTY futures; VIX or a verified volatility proxy;
2-year, 5-year, 10-year, and 30-year Treasury yields; 2s10s curve; 5s30s
curve; Treasury futures (especially ZN and ZB, if reliable); HYG; LQD; broad
US dollar or DXY proxy; SOFR; EFFR; gold; crude oil; semiconductor index or
ETF; Nasdaq-100 market breadth; top Nasdaq-100 constituent premarket
contribution.

### 5.2 Later or optional candidates

MOVE or an approved rates-volatility measure; USD/JPY; USD/CNH; copper;
bitcoin; international equity indices; options-implied expected move; dealer
gamma or positioning (only with defensible provenance).

### 5.3 Required provider-evaluation criteria (for a later session)

Before any market-data provider is selected, that later session must
evaluate it for: delayed versus real-time data, timestamp integrity, symbol
stability, futures rollover handling, market-session coverage, automation
and personal-use terms, rate limits, revision behavior, data gaps,
reliability, and cost. **This document makes no recommendation on any of
these axes.**

**Overlap note:** SOFR and EFFR appear both here (as market observations)
and in §4.3 (as NY Fed reference-rate publications). These are related but
not necessarily identical representations — a market-data provider's SOFR
quote and the NY Fed's own published SOFR fixing may differ in timing or
source mechanism. Do not treat them as interchangeable without a later
session confirming they resolve to the same value at the same timestamp.

---

## 6. Candidate Rollout Waves

Rollout wave is a document-local sequencing axis, independent of authority
tier and lifecycle state. A Wave 1 assignment does not imply `ACTIVE`
lifecycle status, and a later wave does not imply lower authority.

- **Wave 1:** Federal Reserve Board (feeds, calendar), BLS (structured API,
  4 confirmed series), BEA (RSS, schedule), Treasury (Fiscal Data API,
  TreasuryDirect API), official calendars generally.
- **Wave 2:** New York Federal Reserve (provisional — owner may prefer
  Wave 1), Census, SEC EDGAR, dynamic Nasdaq-100 corporate coverage, an
  approved market-data source (once selected).
- **Wave 3:** ECB, BIS, Federal Reserve video/transcript retrieval,
  Bloomberg Television selective context.
- **Wave 4:** Yahoo Finance, Exa discovery and contradiction, additional
  corporate IR sources.
- **Wave 5:** advanced positioning (CFTC — flagged in §4.8 as a candidate
  for earlier promotion given its low technical complexity), options
  structure, rates volatility, international markets beyond ECB/BIS, future
  owner-approved social sources.

---

## 7. Explicit Exclusions and Deferrals

- Reuters as a fixed V1 source — Session 2 verified all retrieval routes
  fail (direct infra-blocked, Jina 403) and corroborated that Reuters
  discontinued its official RSS in June 2020; no viable route exists today.
- Broad X monitoring — excluded from active V1 (§4.12).
- Continuous Bloomberg live-stream transcription — excluded (§4.10).
- Anonymous social accounts — excluded categorically.
- Unverified aggregators — excluded categorically; the CME economic-release
  calendar (§4.4) and any future earnings-calendar aggregator (§4.7) are
  explicitly named as needing independent verification before any reliance,
  and are never treated as an issuer's own authority.
- Sources with no provenance — excluded categorically.
- Paywalled automation without approved access — excluded categorically.
- Sources whose terms prohibit use — excluded categorically (this is the
  entire purpose of the terms-review gate in §3.3).
- Unverifiable dealer-gamma sources — excluded from §5.2 unless a future
  session establishes defensible provenance.
- Monitoring every Nasdaq-100 company equally at full depth — excluded by
  design; scope is limited to the top 10 by weight, same-day earnings
  reporters, and material-event constituents (§4.7).
- Unsupported consensus figures — excluded; consensus-data provider
  selection remains an unresolved owner decision (N.1) and fabrication of
  consensus is forbidden per Session 4.
- Any AI-generated statement treated as an original source — excluded by
  architecture (§2, principle 6; Session 4 §A.4/§E.1).

---

## 8. Owner Preferences and Unresolved Decisions

### 8.1 Pre-existing unresolved items this candidate tree touches

- **N.1 (Session 1):** consensus-data provider — unresolved; blocks any
  calendar feature that would display consensus figures.
- **N.13 (Session 1, reaffirmed Session 4 §C4):** per-source terms-of-service
  review — unresolved for every single candidate in this document, including
  Fed, BLS, BEA, and Treasury.
- **N.16–N.19 (Session 1, reaffirmed Session 4/5):** market-data provider and
  instrument/symbol scope — unresolved; §5 defines desired observations only.
- **Session 3 §U.6 concentration thresholds:** the maximum share of an
  evidence cluster's weight attributable to a single issuer family, and the
  minimum number of issuer families required to avoid a
  `LIMITED_SOURCE_DIVERSITY` flag — both left as `X%`/`N` placeholders,
  unratified.
- **Session 4/5 evidence-type and eligibility ratification:** the tier
  capability matrix (§3.6) and evidence-type taxonomy remain proposed, not
  ratified.

### 8.2 New items raised by this document

- **`DISCOVERY` role ratification (§3.4):** should Session 3's closed
  `source_roles.yaml` vocabulary gain a formal `DISCOVERY` role, or should
  discovery-function sources always resolve to `CONTRADICTION_SEEKING` /
  `CORROBORATING` once evaluated? Unresolved.
- **`CORE`/`USEFUL`/`OPTIONAL`/`LATER` priority scale ratification:** this
  document introduces the scale; it has no prior-session precedent and has
  not been formally adopted into the registry architecture. Unresolved.
- **Wave-sequencing ratification:** §6's five waves are a proposal; in
  particular, whether NY Fed belongs in Wave 1 (alongside Fed Board) or
  Wave 2, and whether CFTC positioning (§4.8) should move earlier than
  Wave 5 given its technical simplicity, are both open questions.
- **Per-company `issuer_family` keying (§4.7):** whether Nasdaq-100 IR
  sources should be grouped as one `CORPORATE_IR` concentration family or
  split per ticker — left open, flagged as a registry-design question for a
  later session.
- **Outstanding route verification:** NY Fed, Census, SEC EDGAR, CFTC, ECB,
  BIS, Bloomberg, Yahoo Finance, a live Exa query, and X remain entirely
  untested by any session. None of this document's `CANDIDATE_UNTESTED`
  labels should be read as low-risk simply because the route looks
  straightforward on paper — Session 2 found real, non-obvious blocks
  (Akamai on `bls.gov`, no feed at all for Treasury press, Reuters'
  discontinued RSS) on sources that also looked straightforward.
- **Production-host re-test:** every `VERIFIED_LIVE_SESSION_2_INFRA` entry
  in this document (Fed Board feeds/calendar, BLS API + schedule/narrative
  pages, BEA RSS/schedule, Treasury Fiscal Data/TreasuryDirect/press-via-Jina)
  was only ever tested from Claude Code's datacenter infrastructure. None of
  these results should be treated as final until re-tested from the owner's
  actual production host.

---

## 9. Classification and Verification Handoff

A later session should perform, in this rough order:

1. **Classification** — for each candidate above, confirm or revise the
   proposed priority, authority tier, function/role, and wave, with owner
   ratification of the new `CORE/USEFUL/OPTIONAL/LATER` scale and the
   `DISCOVERY`-role question (§8.2).
2. **Verification** — live-retrieval testing, from the owner's actual
   production host, of (a) re-testing every `VERIFIED_LIVE_SESSION_2_INFRA`
   candidate and (b) first-time testing of every `CANDIDATE_UNTESTED`
   candidate, following the same read-only-GET discipline Session 2 used.
3. **Terms review** — a genuine, source-specific N.13 determination for
   every candidate this document lists, resolving each `NOT_YET_REVIEWED`
   status to either `CLEARED` or `FAILED` with a recorded rationale/reference.
4. **Registry-ready entry preparation** — once a candidate has passed
   classification, verification, and terms review, prepare the concrete
   field values (per Session 3's logical field inventory: `source_id`,
   `issuer_family`, `authority_tier`, `authority_rationale`, `roles`,
   `coverage_domains`, `cadence_class`, `schedule_ref`, `routes`,
   `dedup_strategy`, `terms_review_status`, `terms_review_ref`, etc.) that a
   future implementation session could use to populate the registry.

**This handoff explicitly does not instruct the next session to create or
populate `config/source_registry.yaml`.** The real registry remains an
implementation artifact to be created only after source verification, terms
review, owner approval, and concrete registry-format design (Session 3's own
schema work is a logical field inventory, not yet a declared schema) are all
complete. Owner decisions surfaced in §8 should be resolved, or explicitly
carried forward, before that implementation work begins.
