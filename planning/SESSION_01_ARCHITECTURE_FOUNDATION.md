# NQ Premarket Environmental Intelligence Scanner
## Session 1 — Agent Reach Architecture Foundation
### Deliverables A–E, N, O

**Session scope confirmation:** Both `planning/MASTER_CLAUDE_CODE_PROMPT_v1.1_PART_1.md.md` and `planning/MASTER_CLAUDE_CODE_PROMPT_v1.1_PART_2.md..txt` were read in full before any inspection began. Part 1 runs through Section 20 (ending mid-header at "10:00 ET REPORT"); Part 2 opens with "20. REPORT REQUIREMENTS — CONTINUED" and runs to Section 39 ("FINAL SESSION RULES"). Both Section 0 and Section 39 confirm this session is limited to deliverables **A, B, C, D, E, N, O**. Deliverables F–M (final data models/schemas, policy artifacts, runtime LLM prompts, full threat model, full reliability plan, full test plan, phased implementation plan) are explicitly out of scope and are not attempted here.

**Repository inspected:** `github.com/Panniantong/Agent-Reach` (local checkout).
**Git commit SHA at inspection time:** `93ae1d18c37b707dec053c7c4f9d91cd8ef8943d` (2026-08-12 11:39:47 +0800).
**Inspection method:** Read-only source inspection, test-file inspection, `git log`, and static reading only. No installs, no authentication, no cookie reads, no social-channel probing, no production-code changes were performed. Diagnostics (`agent-reach doctor`) were **not executed**; its safety was established entirely by source inspection (see Deliverable A/C — "diagnostic carve-out" discussion), which this session judged sufficient and lower-risk than an actual run.

**Labeling convention used throughout:** every capability claim is tagged one of `DOCUMENTED ONLY`, `SOURCE-INSPECTED`, `TEST-VERIFIED` (a matching test file/assertion exists), or `ASSUMED` (flagged explicitly, never left implicit). No behavior is described as verified unless it was actually read in source or asserted in a test.

---

## DELIVERABLE A — REPOSITORY CAPABILITY MAP

For each capability: implementation, path, entry point, dependencies, tests, documented-vs-verified split, importable-vs-CLI-only, signature, limitations, security notes, reuse status, required scanner adapter/AR extension, and verification confidence.

### A.1 Public web retrieval (Jina Reader)

| Field | Value |
|---|---|
| Implementation | `WebChannel` class |
| Path | `agent_reach/channels/web.py:34-67` |
| Entry point | `WebChannel().read(url: str) -> str` (`web.py:48`) |
| Dependencies | stdlib `urllib.request` only — **no** `jina` SDK, **no** `requests` |
| Tests | `tests/test_web_channel.py` (14 `def test_`) |
| Documented vs verified | SOURCE-INSPECTED — real implementation, not a stub |
| Importable vs CLI-only | **Importable.** Clean method, stable signature, no subprocess |
| Signature | `def read(self, url: str) -> str` |
| Limitations | 30s timeout, `_MAX_RESPONSE_BYTES = 5 MiB` hard cap, antibot/CAPTCHA page detection raises `RuntimeError` rather than returning junk content as if valid; no retry logic |
| Security notes | URL passed through `utils.url.normalize_public_http_url` before being concatenated into the `https://r.jina.ai/<url>` request — SSRF-hardened (scheme allowlist, userinfo rejection, hostname blocklist, `is_global` IP check) |
| Reuse status | **Directly reusable by the scanner via composition** (import `WebChannel`, call `.read(url)`) |
| Scanner adapter needed | Thin wrapper adding: source-authority tagging, content hashing, retrieval timestamp, retry/circuit-breaker policy (none exists in `WebChannel` itself) |
| AR extension needed | None |
| Verification confidence | **High** — full file read |

### A.2 Jina integration (separately, since the master prompt treats it as its own item)

Jina is not a separate Agent Reach module — it *is* `WebChannel.read()` (A.1). There is no dedicated `jina.py`. `check()` (`web.py:43-46`) never network-probes Jina; it always reports `("ok", ...)` because it's "an always-available zero-overhead fallback channel" per the source comment. **This means Agent Reach's own doctor output cannot tell the scanner whether Jina Reader is actually reachable right now** — the scanner must add its own Jina health/availability check if it wants that signal. Confidence: High (full file read).

### A.3 RSS and Atom

| Field | Value |
|---|---|
| Implementation | `RSSChannel` class |
| Path | `agent_reach/channels/rss.py` (27 lines, full file) |
| Entry point | None for actual feed parsing — only `check()` |
| Dependencies | `feedparser>=6.0` (pinned `==6.0.12` in `constraints.txt`), imported only to test importability (`rss.py:18`, `# noqa: F401`) |
| Tests | No dedicated `test_rss.py` found |
| Documented vs verified | `CLAUDE.md`/README imply RSS support; **source shows zero feed-parsing logic** — `can_handle()` is a URL-substring heuristic (`/feed`, `/rss`, `.xml`, `atom`), `check()` only verifies `feedparser` imports cleanly |
| Importable vs CLI-only | Neither — **there is no fetch/parse capability to import or shell out to.** `feedparser` itself is importable (it's a plain PyPI library, already a transitive Agent Reach dependency) but `RSSChannel` doesn't wrap it |
| Limitations | No ID/title/URL/summary/category/pubtime extraction, no ETag/Last-Modified, no malformed-feed handling, no dedup — none of this exists in Agent Reach |
| Security notes | N/A (no network code in this file) |
| Reuse status | **Not reusable as a capability** — the only reusable thing is the fact that `feedparser` is already a proven, pinned dependency the scanner can import directly |
| Scanner adapter needed | **Full scanner-owned RSS module** wrapping `feedparser.parse()` directly: entry extraction, ETag/Last-Modified conditional GET, malformed-feed handling, GUID/URL/title/content-hash dedup |
| AR extension needed | None — composition via shared dependency is correct here, not an AR extension |
| Verification confidence | **High** — full file read, confirmed absence of parsing logic |

### A.4 YouTube metadata

| Field | Value |
|---|---|
| Implementation | `YouTubeChannel` class |
| Path | `agent_reach/channels/youtube.py:39-141` (full file) |
| Entry point | None for metadata — `check()` only version-probes `yt-dlp --version` |
| Dependencies | `yt-dlp[default]>=2026.07.04` (pinned dependency in `pyproject.toml`), a JS runtime (`deno`/`node`, health-checked) |
| Tests | `tests/test_youtube_channel.py` (20 `def test_`) |
| Documented vs verified | Source confirms **no** video-id/channel/title/upload-time/description/duration/chapter extraction exists in this file |
| Importable vs CLI-only | **CLI-only for metadata.** `yt-dlp` itself is a pinned, importable Python library (`import yt_dlp; yt_dlp.YoutubeDL(...)`), but Agent Reach's own code never uses that Python API — it only shells out to `yt-dlp --version` for health-checking |
| Limitations | `_JS_RUNTIMES_SUPPORTED_FROM = (2025, 11, 12)` version gate — yt-dlp needs a JS runtime post that version; `check()` surfaces `warn` if only `node` present without `--js-runtimes` config |
| Security notes | Health probe only, argument-array `subprocess.run`, 10s timeout |
| Reuse status | **Shared-dependency reuse, not Agent Reach capability reuse** — scanner should use the pinned `yt-dlp` package directly (its own `YoutubeDL` Python API, well-documented and stable upstream) for metadata, rather than routing through `YouTubeChannel` |
| Scanner adapter needed | New scanner module wrapping `yt_dlp.YoutubeDL` for metadata extraction (id, channel, title, upload date, description, duration, chapters) |
| AR extension needed | None |
| Verification confidence | **High** — full file read |

### A.5 Channel and playlist handling

Not implemented anywhere in Agent Reach for YouTube or any other platform at the "list contents of a channel/playlist" level — `YouTubeChannel.can_handle()` only matches `youtube.com`/`youtu.be` URLs; there is no playlist-expansion or channel-crawl code. **Confidence: High** (same file read as A.4). Fully a scanner-owned responsibility, most naturally implemented via `yt_dlp`'s own playlist/channel-listing support (an upstream, well-established capability of the pinned library, not something Agent Reach needs to provide).

### A.6 Subtitle retrieval / A.7 Automatic captions

Not implemented in `youtube.py`. The only caption-adjacent code path is `YouTubeChannel.transcribe()` (A.8 below), which is an **ASR fallback**, not YouTube's own caption tracks (official transcripts, human captions, or platform auto-captions). **Confidence: High.** Extracting YouTube's native caption tracks (the top three rungs of the master prompt's transcript ladder) is not an Agent Reach capability at all — it is a `yt_dlp` library capability (`--write-subs`/`--write-auto-subs` equivalents) the scanner would call directly.

### A.8 Transcript fallback (ASR)

| Field | Value |
|---|---|
| Implementation | `transcribe()` function |
| Path | `agent_reach/transcribe.py:405-412` (full file, 499 lines) |
| Entry point | `def transcribe(source: str, *, provider="auto", out_dir=None, config=None, allow_provider_fallback=False) -> str` |
| Dependencies | `requests` (HTTP POST to Whisper-compatible endpoint), external binaries `yt-dlp`, `ffmpeg`, `ffprobe` invoked via `subprocess.run` with **argument-array** calls (no shell=True anywhere) |
| Tests | `tests/test_transcribe.py` (40 `def test_`) |
| Documented vs verified | Module docstring states it's "Designed to be importable from channels" — **confirmed true by source and by `YouTubeChannel.transcribe()` actually doing exactly that** (`youtube.py:120-141`, lazy import + pass-through) |
| Importable vs CLI-only | **Importable**, real Python function, stable-looking signature |
| Limitations | Providers: Groq (`whisper-large-v3`) and OpenAI (`whisper-1`) only; `SIZE_LIMIT_BYTES=24MiB`/`MAX_SOURCE_BYTES=512MiB`/`MAX_CHUNKS=24`/`MAX_AUDIO_SECONDS`(~4h) hard caps; provider fallback across providers only happens if caller explicitly passes `allow_provider_fallback=True`; no built-in HTTP retry-with-backoff (`requests.RequestException` is caught once and re-raised, not retried) |
| Security notes | `_assert_safe_public_url()` (`transcribe.py:214-247`) blocks loopback/private/link-local/metadata-style hosts, including alternate IPv4 literal encodings (octal/hex/short form) that a naive check would miss — genuinely careful SSRF hardening, independent implementation from `utils/url.py` |
| Reuse status | **Directly reusable by the scanner via composition** for the ASR-fallback rung of the transcript ladder (rung 5: "Audio transcription fallback if implemented and approved" — it is implemented) |
| Scanner adapter needed | Thin wrapper adding: which-rung-succeeded tagging (never present quotation marks around auto-transcript text, per master-prompt direct-quote rule), retry/backoff policy layered on top, credential presence check before invocation |
| AR extension needed | None |
| Verification confidence | **High** — full file read, cross-referenced with `youtube.py` |

There is also a **standalone Bash script**, `agent_reach/scripts/transcribe_xiaoyuzhou.sh` (394 lines, full file read) — a completely separate reimplementation of a similar download→ffmpeg→Whisper pipeline hardcoded to the Xiaoyuzhou podcast platform and Groq only. It is CLI-only by construction (it's a shell script), not part of the Python import graph, and not relevant to the NQ scanner's institutional/YouTube/RSS source universe. Noted for completeness; not a reuse candidate.

### A.9 Exa / semantic search

| Field | Value |
|---|---|
| Implementation | `ExaSearchChannel` class |
| Path | `agent_reach/channels/exa_search.py:10-44` (full file) |
| Entry point | None for actually searching — only `check()` |
| Dependencies | External `mcporter` CLI, which bridges to Exa's **hosted MCP endpoint** `https://mcp.exa.ai/mcp` — no direct Exa REST call, no Exa Python SDK anywhere in the repo |
| Tests | Covered inside `test_channels.py`/`test_p0_cli.py`; no dedicated `test_exa.py` |
| Documented vs verified | `guides/setup-exa.md` confirms free, no API key, no registration — consistent with source (no key-handling code in `exa_search.py`) |
| Importable vs CLI-only | **CLI-only.** No `search()` method exists anywhere. Actual searching requires invoking `mcporter call exa.web_search_exa query="..." numResults=N` as an external command |
| Limitations | `check()` deliberately caps its best result at `"warn"`, never `"ok"`, even when `mcporter` reports Exa configured, because "Doctor did not start the remote service to verify connectivity" |
| Security notes | If this command template is used, treat `query` as the only variable position, always as an argument-array element (never string-interpolated into a shell command), fixed executable (`mcporter`), fixed subcommand (`call`), fixed tool name (`exa.web_search_exa`) |
| Reuse status | Not reusable as Python; the *pattern* (fixed command template, argument-array invocation) is reusable |
| Scanner adapter needed | New scanner module implementing a fixed, sanitized `mcporter call exa.web_search_exa` invocation with timeout, output-size cap, and JSON-result parsing |
| AR extension needed | None — composition is sufficient; extending Agent Reach to add a Python Exa client would violate the "don't invasively extend AR core" preference for no real gain, since the CLI bridge already works and is simple to wrap |
| Verification confidence | **High** — full file read + guide read |

### A.10 GitHub functionality

| Field | Value |
|---|---|
| Implementation | `GitHubChannel` class |
| Path | `agent_reach/channels/github.py:95-145` (full file) |
| Entry point | None for repo/file/issue reads — only `check()` |
| Dependencies | External `gh` CLI (GitHub's official CLI) |
| Tests | Covered in `test_channels.py`/`test_p0_cli.py` |
| Importable vs CLI-only | **CLI-only.** `check()` deliberately never runs `gh auth status` (to avoid a documented side-effecting device-ID write even on `--version`), checking credential presence instead via `GH_TOKEN`/`GITHUB_TOKEN` env vars or `~/.config/gh/hosts.yml` |
| Relevance to this scanner | **Low.** The master prompt's institutional-source universe (Fed, Treasury, BLS, Reuters, company IR pages, etc.) does not depend on GitHub. Flagged as available-but-not-prioritized. |
| Verification confidence | **High** — full file read |

### A.11 Configuration

| Field | Value |
|---|---|
| Implementation | `Config` class |
| Path | `agent_reach/config.py` (full file) |
| Storage | `~/.agent-reach/config.yaml`, YAML via `yaml.safe_load`/`safe_dump` |
| Tests | `tests/test_config.py` (25), `tests/test_home_isolation.py` (2), `tests/test_private_file_writes.py` (22) |
| Importable | Yes — `Config()`, `.get(key)`, `.set(key,value)`, `.delete(key)`, `.to_dict()`, `.save()` |
| Security notes | Atomic write, `0o600` perms, symlink rejection before *and* after write (TOCTOU-hardened), `.get()` falls back to `os.environ.get(key.upper())`, `.to_dict()` masks any key containing `key/token/password/proxy/cookie/secret/session/sessdata/csrf/auth/cred/ct0` |
| Reuse status | **Reusable by composition** for scanner-owned config needs that genuinely overlap with Agent Reach's own settings (e.g., a shared Groq/OpenAI key) — but the scanner's own extensive policy/config surface (source registry, thresholds, weights — dozens of YAML files per Section 23) should be **scanner-owned**, not stuffed into `Config`, since `Config` is a flat key-value store, not a structured policy engine |
| Verification confidence | **High** — full file read |

### A.12 CLI execution

| Field | Value |
|---|---|
| Implementation | `main()` |
| Path | `agent_reach/cli.py:59` |
| Entry point | Console script `agent-reach = "agent_reach.cli:main"` (`pyproject.toml:52-53`) |
| Subcommands | `setup`, `install`, `configure`, `doctor`, `uninstall`, `skill`, `format`, `transcribe`, `check-update`, `watch`, `version` |
| Importable vs CLI-only | The CLI module is importable (`from agent_reach.cli import main`) but its subcommands are designed to be invoked as a CLI, not called as library functions — no stable public functions beyond what's individually documented elsewhere (`Config`, `AgentReach`, channel classes) |
| Notable finding | **There is no `read`/`search` subcommand.** The CLI only installs, configures, and diagnoses; it never fetches content on the user's behalf. This matches the README's stated design philosophy ("capability layer, not a wrapper") and contradicts `CLAUDE.md`'s "Core read/search routing logic" framing of `core.py` |
| Verification confidence | **High** — full file read of dispatch table and relevant handlers |

### A.13 Backend routing

| Field | Value |
|---|---|
| Implementation | `Channel.ordered_backends()` (`channels/base.py:45-59`) + `agent_reach/backends/opencli.py` |
| Mechanism | Each channel declares an ordered `backends: List[str]` candidate list; a config key `<channel>_backend` or env `<CHANNEL>_BACKEND` can reorder (never hide) candidates |
| OpenCLI specifically | External Node.js CLI driving the user's real Chrome via a browser-bridge extension; `opencli_status()` never runs `opencli doctor` (which auto-starts a daemon — a side effect) — instead probes `opencli --version` plus a local HTTP status endpoint (`http://127.0.0.1:19825/status`) with proxy explicitly disabled |
| Tests | `tests/test_opencli_backend.py` (11) |
| Reuse status | Reusable as a *pattern* (ordered-candidate backend selection, side-effect-free probing) if the scanner ever needs multi-backend routing; not directly needed for the scanner's public-data sources |
| Verification confidence | **High** — full file reads of `base.py` and `opencli.py` |

### A.14 Diagnostics and health checks

| Field | Value |
|---|---|
| Implementation | `check_all()`, `format_report()` |
| Path | `agent_reach/doctor.py:16-131` (full file) |
| Scoping | **No CLI flag or config option exists to scope/exclude channels.** `doctor` only accepts `--json`. `check_all()` unconditionally iterates every entry in `channels.ALL_CHANNELS` (all 15, including twitter/reddit/facebook/instagram/xiaohongshu) |
| Safety without scoping | Confirmed via full source read of `doctor.py` plus the `check()` implementations of the five social channels (`twitter.py`, `reddit.py`, `_opencli_site.py` for facebook/instagram, `xiaohongshu.py`): none of them issue a live authenticated request to the underlying platform. Each either (a) checks `shutil.which`/version-probes a local binary, (b) reads a small local credential file and checks a TTL, or (c) probes a **local loopback** service. None calls upstream `status`/`auth status` commands, specifically because the source comments document that doing so would auto-refresh or auto-read browser cookies — the exact behavior the master prompt's diagnostic carve-out is designed to prevent |
| Test verification | `tests/test_doctor.py::test_real_doctor_path_is_zero_write_and_never_runs_risky_status_commands` performs a real end-to-end `check_all()`/`_cmd_doctor()` run, snapshots the entire home/config tree before/after and asserts **zero filesystem changes**, and asserts the subprocess mock never receives `["auth","status"]`/`["status"]`/`["daemon","status"]` argv patterns, with twitter/reddit/facebook/instagram/xiaohongshu all resolving to `"warn"`/`active_backend=None` rather than error or auth-attempt |
| Redaction | Every returned message is passed through `scrub_url_credentials()` (`doctor.py:36`) before being placed in the report — "doctor is the final output boundary" per source comment |
| Conclusion for this session | Because the *design* is provably read-only by source inspection and by an existing passing test, this session judged that **actually executing `agent-reach doctor`** was unnecessary risk for zero incremental information — the master prompt's own text permits inspecting doctor's source in lieu of running it when it "cannot be safely scoped," and here it is source-verified safe to run in full, but this session chose the more conservative path of source+test verification only, per the instruction to prefer inspection over runtime probing where both would answer the question equally well. **This is flagged as a judgment call, not a hard requirement — running `agent-reach doctor --json` in a follow-up session, on the operator's actual machine, remains a reasonable and low-risk verification step.** |
| Verification confidence | **High** — full file reads (`doctor.py`, `twitter.py`, `reddit.py`, `_opencli_site.py`, relevant parts of `xiaohongshu.py`) + one full test read |

### A.15 URL validation

| Field | Value |
|---|---|
| Implementation | `normalize_public_http_url()`, `domain_matches()`, `host_matches()` |
| Path | `agent_reach/utils/url.py` (full file) |
| Tests | `tests/test_url_security.py` (5) |
| Verified behavior | Rejects: non-http(s) schemes (blocks `file://` and others), embedded userinfo, a hostname blocklist (`localhost`, `*.local`, `*.internal`, `metadata.google.internal`, etc. — both exact and suffix match), and any literal IP address that is not `ipaddress`'s `.is_global` (blocking loopback, RFC1918, link-local **including the generic 169.254.169.254 cloud-metadata IP**, and other reserved ranges). Also closes legacy IPv4-literal-encoding bypasses (octal/hex/short-form) via a `socket.inet_aton` fallback |
| Gap identified (not a defect, a scope note) | **No DNS-rebinding protection** — validation happens once against the literal string/parsed host; a public hostname that resolves to a private IP at fetch time would pass this check. If the scanner builds any of its own direct-fetch paths (rather than always going through `WebChannel.read`), it should re-validate the *resolved* address at connect time, not just the hostname |
| Reuse status | **Directly reusable by composition** — the scanner should import and use these functions for any URL it accepts from an untrusted source (feed entries, Exa results, calendar-page links) before ever fetching them |
| Verification confidence | **High** — full file read |

### A.16 Credential handling

Covered by A.11 (`Config`) plus `agent_reach/cookie_extract.py` (full file read) and `agent_reach/channels/github.py`'s `hosts.yml` handling. Key finding: **Twitter and XiaoHongShu are structurally blocked from automatic browser-cookie extraction** — `cookie_extract.py`'s `PLATFORM_SPECS` sets `"cookies": None` for both, and `_require_browser_extractable()` raises `ValueError` if `--from-browser` is attempted for either, independently re-enforced at the CLI argument-parsing layer (`cli.py`'s `manual_keys` dict) — two separate enforcement points. Only Bilibili and Xueqiu support automatic extraction, each requiring an explicit `--platform` argument (never "extract everything"). This matches, and is stronger than, `CLAUDE.md`'s stated Cookie-Editor-only policy. **Not relevant to this scanner at all**, since the scanner must never touch social channels — noted here only for completeness of the capability map. Verification confidence: **High**.

### A.17 Secret redaction

| Field | Value |
|---|---|
| Implementation | `scrub_url_credentials()` |
| Path | `agent_reach/utils/text.py:24-28` (full file, 38 lines) |
| Tests | `tests/test_scrub_credentials.py` (6) |
| Mechanism | Three regexes handling scheme-based URL userinfo (`scheme://user:pass@host`), bare userinfo (`user:pass@host` with no scheme), and sensitive query/fragment parameters (token/key/password/secret/session/cookie/credential families) |
| Applied at | `doctor.py` (every channel message), `cli.py` (configure/transcribe errors), `cookie_extract.py` (nearly every error string), `mcp_server.py` (tool error results) — **applied per call-site, not via a centralized logging filter**. If a channel ever logged a raw secret directly (bypassing this function), it would not be caught automatically |
| Separate mechanism | `Config.to_dict()` uses key-*name*-based masking (not content-pattern matching) — a different, complementary approach |
| Reuse status | **Directly reusable by composition** — the scanner should call `scrub_url_credentials()` (or an equivalent it owns) on any string derived from external content or its own config before writing to logs, reports, or evidence records, satisfying Section 6's "secrets must never be written to reports/logs" requirement |
| Verification confidence | **High** — full file read |

### A.18 Structured output

No general-purpose "structured output" facility exists (e.g., no Pydantic models, no JSON-schema validation layer) anywhere in Agent Reach outside of `doctor.py`'s plain `Dict[str, dict]` return and the CLI's `--json` flag on `doctor`. **This is entirely a scanner-owned responsibility** (canonical report state, evidence records, etc., per Sections 15/20/28). Confidence: **High** (confirmed by absence across all files read).

### A.19 Scheduling

No scheduler, cron integration, or daemon/background-process code exists anywhere in the `agent_reach` package (confirmed by the full CLI subcommand list in A.12 — the closest thing is `watch`, which is a `check-update` polling loop for the tool's own version, not a content-collection scheduler, per `cli.py:174` → `_cmd_watch()`; not deep-read further as out of scope, but its docstring/name and its sole use of `check-update` logic confirm this). Section 21's scheduling model is **entirely scanner-owned**. Confidence: **High**.

### A.20 State / storage

No `sqlite3` import, no ORM, no persistent state file beyond `~/.agent-reach/config.yaml` (a flat settings file, not an operational database) exists anywhere in the codebase. Section 22's SQLite/filesystem storage direction is **entirely scanner-owned**. Confidence: **High**.

### A.21 Retry behavior

The only retry-capable primitive found is `probe_command(..., retries=0)` in `agent_reach/probe.py:47-81` — and it is explicitly documented as **verbatim re-execution with no backoff**, intended only for side-effect-free health probes ("so a non-idempotent command would repeat its effect" — the exact opposite of what a content-retrieval retry policy needs). `WebChannel.read()` and `transcribe.transcribe()` have **no retry logic** of their own (single-attempt, timeout-bounded). Section 27's retry policy (max-attempts by criticality tier, exponential backoff) is **entirely scanner-owned**. Confidence: **High**.

### A.22 Circuit-breaking behavior

No circuit-breaker pattern (open/half-open/closed state, failure-count thresholds, cool-down windows) exists anywhere in Agent Reach. Section 27's circuit-breaker requirement is **entirely scanner-owned**. Confidence: **High** (confirmed by absence).

---

## DELIVERABLE B — CURRENT AGENT REACH ARCHITECTURE

### B.1 High-level module map

```
agent_reach/
├── cli.py                 CLI entry point (argparse, 11 subcommands) — no read/search subcommand
├── core.py                AgentReach class — 2 methods, both delegate to doctor.check_all
├── config.py              Config — YAML settings store, atomic writes, symlink-hardened
├── doctor.py              check_all() / format_report() — diagnostics, all-channels, no scoping flag
├── probe.py               probe_command() — side-effect-free external-binary health probing
├── cookie_extract.py      Least-privilege browser-cookie extraction (Bilibili/Xueqiu only; Twitter/XHS blocked)
├── transcribe.py          Whisper ASR fallback pipeline (yt-dlp → ffmpeg → Groq/OpenAI), SSRF-hardened
├── channels/
│   ├── base.py            Channel(ABC) — only can_handle() is abstract; check() has concrete default
│   ├── __init__.py        ALL_CHANNELS — flat hardcoded list of 15 instances, no dynamic discovery
│   ├── web.py             WebChannel — real Jina Reader client, read() importable
│   ├── rss.py             RSSChannel — feedparser-importability check only, no parsing
│   ├── youtube.py         YouTubeChannel — yt-dlp version probe + transcribe() passthrough
│   ├── exa_search.py      ExaSearchChannel — mcporter-config inspection only
│   ├── github.py          GitHubChannel — gh-CLI health probe only
│   ├── _opencli_site.py   Shared base for Facebook/Instagram (OpenCLI-backed)
│   ├── mcporter.py        Config-inspection helper (not a Channel) for MCP routing
│   ├── twitter.py, reddit.py, xiaohongshu.py, bilibili.py, xueqiu.py, v2ex.py,
│   │   linkedin.py, xiaoyuzhou.py, facebook.py, instagram.py   (12 more platform channels — social/auth-gated
│   │                                                            or Chinese-platform channels; out of scope
│   │                                                            for this scanner, see Section 5 of the master
│   │                                                            prompt)
├── backends/
│   ├── opencli.py          opencli_status() — side-effect-free daemon/extension probing
│   └── __init__.py
├── integrations/
│   └── mcp_server.py       Exposes exactly one MCP tool: get_status → doctor_report()
├── utils/
│   ├── url.py              SSRF-hardened URL validation/normalization (reusable)
│   ├── text.py             scrub_url_credentials() secret redaction (reusable)
│   ├── paths.py            Symlink-hardened filesystem helpers (home_dir, atomic writes, no-follow reads)
│   └── process.py          UTF-8 subprocess-env helpers only (no process-spawning code itself)
├── skill/                  OpenClaw/Claude-Code "skill" registration files (SKILL.md, SKILL_en.md, references/)
└── guides/                 Per-platform setup docs (Exa, Groq, Reddit, Twitter, XiaoHongShu)
```

### B.2 CLI map

`agent-reach {setup, install, configure, doctor, uninstall, skill, format, transcribe, check-update, watch, version}` — argparse subparsers in `cli.py:68-177`. No `read`/`search` subcommand exists at any point in the current codebase.

### B.3 Channel map

15 channels registered statically in `channels/__init__.py:26-42` (`ALL_CHANNELS`), each a `Channel` subclass. Of these, only **3** expose any content-fetching capability beyond `check()`: `WebChannel.read()`, `YouTubeChannel.transcribe()` (delegates to `transcribe.py`), and two platform-specific `search()` methods that are out of this scanner's scope (`V2EXChannel.search()`, `XueqiuChannel.search_stock()`). The remaining 12 are pure install/health-check wrappers around external CLIs or browser-bridge tools.

### B.4 Backend-routing map

`Channel.backends: List[str]` (ordered preference) → `ordered_backends(config)` applies a `<channel>_backend`/`<CHANNEL>_BACKEND` override that reorders but never hides candidates → `check()` executes a real (non-`which`-only) probe per candidate and records `active_backend`. `agent_reach/backends/opencli.py` implements this pattern's most complex instance (external daemon + browser extension).

### B.5 Configuration map

`Config` (`config.py`) → single YAML file `~/.agent-reach/config.yaml`, flat key-value, `.get()` falls through to `os.environ`, `.to_dict()` masks secret-shaped keys by name. No schema, no nesting, no versioning. `FEATURE_REQUIREMENTS` dict maps five named features (`exa_search`, `twitter_xreach`, `groq_whisper`, `openai_whisper`, `github_token`) to required config keys — not a general channel-enablement mechanism.

### B.6 Diagnostic map

`agent-reach doctor [--json]` → `doctor.check_all(config)` → iterates `ALL_CHANNELS` unconditionally → each `channel.check(config)` → results dict, every message scrubbed → `format_report()` renders Rich-markup text grouped by tier, or raw JSON with `--json`.

### B.7 Credential-boundary map

Three enforcement layers, all confirmed by source: (1) `Config.to_dict()` name-based masking, (2) `scrub_url_credentials()` content-pattern masking at every output call-site, (3) `cookie_extract.py`'s structural block on automatic Twitter/XiaoHongShu extraction, reinforced independently at the CLI argument-parsing layer. `doctor.py` never reads live credentials from the network — only local files with TTL checks, or local loopback probes.

### B.8 Process-execution map

Every subprocess invocation found across the codebase (`probe.py:98-105`, `transcribe.py`'s yt-dlp/ffmpeg/ffprobe calls, `opencli.py`'s version probe, CLI install helpers) uses **argument-array `subprocess.run([executable, *args], ...)`** — `shell=True` was not found anywhere. Timeouts are set on every observed call. `shutil.which()` pre-checks precede most invocations. No centralized executable allowlist module exists as a named artifact, but the effect is achieved by (a) hardcoded executable names at each call site (never derived from external content) and (b) argument-array invocation eliminating shell-metacharacter injection as a vector.

### B.9 Relevant security controls (summary; see A.15/A.17 for detail)

SSRF-hardened URL validation (`utils/url.py`), SSRF-hardened media-URL validation in the transcribe pipeline (`transcribe.py`, independent implementation), symlink-hardened file I/O (`utils/paths.py`), atomic + `0o600` config writes, content-pattern + key-name secret redaction, provably read-only diagnostics (test-enforced), structural (not just documented) blocking of automatic Twitter/XiaoHongShu cookie extraction.

### B.10 Import-versus-CLI findings (summary)

| Capability | Importable Python? | Notes |
|---|---|---|
| Web/Jina retrieval | **Yes** | `WebChannel.read(url)` |
| ASR transcript fallback | **Yes** | `transcribe.transcribe(source, ...)` |
| Config | **Yes** | `Config` |
| URL validation | **Yes** | `utils.url.*` |
| Secret redaction | **Yes** | `utils.text.scrub_url_credentials` |
| Filesystem hardening | **Yes** | `utils.paths.*` |
| External-binary health probe | **Yes** | `probe.probe_command` |
| Doctor | **Yes** | `doctor.check_all/format_report` |
| RSS parsing | No (feature doesn't exist) — but `feedparser` itself is | Scanner should call `feedparser` directly |
| YouTube metadata/captions | No (feature doesn't exist in AR) — but `yt-dlp` itself is | Scanner should call `yt_dlp.YoutubeDL` directly |
| Exa search | **No** | CLI-only via `mcporter call ...` |
| GitHub content | **No** | CLI-only via `gh ...` (not prioritized for this scanner) |

### B.11 Current Git commit SHA

`93ae1d18c37b707dec053c7c4f9d91cd8ef8943d` (2026-08-12). **`CHANGELOG.md` is stale relative to this commit** — its latest entry is `[1.3.1] - 2026-03-27` while `pyproject.toml`/`__init__.py` both report `version = "1.5.0"`. A dedicated security-hardening commit series postdates the changelog (e.g. `fix(doctor): make health checks truthful and read-only`, `fix(security): make diagnostics provably read-only`, `fix(security): enforce least-privilege credential boundaries`, `fix(security): harden local credential handling`) — this session's security claims are grounded in current source and `git log`, not in the changelog.

### B.12 Current Python and packaging expectations

`requires-python = ">=3.10"` (classifiers list 3.10/3.11/3.12; CI matrix additionally tests 3.13). Build backend `hatchling`. Core deps: `requests>=2.28`, `feedparser>=6.0`, `python-dotenv>=1.0`, `loguru>=0.7`, `pyyaml>=6.0`, `rich>=13.0`, `yt-dlp[default]>=2026.07.04`. Optional extras: `browser` (`playwright`), `cookies` (`browser-cookie3`), `all` (adds `mcp[cli]`), `dev` (`pytest`, `ruff`, `mypy`). `constraints.txt` pins exact tested versions for CI/dev. **`mcp` (the MCP server dependency) is only present via the `all` extra** — a base install has no MCP server capability.

### B.13 Identified extension points

- `Channel` base class — new channels can be added by subclassing and registering in `ALL_CHANNELS` (per `CONTRIBUTING.md`'s documented "Adding New Channels" steps: new file, contract methods, tests in `test_channels.py`, doctor registration, docs update). **This session recommends against adding a channel this way for scanner sources** — see Deliverable C/D — since the `Channel` contract is check()-only and doesn't buy the scanner anything it can't get by direct composition.
- `Config.FEATURE_REQUIREMENTS` — extendable dict for new named-feature credential requirements, if the scanner ever shares a credential concept with Agent Reach (unlikely — scanner sources are public, no credentials needed for the core Tier-1/2/3 universe).
- `probe.probe_command()` — reusable for the scanner's own external-tool health checks (e.g., verifying `yt-dlp`/`ffmpeg` presence) without reimplementing the missing/broken/timeout distinction.

### B.14 Identified unstable or private interfaces

`CLAUDE.md`'s description of `core.py` ("Core read/search routing logic") and of the `Channel` contract ("must implement can_handle(url), read(url), search(query), check()") are **both contradicted by current source** — treat `CLAUDE.md`'s per-file architecture summary as stale documentation, not verified behavior, for these two claims specifically. All internal helper functions prefixed `_` (e.g. `_check_twitter_cli`, `_is_antibot_page`, `_literal_ip_address`) are treated as private/unstable and excluded from the scanner's reuse surface; only the public class/function names cited throughout Deliverable A are treated as the stable surface.

### B.15 Architecture diagrams

**CLI → doctor → channels flow (verified):**

```mermaid
flowchart LR
    A["agent-reach doctor [--json]"] --> B["doctor.check_all(config)"]
    B --> C["channels.get_all_channels() — 15 fixed instances"]
    C --> D1["Channel.check() — local binary/file/loopback probe only"]
    D1 --> E["scrub_url_credentials() on every message"]
    E --> F["format_report() text, or raw JSON"]
```

**Trust boundary relevant to the scanner (Section 6 of the master prompt, mapped onto verified Agent Reach pieces):**

```mermaid
flowchart TB
    U["Untrusted external content\n(webpages, feeds, transcripts, search results)"] -->|"fetched via"| W["WebChannel.read() / feedparser / yt_dlp\n(scanner-invoked, composition only)"]
    W -->|"raw bytes, treated as data only"| N["Scanner-owned normalization + validation\n(never executes or obeys content)"]
    N --> V["Deterministic Python validation gate"]
    V --> EV["Evidence store (scanner-owned, outside Agent Reach)"]
    N -.->|"URL/host checks before any fetch"| SSRF["utils.url.normalize_public_http_url / host_matches\n(reused by composition)"]
    N -.->|"secret scrubbing before persistence"| RED["utils.text.scrub_url_credentials\n(reused by composition)"]
```

No code path in Agent Reach constructs or selects an executable command from external content — every external-facing string is either passed to a fixed-template external command (e.g. `mcporter call exa.web_search_exa query=<value>`) as one argument-array element, or used only as validated request data (URLs to `WebChannel.read`). This is the property the scanner's own adapters must preserve when they add their own command-invoking code (`yt_dlp`, `feedparser`, `mcporter`).

---

## DELIVERABLE C — SCANNER GAP ANALYSIS

Classification taxonomy: **ALREADY SUPPORTED** / **SUPPORTED WITH CONFIGURATION** / **SUPPORTED WITH SCANNER ADAPTER** / **REQUIRES NEW SCANNER MODULE** / **REQUIRES AGENT REACH EXTENSION** / **DEFERRED FROM V1** / **REQUIRES SOURCE VERIFICATION** / **REQUIRES TERMS REVIEW** / **UNRESOLVED OWNER DECISION**.

| # | Requirement | Classification | Current state / evidence | Target state | Ownership | Primary risk | Next validation step |
|---|---|---|---|---|---|---|---|
| 1 | Source registry | **REQUIRES NEW SCANNER MODULE** | Nothing exists in AR (Deliverable A.18/B) | `source_registry.yaml` + loader, scanner-owned | Scanner | Registry design must not couple to AR's `Channel` list | Design registry schema in a follow-up session (deferred to F–M) |
| 2 | Fixed-source monitoring | **REQUIRES NEW SCANNER MODULE** | No polling/collection loop anywhere in AR | Scanner collection orchestrator | Scanner | Duplicate-effort risk if built to mimic `Channel` unnecessarily | Confirm orchestrator design in Deliverable D |
| 3 | Feedparser integration | **SUPPORTED WITH SCANNER ADAPTER** | `feedparser` is a pinned AR dependency (A.3); `RSSChannel` itself does no parsing | Scanner-owned RSS module calling `feedparser.parse()` directly | Scanner | None significant — mature upstream library | Write a minimal adapter against 1 known-good feed URL |
| 4 | Jina retrieval | **ALREADY SUPPORTED** | `WebChannel.read(url)` is real, importable, SSRF-hardened (A.1) | Reuse as-is via composition | Agent Reach (reused) | Antibot/CAPTCHA failures on some sites (visible via `RuntimeError`) | None — reuse directly |
| 5 | YouTube metadata | **SUPPORTED WITH SCANNER ADAPTER** | AR only version-probes `yt-dlp` (A.4); `yt-dlp` itself is a pinned, importable library | Scanner module using `yt_dlp.YoutubeDL` Python API directly | Scanner (using shared dependency) | yt-dlp requires a JS runtime as of the version threshold AR already documents (`_JS_RUNTIMES_SUPPORTED_FROM`) — scanner inherits this operational requirement | Verify `yt-dlp`+JS-runtime works on the target machine |
| 6 | Transcript retrieval | **REQUIRES NEW SCANNER MODULE** | No official/human/auto-caption retrieval exists in AR (A.6/A.7) | Scanner module using `yt_dlp`'s subtitle options directly | Scanner | Model transcript retrieval as expected to fail per master-prompt Section 10.C | Test caption retrieval against 1–2 known Fed/ECB YouTube videos |
| 7 | Transcript fallback (ASR) | **ALREADY SUPPORTED** | `transcribe.transcribe()` is real, importable, SSRF-hardened (A.8) | Reuse as-is via composition, wrapped with rung-tagging | Agent Reach (reused) | Requires Groq/OpenAI API key (owner-provided) | None — reuse directly once a key exists |
| 8 | Exa discovery | **SUPPORTED WITH SCANNER ADAPTER** | AR only inspects `mcporter` config (A.9); no Python search call exists | Scanner module with a fixed `mcporter call exa.web_search_exa` command template | Scanner | CLI-invocation security surface (argument-array, no shell string) | Verify `mcporter`+Exa MCP server actually returns results end-to-end |
| 9 | GitHub functionality | **DEFERRED FROM V1** | `gh`-CLI health-check only (A.10); not part of the institutional-source universe | Not needed for V1 | N/A | None | None |
| 10 | Calendar intelligence | **REQUIRES NEW SCANNER MODULE** | Nothing in AR | Scanner-owned calendar module + lifecycle states (Section 12) | Scanner | Depends on unresolved consensus-provider decision (N.1) | See N.1, N.2 |
| 11 | Consensus data | **UNRESOLVED OWNER DECISION** | No AR capability; no provider chosen (per master-prompt Section 12) | Owner must approve a provider or accept "unavailable" | Owner | Fabricated-consensus risk if skipped | See N.1 |
| 12 | Market observations | **REQUIRES NEW SCANNER MODULE** | No market-data capability anywhere in AR | Scanner module wrapping `yfinance` (external, not an AR dependency) or an approved alternative | Scanner | Symbol-mapping correctness (Section 11/24) | Test a small representative symbol subset (see O) |
| 13 | Symbol verification | **REQUIRES SOURCE VERIFICATION** | N/A to AR; scanner-owned verification procedure | Verification checklist defined in Section 24, executed against representative subset | Scanner | Wrong-contract/rollover risk | Execute against NQ/VXN/HYG/one duration proxy |
| 14 | Timestamp normalization | **REQUIRES NEW SCANNER MODULE** | No timezone-normalization logic found in AR | Scanner-owned, America/New_York throughout | Scanner | Low — well-understood problem | None beyond implementation |
| 15 | Deduplication | **REQUIRES NEW SCANNER MODULE** | `RSSChannel` has zero dedup logic (A.3); no dedup anywhere else | Scanner-owned GUID/URL/title/content-hash dedup | Scanner | Low | None beyond implementation |
| 16 | Syndication grouping | **REQUIRES NEW SCANNER MODULE** | Not present in AR | Scanner-owned clustering by canonical URL/content similarity | Scanner | Medium — heuristic quality | Design in a follow-up session (deferred to F–M) |
| 17 | Revision tracking | **REQUIRES NEW SCANNER MODULE** | Not present in AR | Scanner-owned content-hash diffing | Scanner | Low | None beyond implementation |
| 18 | Source authority | **REQUIRES NEW SCANNER MODULE** | Not present in AR (AR channels don't carry an "authority tier" concept — `tier` on `Channel` means UI-grouping by install-criticality, not source-authority per Section 13) | Scanner-owned authority-tier framework (Section 13) | Scanner | Must not conflate AR's `Channel.tier` with the scanner's authority tiers — different concepts, same word | Note explicitly in scanner code/docs to avoid confusion |
| 19 | Source concentration | **REQUIRES NEW SCANNER MODULE** | Not present in AR | Scanner-owned (Section 13) | Scanner | Medium — requires evidence-clustering to exist first | Depends on item 16 |
| 20 | Evidence lifecycle/extraction/validation/clustering | **REQUIRES NEW SCANNER MODULE** | Not present in AR | Scanner-owned (Sections 15/16) | Scanner | High complexity — deferred detail to F–M | Design in follow-up session |
| 21 | Narrative synthesis | **REQUIRES NEW SCANNER MODULE** | Not present in AR | Scanner-owned LLM-assisted + deterministic-validated (Section 16) | Scanner | Requires unresolved LLM-provider decision (N.5/N.6) | See N.5, N.6 |
| 22 | Contradiction search | **SUPPORTED WITH SCANNER ADAPTER** | Exa discovery adapter (item 8) can serve this, no dedicated AR capability | Scanner module reusing the Exa adapter with contradiction-focused queries | Scanner | Same as item 8 | Same as item 8 |
| 23 | Morning-archetype selection | **REQUIRES NEW SCANNER MODULE** | Not present in AR | Scanner-owned (Section 16) | Scanner | Medium | Deferred to F–M |
| 24 | Regime classification | **REQUIRES NEW SCANNER MODULE** | Not present in AR | Scanner-owned (Section 19) | Scanner | Medium | Deferred to F–M |
| 25 | Directional assessment | **REQUIRES NEW SCANNER MODULE** | Not present in AR | Scanner-owned, LLM-proposes/Python-validates (Section 17) | Scanner | High — this is the core research-boundary-sensitive logic | Deferred to F–M, with extra scrutiny against Section 1's boundary |
| 26 | Confidence validation | **REQUIRES NEW SCANNER MODULE** | Not present in AR | Scanner-owned deterministic caps (Section 18) | Scanner | Medium | Deferred to F–M |
| 27 | Canonical report state | **REQUIRES NEW SCANNER MODULE** | Not present in AR | Scanner-owned single source-of-truth object (Section 20/28) | Scanner | High — Markdown/JSON must render from one object, not two | Deferred to F–M |
| 28 | Markdown rendering | **REQUIRES NEW SCANNER MODULE** | Not present in AR | Scanner-owned renderer reading canonical state | Scanner | Low once canonical state exists | Deferred to F–M |
| 29 | JSON rendering | **REQUIRES NEW SCANNER MODULE** | Not present in AR | Scanner-owned renderer reading canonical state | Scanner | Low once canonical state exists | Deferred to F–M |
| 30 | SQLite persistence | **REQUIRES NEW SCANNER MODULE** | No `sqlite3` usage anywhere in AR (A.20) | Scanner-owned, per Section 22 | Scanner | Low — SQLite is well-suited to single-user local use; confirmed no AR conflict | Confirm no concurrency conflict with scheduled runs (N — scheduling) |
| 31 | Filesystem artifacts | **REQUIRES NEW SCANNER MODULE** | No `data/` convention in AR | Scanner-owned, per Section 22's suggested layout | Scanner | Low | None beyond implementation |
| 32 | Scheduling | **REQUIRES NEW SCANNER MODULE** + **UNRESOLVED OWNER DECISION** | No scheduler anywhere in AR (A.19) | Scanner-owned, OS-appropriate mechanism | Scanner + Owner | Depends on actual OS (N.3) | See N.3, N.4 |
| 33 | Health monitoring | **SUPPORTED WITH SCANNER ADAPTER** | AR's `probe_command` pattern is reusable for scanner's own external-tool checks (`yt-dlp`, `ffmpeg`) | Scanner-owned source/capability health system, reusing `probe.probe_command` for tool checks | Scanner (reusing AR utility) | Low | None beyond implementation |
| 34 | Publication integrity | **REQUIRES NEW SCANNER MODULE** | Not present in AR | Scanner-owned gate (Section 28) | Scanner | High — must block on failed checks, not just warn | Deferred to F–M |
| 35 | Shadow-mode evaluation | **DEFERRED FROM V1** (per master prompt Section 29 framing, though the data-capture hooks for it should be designed now) | Not present in AR | Scanner-owned, later phase | Scanner | N/A this session | Identify what fields must be preserved now (see O) |
| 36 | Human feedback | **REQUIRES NEW SCANNER MODULE** | Not present in AR | Scanner-owned, vocabulary in Section 14 | Scanner | Low priority for V1 | Deferred to F–M |
| 37 | Security controls | **SUPPORTED WITH SCANNER ADAPTER** | AR provides real, reusable primitives: `utils.url.*` (SSRF), `utils.text.scrub_url_credentials` (redaction), argument-array subprocess pattern (A.15/A.17/B.8) | Scanner reuses these directly via composition and extends with its own command-execution allowlist for `yt-dlp`/`feedparser`/`mcporter` invocations | Scanner (reusing AR utilities) + Scanner-owned extensions | Medium — scanner's own new command-invocation surface (Exa via mcporter) must inherit the same argument-array discipline | Code review of the first scanner adapter against Section 6's checklist |

**Overall pattern:** the large majority of Section 34's checklist lands in **REQUIRES NEW SCANNER MODULE**, exactly as the evidence in Deliverable A predicts — Agent Reach's actual Python surface is narrow (install + doctor + a handful of directly-reusable utilities), not a content or reasoning engine. Nothing here surprises against the master prompt's own framing ("Agent Reach is a general capability and connectivity layer... not the evidence model... not the regime engine"); it does, however, correct `CLAUDE.md`'s and some casual documentation's overstatement of `core.py`'s role, which matters because it changes where the scanner's adapter boundary should sit (composition around a few specific functions, not subclassing `Channel`).

---

## DELIVERABLE D — RECOMMENDED TARGET ARCHITECTURE

### D.1 Guiding principle

Per Section 3's composition preference, **no Agent Reach core extension is proposed**. Every genuinely reusable AR capability (`WebChannel.read`, `transcribe.transcribe`, `utils.url.*`, `utils.text.scrub_url_credentials`, `utils.paths.*`, `probe.probe_command`, `Config`) is consumed by **import and direct call**, pinned to the recorded commit SHA (Section 4). Everywhere AR has no Python surface (RSS, YouTube metadata/captions, Exa), the scanner either (a) calls an already-pinned shared dependency directly (`feedparser`, `yt_dlp`), or (b) owns a narrow, fixed-template CLI adapter (`mcporter`) — never by subclassing `Channel` or registering into `ALL_CHANNELS`, since that contract offers nothing beyond `check()` and would couple the scanner's install-time diagnostics to AR's doctor system for no benefit.

### D.2 Component boundaries

| Component | Responsibility | Owner package | Inputs | Outputs | Trust level | Invokes AR? | Can execute commands? | Receives raw untrusted content? | Deterministic validation follows? | Owner-decision dependency |
|---|---|---|---|---|---|---|---|---|---|---|
| **Agent Reach capability boundary** | Provide `WebChannel.read`, `transcribe.transcribe`, `utils.*`, `Config`, `probe_command` at a pinned commit | `agent_reach` (external, pinned) | URLs, media sources | Raw text/JSON | Untrusted output, trusted code | N/A | Yes (yt-dlp/ffmpeg, argument-array, AR-internal) | Yes (this is its job) | No (AR does not validate scanner semantics) | None |
| **Scanner adapter boundary** | Thin wrappers around AR functions + shared deps (`feedparser`, `yt_dlp`, `mcporter`), tagging retrieval metadata | `nq_scanner.adapters.*` | URLs/queries | Tagged raw records (source, retrieval time, content hash) | Receives untrusted content, must not act on it | Yes, by composition | Yes, fixed templates only (`mcporter call ...`) | Yes | No (adapters are collection-only, Wave 1) | None |
| **Shell-execution boundary** | Fixed-template invocation of `yt-dlp`/`ffmpeg`/`mcporter`, argument-array only | `nq_scanner.adapters.exec` | Fixed command + validated args | Subprocess stdout/stderr | Command-construction is privileged; content is not | No (parallel pattern, not shared code) | Yes | No — content never selects the executable or template | N/A | None |
| **Untrusted-content boundary** | Structured wrapper (`untrusted_external_content` per Section 6) around every fetched item before any further processing | `nq_scanner.ingestion.boundary` | Adapter output | `UntrustedContent{source_url, publisher, content}` dataclass | Boundary object — everything past this point treats `.content` as data only | No | No | Yes | Enforces "never obey instructions in content" at the type level | None |
| **Source-registry boundary** | Config-driven source activation, authority tiers, validation states (Section 8) | `nq_scanner.registry` | `source_registry.yaml` | Activated source objects | Trusted config, not content | No | No | No | Gate: no source activates without required validation state | Initial critical-source set (N.12) |
| **Collection orchestration** | Wave 1 fan-out across adapters per run | `nq_scanner.pipeline.collect` | Registry + calendar | Raw candidate items | Orchestration logic, trusted | Via adapters | No directly | No directly (delegates) | No | Scheduling mechanism (N.4) |
| **Calendar input** | Official-calendar polling, event lifecycle | `nq_scanner.calendar` | Official calendar sources (via Jina/RSS adapters) | Calendar events with lifecycle state | Receives untrusted content via adapters | Via adapters | No | Yes (via adapters) | Yes | Consensus provider (N.1) |
| **Consensus input** | Consensus-figure ingestion, if approved | `nq_scanner.calendar.consensus` | Approved provider or manual file | Consensus values | Untrusted (external) or operator-trusted (manual file) | Maybe (if provider needs an adapter) | No | Yes (if external) | Yes | Provider choice (N.1) |
| **Market observations** | Delayed public market-data ingestion | `nq_scanner.market` | `yfinance` or approved alternative (not an AR dependency — new) | Tagged market observations with freshness | Untrusted (external) | No | No | Yes | Yes (freshness policy, Section 11) | Provider/symbol scope (N.16–N.19) |
| **Raw-item ownership** | Normalizes adapter output into a common raw-item shape | `nq_scanner.ingestion.raw_item` | Untrusted-content objects | `RawItem` records | Boundary-adjacent, treats content as data | No | No | Yes | Partially (Wave 2 starts here) | None |
| **Document and transcript retrieval** | Wave 3 deep retrieval (Jina full text, transcripts) | `nq_scanner.retrieval` | Prioritized candidates | Documents/transcripts | Receives untrusted content | Yes (`WebChannel.read`, `transcribe.transcribe`) | Yes (via `transcribe.transcribe`'s internal yt-dlp/ffmpeg calls) | Yes | No (feeds Wave 4) | None |
| **Normalization** | Timestamp/URL canonicalization, hashing | `nq_scanner.ingestion.normalize` | Raw items | Normalized items | Trusted logic over untrusted data | Uses `utils.url.*` | No | Yes (as data) | Yes | None |
| **Deduplication** | GUID/URL/title/hash dedup | `nq_scanner.ingestion.dedup` | Normalized items | Deduplicated items | Trusted logic | No | No | Yes (as data) | Yes | None |
| **Evidence extraction** | Wave 4 structured-fact extraction (LLM-assisted) | `nq_scanner.evidence.extract` | Documents/transcripts | Extracted facts | Receives untrusted content **into an LLM context** — highest-sensitivity boundary for prompt injection | No | No | Yes | Yes, post-extraction | LLM provider/model (N.5, N.6) |
| **Evidence validation** | Deterministic checks on extracted facts | `nq_scanner.evidence.validate` | Extracted facts | Validated evidence | Trusted logic | No | No | No (operates on structured output only) | Yes | None |
| **Evidence clustering** | Wave 5 grouping into clusters | `nq_scanner.evidence.cluster` | Validated evidence | Clustered evidence | Trusted logic | No | No | No | Yes | None |
| **Narrative synthesis** | Wave 6 LLM-assisted narrative + regime proposal | `nq_scanner.synthesis.narrative` | Clustered evidence | Proposed narrative/regime | LLM-context boundary again | No | No | Indirectly (already-extracted facts, lower risk than raw content) | Yes (Wave 7 validates) | LLM provider/model (N.5, N.6) |
| **Contradiction analysis** | Active search for counter-evidence | `nq_scanner.synthesis.contradiction` | Clusters + Exa adapter | Contradiction records | Same as narrative synthesis | Via Exa adapter | Yes (via adapter) | Yes | Yes | Same as narrative |
| **Morning-archetype validation** | Deterministic check of LLM-proposed archetype | `nq_scanner.synthesis.archetype` | Proposed archetype + evidence | Validated archetype | Trusted logic | No | No | No | Yes | None |
| **Regime analysis** | Wave 6/19 regime assignment | `nq_scanner.synthesis.regime` | Validated evidence/archetype | Regime + modifier | Trusted logic, LLM-proposed | No | No | No | Yes | None |
| **Directional assessment** | Wave 7 long/short environmental status | `nq_scanner.synthesis.directional` | Clusters, regime, market obs | Status + confidence | LLM-proposes, Python-validates (Section 17/18) | No | No | No | Yes — this is the deterministic gate itself | None |
| **Confidence validation** | Deterministic caps (Section 18) | `nq_scanner.synthesis.confidence` | All upstream assessments | Final confidence labels | Trusted logic | No | No | No | Yes, this **is** the validation | None |
| **Canonical report state** | Wave 8 single source-of-truth object | `nq_scanner.report.state` | All synthesis outputs | One `ReportState` object | Trusted logic | No | No | No | Enforces publication gate | None |
| **Markdown rendering** | Renders `ReportState` → Markdown | `nq_scanner.report.render_md` | `ReportState` | Markdown file | Trusted logic | No | No | No | Reads validated state only | None |
| **JSON rendering** | Renders `ReportState` → JSON | `nq_scanner.report.render_json` | `ReportState` | JSON file | Trusted logic | No | No | No | Reads validated state only | None |
| **SQLite persistence** | Structured operational state (Section 22) | `nq_scanner.storage.db` | All pipeline stages | Persisted rows | Trusted logic | No | No | Stores untrusted content as opaque text/blobs only | N/A (storage, not validation) | None |
| **Filesystem artifact storage** | Reports/documents/transcripts on disk (Section 22) | `nq_scanner.storage.fs` | Documents, transcripts, reports | Files under `data/` | Trusted logic; stored content still untrusted | No | No | Yes (stores it) | N/A | Storage root (N.22) |
| **Scheduling boundary** | Triggers runs at target times | `nq_scanner.scheduling` | OS scheduler | Run invocations | Trusted logic, OS-dependent | No | Possibly (invokes the scanner's own entry point) | No | N/A | OS + mechanism (N.3, N.4) |
| **Source and capability health** | Wave "0" pre-run checks | `nq_scanner.health` | Registry, AR's `probe_command` | Health status | Trusted logic | Yes (`probe.probe_command` reused) | Yes (probes only) | No | Yes | None |
| **Publication integrity** | Wave 8 gate (Section 28) | `nq_scanner.report.gate` | `ReportState` | Pass/fail + degraded report | Trusted logic | No | No | No | This **is** the gate | None |
| **Security policy** | Command allowlists, network controls, secret handling | `nq_scanner.security` | N/A (policy/config) | Enforced constraints | Trusted, security-critical | Reuses `utils.url.*`/`utils.text.*` | N/A | N/A | N/A | None |
| **Evaluation and feedback** | Shadow-mode metrics, human feedback capture | `nq_scanner.evaluation` | Reports + human input | Metrics | Trusted logic | No | No | No | N/A | Deferred (Section 29) |

### D.3 Trust-boundary pipeline diagram

```mermaid
flowchart TB
    subgraph AR["Agent Reach (pinned commit, external dependency)"]
        W["WebChannel.read()"]
        T["transcribe.transcribe()"]
        U["utils.url / utils.text / utils.paths"]
        P["probe.probe_command()"]
    end
    subgraph ADAPT["Scanner adapters (composition only)"]
        RSSA["RSS adapter (feedparser direct)"]
        YTA["YouTube adapter (yt_dlp direct)"]
        EXAA["Exa adapter (mcporter, fixed template)"]
        MKT["Market-data adapter (yfinance/approved)"]
    end
    RSSA --> BOUND
    YTA --> BOUND
    EXAA --> BOUND
    MKT --> BOUND
    W --> BOUND
    T --> BOUND
    BOUND["untrusted_external_content boundary\n(source_url, publisher, content — data only, never obeyed)"]
    BOUND --> NORM["Normalization + validation (Wave 2)\nuses utils.url for revalidation, utils.text for redaction"]
    NORM --> DEDUP["Dedup + syndication grouping"]
    DEDUP --> EXTRACT["Evidence extraction (LLM context — highest injection risk)"]
    EXTRACT --> VALID["Deterministic evidence validation"]
    VALID --> CLUSTER["Evidence clustering"]
    CLUSTER --> SYNTH["Narrative + regime synthesis (LLM-proposes)"]
    SYNTH --> DIR["Directional assessment (Python validates)"]
    DIR --> STATE["Canonical ReportState (one object)"]
    STATE --> GATE["Publication integrity gate"]
    GATE -->|"pass"| REPORTS["Markdown + JSON (from same state)"]
    GATE -->|"fail"| DEGRADED["Degraded/error report, state preserved for debugging"]
```

No arrow in this diagram allows content to flow from `BOUND` (or anything downstream of it) into the **shell-execution boundary** — command construction only ever happens inside the adapters, using fixed templates and arguments that are validated data (a URL, a query string passed as one argument-array element), never a full command string built from external content.

---

## DELIVERABLE E — PROPOSED REPOSITORY LAYOUT

Per Section 3's "keep scanner-specific concerns outside Agent Reach core" directive and Section 36, the scanner should live as a **sibling package**, not inside `agent_reach/`. Two placement options exist (see N — this is genuinely undecided): (a) a new top-level directory in *this* repository, or (b) a wholly separate repository that declares Agent Reach as a pinned dependency. The tree below is written assuming option (a) for concreteness; it maps directly onto option (b) by simply becoming the separate repo's root.

```
nq_scanner/                        # NOT created this session — proposed only
├── nq_scanner/                    # Python package
│   ├── adapters/                  # Composition wrappers around AR + shared deps
│   │   ├── web_jina.py            #   wraps agent_reach.channels.web.WebChannel
│   │   ├── rss.py                 #   wraps feedparser directly
│   │   ├── youtube.py             #   wraps yt_dlp.YoutubeDL directly
│   │   ├── transcript_asr.py      #   wraps agent_reach.transcribe.transcribe
│   │   ├── exa_search.py          #   fixed mcporter command template
│   │   └── market_data.py         #   wraps yfinance or approved alternative
│   ├── ingestion/                 # Wave 1–2: boundary, raw item, normalize, dedup
│   ├── calendar/                  # Wave 1: calendar intelligence, consensus
│   ├── market/                    # Wave 1/3: market observation orchestration
│   ├── evidence/                  # Wave 4–5: extract, validate, cluster
│   ├── synthesis/                 # Wave 6–7: narrative, regime, directional
│   ├── report/                    # Wave 8: canonical state, gate, renderers
│   ├── registry/                  # Source-registry loading/validation
│   ├── storage/                   # SQLite + filesystem artifact ownership
│   ├── scheduling/                # OS-appropriate run triggers
│   ├── health/                    # Source/capability health checks
│   ├── security/                  # Shared security policy (reuses AR utils)
│   └── evaluation/                # Shadow-mode metrics, human feedback (later phase)
├── config/                        # Versioned policy YAML (Section 23's file list)
│   ├── source_registry.yaml
│   ├── instruments.yaml / market_symbols.yaml
│   ├── calendars.yaml / calendar_priorities.yaml
│   ├── authority_tiers.yaml / source_roles.yaml
│   ├── controlled_vocabularies.yaml
│   ├── evidence_clusters.yaml / cluster_weights.yaml
│   ├── morning_archetypes.yaml / regime_vocabulary.yaml
│   ├── directional_thresholds.yaml / confidence_caps.yaml
│   ├── report_schemas.yaml
│   ├── retention.yaml / run_budgets.yaml / retry_policy.yaml
│   └── security_policy.yaml
├── prompts/                       # Runtime LLM prompt templates (content deferred to Deliverable I)
├── migrations/                    # SQLite schema migrations
├── templates/                     # Report Markdown/JSON templates
├── tests/                         # Scanner-owned test suite (mirrors nq_scanner/ package layout)
├── fixtures/                      # Recorded sample responses for offline tests
├── docs/                          # Scanner-specific documentation
├── planning/                      # This session's artifacts + future architecture sessions
├── data/                          # Runtime artifacts — see Section 22 layout (reports/, documents/, transcripts/, raw/, logs/, feedback/, exports/)
└── pyproject.toml                 # Declares agent_reach as a pinned dependency (exact commit or version)
```

| Top-level item | Source-controlled? | Runtime data? | Touches untrusted content? | Accesses secrets? | Executes commands? | Inside or outside AR core? |
|---|---|---|---|---|---|---|
| `nq_scanner/` (package) | Yes | No | Yes (by design, as data) | Only LLM/market-data API keys, via `Config`-equivalent | Yes (`adapters/`, `security/` only) | Outside — new sibling package |
| `config/` | Yes | No | No | No | No | Outside |
| `prompts/` | Yes | No | No (templates only; filled content is untrusted data at runtime, not stored here) | No | No | Outside |
| `migrations/` | Yes | No | No | No | No | Outside |
| `templates/` | Yes | No | No | No | No | Outside |
| `tests/` | Yes | No | Fixtures may contain sanitized untrusted-content samples | No | Test-only, sandboxed | Outside |
| `fixtures/` | Yes | No | Yes, deliberately (recorded samples) | No | No | Outside |
| `docs/` | Yes | No | No | No | No | Outside |
| `planning/` | Yes | No | No | No | No | Outside |
| `data/` | **No** (gitignored) | Yes | Yes | **No — secrets must never land here (Section 22)** | No | Outside |

No files or directories from this layout are created during this session, per the master prompt's explicit instruction.

---

## DELIVERABLE N — UNRESOLVED DESIGN DECISIONS

Only genuine owner decisions or externally unresolved facts are listed; nothing already settled by Parts 1/2 is repeated as a question.

### N.1 Consensus-data provider

- **Why it matters:** Directional assessment quality depends on knowing whether an economic release beat/missed consensus; fabricating consensus is explicitly forbidden (Section 12).
- **Current evidence:** No provider is referenced anywhere in Agent Reach (it's out of scope for a connectivity layer); no evaluation of any specific provider was performed this session (explicitly deferred by the master prompt).
- **Options:** (a) a free public aggregator if one with acceptable methodology disclosure and automated-use terms can be identified, (b) a paid data vendor, (c) manual operator-input file before each morning run, (d) "consensus unavailable" disclosure with no numeric consensus at all.
- **Tradeoffs:** (a) methodology/reliability risk; (b) cost + vendor terms review; (c) removes automation but is simplest/safest; (d) weakens directional-assessment quality but is fully safe.
- **Recommended next validation step:** Owner identifies 1–2 candidate providers (or confirms manual-input is acceptable); a follow-up session evaluates automated-use permissibility and update timing.
- **Owner approval required:** Yes. **Blocks first vertical slice:** No (the slice can use a non-calendar source). **Blocks eventual 8:00 report:** Yes, if calendar/consensus features are included in that report. **Safely deferrable:** Yes, for the first vertical slice.

### N.2 Whether manual consensus input is acceptable

- **Why it matters:** Determines whether N.1 can be resolved without a new external vendor relationship.
- **Current evidence:** Master prompt explicitly proposes this as one acceptable fallback (Section 12).
- **Options:** Accept manual pre-market operator input as the V1 default vs. requiring an automated provider before any report ships.
- **Recommended next step:** Owner confirms preference.
- **Owner approval required:** Yes. **Blocks first slice:** No. **Safely deferrable:** Yes.

### N.3 Actual operating system

- **Why it matters:** Section 21 requires the scheduling mechanism to be chosen only after inspecting the actual operating environment.
- **Current evidence:** This session ran on Windows (per environment metadata: `Platform: win32`, PowerShell primary shell) — **but that is this Claude Code session's execution environment, not necessarily confirmed as the machine that will run the scanner in production.** Treat as `ASSUMED`, not verified, until the owner confirms the target deployment machine.
- **Options:** Windows Task Scheduler, WSL + cron, a dedicated Linux/macOS host with cron/systemd-timer/launchd, or a cloud scheduler (explicitly deferred/discouraged for V1 per Section 21's "no cloud-hosted scheduler" assumption).
- **Recommended next step:** Owner confirms the actual machine/OS that will run scheduled scanner jobs.
- **Owner approval required:** Yes. **Blocks first slice:** No (a manual `python -m nq_scanner run` invocation suffices for the vertical slice). **Blocks 8:00 report automation:** Yes. **Safely deferrable:** Yes, for the first slice.

### N.4 Scheduling mechanism

- Depends directly on N.3. Options narrow once the OS is confirmed (Task Scheduler XML/`schtasks` vs. cron vs. systemd timer vs. launchd vs. APScheduler-as-a-long-running-process). **Owner approval required:** Yes. **Safely deferrable:** Yes, for the first slice.

### N.5 Runtime LLM provider

- **Why it matters:** Waves 4/6/7 all require LLM calls (extraction, synthesis, directional proposal); Deliverable I (prompt architecture) is fully blocked without this.
- **Current evidence:** Agent Reach itself has no LLM dependency in its core `dependencies` list; `mcp[cli]` is only an optional extra for a different purpose (exposing AR's own doctor report as an MCP tool, not for calling an LLM). No provider is implied anywhere in the repo.
- **Options:** Anthropic Claude API, OpenAI API, a local/open-weight model, or a provider-agnostic abstraction layer.
- **Recommended next step:** Owner selects a provider (and confirms budget/rate-limit expectations) before any Deliverable I session.
- **Owner approval required:** Yes. **Blocks first slice:** Yes, if the slice includes any extraction/synthesis step — **not** if the first slice is scoped to collection→evidence-object→canonical-state without LLM-assisted extraction (a defensible way to unblock a slice while this is pending). **Safely deferrable:** Only if the first slice is scoped that narrowly.

### N.6 Runtime LLM model

- Depends on N.5. Same blocking profile as N.5.

### N.7 LLM credential approach

- **Why it matters:** Section 6 forbids secrets in reports/logs/repo; the scanner needs its own credential-storage decision (reuse `agent_reach.Config`'s pattern by composition, or an entirely separate scanner-owned secrets file with the same hardening as `utils.paths.atomic_write_private_text`).
- **Options:** (a) scanner writes its own config file using AR's `utils.paths`/`utils.text` primitives directly (recommended — reuses proven hardening without coupling to AR's `Config` schema), (b) scanner adds keys into AR's own `~/.agent-reach/config.yaml` via `Config.set()` (not recommended — conflates AR's and the scanner's configuration lifecycles).
- **Recommended next step:** Owner confirms preference; this session recommends (a).
- **Owner approval required:** Yes (architectural preference, low stakes). **Safely deferrable:** Yes.

### N.8 BLS access from the operator's intended network

- **Why it matters:** Section 25 flags prior host-level BLS access restrictions from a datacenter environment; the operator's actual network may differ.
- **Current evidence:** Not tested this session (would require live network calls from the target machine, out of scope for a source-inspection-only session and not safely performable from this environment without confirming it matches the eventual production network).
- **Options:** Test direct reachability from the actual operator machine in a follow-up step; if restricted, evaluate sanctioned alternatives (official release pages via a different route, or accept Tier-1 BLS coverage as degraded/unavailable rather than proxying around a bot restriction).
- **Owner approval required:** No (technical validation), but the *outcome* may require owner sign-off on a coverage gap. **Blocks 8:00 report for CPI/NFP/PCE-type releases:** Yes, if unresolved. **Safely deferrable:** Yes, for the first vertical slice if it doesn't use a BLS source.

### N.9 Reuters discovery and retrieval route

- **Why it matters:** Section 8 explicitly warns not to assume historical Reuters RSS endpoints still exist.
- **Current evidence:** Not tested this session (no source URLs were activated, per session restrictions).
- **Options:** Exa-based discovery, direct permitted public-page retrieval via Jina, or excluding Reuters from V1's fixed-source set and relying on other Tier-3 sources.
- **Recommended next step:** Runtime verification in a dedicated source-validation session.
- **Owner approval required:** No (technical), but terms-of-use review may be needed depending on route chosen. **Safely deferrable:** Yes.

### N.10 Treasury official access routes

- Same pattern as N.9 — Section 8 warns against assuming historical press-RSS paths are valid; Treasury Fiscal Data / official auction datasets need runtime verification. **Safely deferrable:** Yes.

### N.11 Agent Reach import versus CLI use

- **Why it matters:** This session's own Deliverable A/B answers this in detail for every inspected capability — but the *scanner's development team* (owner) should explicitly ratify the recommended pattern (composition-by-import for `WebChannel`/`transcribe`/`utils.*`; direct shared-dependency use for `feedparser`/`yt_dlp`; fixed-template CLI adapter for `mcporter`) before implementation begins, since it determines the scanner's dependency-pinning strategy.
- **Owner approval required:** Yes (architectural sign-off, not new research). **Blocks first slice:** Yes, informally — the slice should be built against an approved pattern. **Safely resolved now:** This session recommends approval of the pattern as described in Deliverable D.

### N.12 Initial critical-source set

- **Why it matters:** Section 8 requires every source-registry entry to pass a verification gate before activation; no sources were activated this session (explicitly out of scope).
- **Options:** Owner/architect selects a small initial set (e.g., Federal Reserve press releases RSS, one Treasury official page, NQ/ES/VIX symbols) for the first source-verification pass.
- **Owner approval required:** Yes. **Blocks first vertical slice:** Yes — the slice needs at least one verified source. **Recommended next step:** A dedicated source-verification session, using the method defined in Deliverable O's first-slice recommendation.

### N.13 Source terms requiring review

- Depends on N.12 — once candidate sources are chosen, each needs an automation/terms-of-service check (explicitly required by Section 8's `REQUIRES_TERMS_REVIEW` state). **Owner approval required:** Likely yes for any borderline source. **Safely deferrable:** No, blocks activation of any specific source, but doesn't block choosing which sources to review first.

### N.14 Feasibility of the 25-minute Deep run

- **Why it matters:** Section 26 requires a latency/workload assessment before committing to the candidate workload (up to 30 Jina docs, 15 transcripts, 15 Exa queries, 40 full-text LLM items).
- **Current evidence:** No runtime load test was performed this session (would require activating real sources, out of scope). Qualitatively: `WebChannel.read()` has a 30s timeout and no built-in concurrency; `transcribe.transcribe()`'s `yt-dlp` download step alone has a 1800s (30-minute) internal timeout ceiling per call, which **on its own approaches or exceeds the entire 25-minute budget for a single transcript** if a download is slow — this is a real feasibility signal, not a hypothetical one.
- **Recommended next step:** Once N.12's initial source set is chosen, run an actual timed Deep-run rehearsal with real (but budget-capped) source counts.
- **Owner approval required:** No (technical), but budget cuts (Section 26) may need owner sign-off if the 25-minute target isn't achievable at the desired scope.

### N.15 Initial processing budgets

- Depends on N.14's outcome. **Safely deferrable:** Yes, until after a rehearsal.

### N.16 Initial instrument deferrals

- **Why it matters:** Section 11 lists candidate deferrals (30Y yield, ZB, ZF, USD/CNH, Copper, international indices, BTC) but Section 24 explicitly says not to auto-defer the 30-year yield or ZB.
- **Options:** Owner ratifies the master prompt's own default guidance (defer USD/CNH, Copper, international indices, BTC; keep 30Y/ZB in scope; treat ZF as a secondary/evaluate-later input) or adjusts it.
- **Owner approval required:** Yes. **Safely deferrable:** Yes, doesn't block the first vertical slice (which needs only NQ + a couple of confirmation instruments).

### N.17 Inclusion of the U.S. 30-year yield / N.18 Inclusion of ZB / N.19 Treatment of ZF

- Each requires a provider-availability/freshness/reliability check (Section 24) not performed this session (no market-data provider was activated). **Owner approval required:** Yes, once data is gathered. **Safely deferrable:** Yes.

### N.20 Audio-transcription fallback timing

- **Why it matters:** `transcribe.transcribe()` (A.8) is real and reusable, but *when* the scanner should invoke it (immediately when official captions are missing, vs. only after a delay to let official transcripts post) is a policy choice, not a technical one.
- **Options:** Immediate fallback (fastest but always burns Groq/OpenAI budget), delayed fallback (waits N minutes for official captions first), or fallback only on the 8:00 Deep run (never on 9:00/10:00 incrementals).
- **Owner approval required:** Yes. **Safely deferrable:** Yes.

### N.21 Initial international-market scope

- Section 24 lists international indices as a deferrable candidate. **Owner approval required:** Yes. **Safely deferrable:** Yes.

### N.22 Storage root and backup expectations

- **Why it matters:** Section 22 proposes a `data/` layout but doesn't fix where it lives (repo-adjacent vs. a separate data volume) or whether/how it's backed up.
- **Options:** Repo-adjacent `data/` (gitignored) on the same machine as the scheduler, vs. a separate storage location with its own backup policy.
- **Owner approval required:** Yes. **Safely deferrable:** Yes, for the first slice (a local `data/` directory suffices).

### N.23 Whether reports require delivery beyond local Markdown/JSON

- **Why it matters:** Section 20 only specifies Markdown+JSON generation; it doesn't say whether reports must also be emailed, posted to Slack, etc.
- **Options:** Local-file-only (simplest, matches the literal spec), vs. an additional delivery channel (each with its own credential/security surface that would need review).
- **Owner approval required:** Yes. **Safely deferrable:** Yes — local-file-only is a safe default for V1 and for the first vertical slice.

### N.24 Any required API credentials not already available

- **Why it matters:** Groq/OpenAI (transcription fallback), and whatever LLM provider is chosen (N.5), and any market-data provider (item 12/C) all need credentials the owner must supply; none currently exist in this repo's `.env.example` beyond `GROQ_API_KEY`/`OPENAI_API_KEY`/`EXA_API_KEY`/`GITHUB_TOKEN`/`REDDIT_PROXY` (and the last two are irrelevant to this scanner).
- **Recommended next step:** Owner confirms which of Groq/OpenAI/chosen-LLM-provider/market-data-provider keys already exist vs. need to be obtained.
- **Owner approval required:** Yes. **Blocks first vertical slice:** Only if the slice needs LLM extraction or ASR fallback — the narrowly-scoped slice recommended in Deliverable O does not strictly require either.

---

## DELIVERABLE O — IMPLEMENTATION-READINESS ASSESSMENT

**Current Agent Reach commit SHA:** `93ae1d18c37b707dec053c7c4f9d91cd8ef8943d` (2026-08-12).

**What Agent Reach already supports (reusable now, via composition, pinned to this commit):**
- `WebChannel.read(url)` — full-text web/Jina retrieval, SSRF-hardened.
- `transcribe.transcribe(source, ...)` — Whisper ASR fallback pipeline, SSRF-hardened.
- `utils.url.normalize_public_http_url / host_matches / domain_matches` — SSRF-safe URL handling.
- `utils.text.scrub_url_credentials` — secret redaction.
- `utils.paths.*` — symlink-hardened, atomic private-file I/O (directly reusable for the scanner's own credential/config storage, per N.7).
- `probe.probe_command` — side-effect-free external-tool health probing (reusable for the scanner's own `yt-dlp`/`ffmpeg` checks).
- `Config` — reusable only if the scanner deliberately chooses to share AR's settings file (not recommended as the primary store; see N.7).

**What can be built immediately (no unresolved owner decision blocking it):**
- RSS adapter (direct `feedparser` use).
- Jina/web adapter (direct `WebChannel.read` reuse).
- URL/SSRF and redaction reuse throughout the ingestion boundary.
- The `untrusted_external_content` boundary type and Wave 1–2 normalization/dedup skeleton (no LLM, no market data, no calendar needed to build the *shape* of this).
- The canonical `ReportState` object shape and Markdown/JSON renderers (can be built and unit-tested against synthetic data before any real source is activated).

**What requires scanner adapters:** YouTube metadata/captions (`yt_dlp` direct), Exa discovery (`mcporter` fixed-template), market observations (new provider, not yet in AR's dependency set).

**What requires new scanner modules (the large majority — see Deliverable C table):** source registry, calendar intelligence, evidence lifecycle/extraction/validation/clustering, narrative synthesis, regime/directional assessment, confidence validation, canonical-state gate, SQLite persistence, scheduling, health monitoring, publication integrity.

**What requires Agent Reach extension:** **None identified.** Every gap in Deliverable C is addressable by scanner-owned composition or a scanner-owned adapter; no case was found where composition was genuinely insufficient (the bar set by Section 3.4).

**What requires source verification:** the entire fixed-source universe (Sections 8–10) — zero sources are currently `VERIFIED_WORKING` because none were activated this session, per explicit restriction.

**What requires terms review:** any source whose automation permissibility is unconfirmed (Reuters route, any BLS alternate route if the primary is restricted, any market-data provider's terms of use for automated personal-research use).

**What requires credentials:** Groq/OpenAI (ASR fallback, optional for the narrowest slice), the LLM provider chosen per N.5 (required once extraction/synthesis is built), a market-data provider (required once market observations are built).

**What requires owner approval:** all 24 items in Deliverable N, none of which this session decided unilaterally.

**What should be deferred:** Deliverables F–M in full (per session scope), shadow-mode tooling implementation (though its data-capture requirements should be kept in mind — every evidence/report object should already carry the fields Section 29 will need to evaluate later: source used, extraction confidence, contradiction presence, confidence label, expiration), human-feedback tooling, event-triggered special reports (Section 31 explicitly defers this).

**Primary technical risks:**
1. `transcribe.transcribe()`'s internal `yt-dlp` download timeout (1800s) is individually larger than the entire 25-minute Deep-run budget — a single slow transcript could dominate a run unless the scanner imposes its own tighter per-item timeout and treats transcript retrieval as expected-to-fail (which the master prompt already directs, Section 10.C).
2. Agent Reach's RSS/YouTube "channels" provide **no actual content-fetching code** to lean on — the scanner is building genuinely new ingestion logic against `feedparser`/`yt_dlp` directly, not thin wrappers around tested AR behavior. This is more implementation work than a superficial reading of `CLAUDE.md` would suggest, and testing burden shifts fully onto the scanner's own test suite.
3. No DNS-rebinding protection in `utils.url.normalize_public_http_url` — low risk for this use case (fetching known institutional URLs, not arbitrary user input) but worth a defense-in-depth note in the scanner's own security policy.

**Primary source-reliability risks:** BLS network-access uncertainty (N.8), Reuters/Treasury dead-endpoint risk (N.9/N.10), transcript-availability unpredictability (inherent to the master prompt's own design, Section 10.C — "model transcript retrieval as expected to fail occasionally").

**Primary security risks:** none identified in Agent Reach itself that the scanner would inherit; the scanner's *new* surface (fixed `mcporter`/`yt_dlp` command construction) must independently maintain the argument-array, no-shell-string discipline Agent Reach itself consistently follows — this is a scanner implementation responsibility, not a gap in AR.

**Primary operational risks:** scheduling mechanism entirely undecided pending OS confirmation (N.3/N.4); consensus-provider gap could force early "Indeterminate" or "Conditional" outcomes for calendar-driven mornings until N.1 is resolved.

**Preliminary Deep-run latency assessment:** qualitative only, per session scope (no live load test performed). The two AR-provided primitives with hard per-call timeouts (`WebChannel.read`: 30s; `transcribe.transcribe`'s yt-dlp step: up to 1800s) bound worst-case single-item latency; the scanner's own concurrency/parallelism design (not yet built) will determine whether 25 minutes is achievable at the candidate workload sizes in Section 26. **This is flagged as requiring an actual timed rehearsal (N.14) before the budget is trusted.**

**Verified-reachable versus desired source count:** 0 verified-reachable / a large desired institutional universe (Section 10.A's candidate list) — expected and correct for this session, since no source URLs were activated (explicit restriction). This number should change materially after a dedicated source-verification session.

**Readiness of the first vertical slice:** **Not yet started, but well-defined and low-risk to start.** All Agent Reach pieces the slice would need (`WebChannel.read`, `utils.url.*`, `utils.text.scrub_url_credentials`) are already verified reusable (Deliverable A). The slice does not require N.1/N.5/N.6/N.7's resolution if scoped as recommended below (no LLM, no consensus data).

**Recommended next architecture session:** a source-verification session focused narrowly on N.9/N.10/N.12/N.13 (Reuters, Treasury, BLS reachability, and picking the initial critical-source set) — this unblocks the first vertical slice concretely, is low-risk (read-only network tests against public endpoints), and directly informs Deliverable F/G's eventual scope without producing them prematurely.

**Recommended first implementation slice** (following the required pattern exactly):

1. **One verified source** — a single Tier-1 official RSS feed (e.g., a Federal Reserve press-release feed), verified reachable and current per Section 8's gate, chosen in the recommended next session (N.12), not this one.
2. **One safe retrieval path** — the scanner's own RSS adapter calling `feedparser.parse()` directly on that one feed URL, with `utils.url.normalize_public_http_url` validating the feed URL first.
3. **One normalized raw item** — the newest entry from that feed, timestamp-normalized to America/New_York, wrapped in the `untrusted_external_content` boundary type.
4. **One validated evidence object** — the entry's title/URL/publication-time/summary mapped into a minimal evidence record (Tier-1 authority, no LLM extraction needed for this slice — the "claim" can be the feed's own title/summary field, explicitly marked low-extraction-confidence since it wasn't LLM-verified against full text).
5. **One evidence cluster** — a single-item cluster in the appropriate category (e.g., "Monetary policy and rates" if a Fed feed is chosen).
6. **One canonical report state** — a minimal `ReportState` object containing just this one cluster, with all required metadata fields present (run ID, evidence cutoff, coverage explicitly marked `INSUFFICIENT` since it's a single-source demo, not a real report).
7. **One Markdown test output** — rendered from that state, including the mandatory `RESEARCH-ONLY BOUNDARY` closing block (Section 20).
8. **One matching JSON record** — rendered from the *same* state object, not synthesized independently, with status/confidence/expiration fields cross-checked to match the Markdown output exactly (Section 28's consistency requirement, demonstrated even at this minimal scale).

This slice proves: Agent Reach integration (via `WebChannel`/`utils` reuse, if the first source needs Jina full-text retrieval), trust-boundary separation, source provenance, timestamp normalization, evidence validation, a storage-direction decision (even if just "write to `data/` as files for now, SQLite deferred to slice 2"), canonical-state generation, Markdown/JSON consistency, and visible failure handling (the feed being unreachable should produce a visible, honest failure — not a silently empty report) — exactly the proof points Section 38 requires, without building the entire scanner in one implementation phase.

---

**End of Session 1 deliverable package (A–E, N, O).** Per Section 39, this session stops here and awaits the next instruction — no scanner code, no source activation, and no owner-sensitive decisions were made.
