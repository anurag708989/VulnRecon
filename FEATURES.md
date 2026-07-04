# Features Reference

## 1. Smart Input Handling

Auto-detects whether input is a single domain, comma/newline-separated list, or a file path; normalizes protocol/path prefixes and applies a small typo-correction map (e.g. `goggle.com` → `google.com`).

**Current limitation:** the correction map is a hardcoded dictionary of a handful of entries — it is a nice-to-have, not a robust fuzzy-matching system. Good first-issue candidate: replace with a Levenshtein-distance suggestion against a known top-domains list.

## 2. Subdomain Discovery

Combines:
- `subfinder` (passive sources)
- Certificate Transparency lookups
- Common subdomain pattern generation
- New subdomains surfaced from Wayback/CDX history

Output: deduplicated subdomain list per target, saved under `subdomains/`.

## 3. Wayback / CDX Bulk URL Processing

Downloads historical URLs for `*.target.com/*` from the Wayback CDX API in a single bulk request (avoiding per-URL calls), then processes results **locally** with pattern matching instead of live crawling — this is what keeps notification/API volume low compared to naive approaches.

Pipeline stages:
1. Bulk download (`web.archive.org/cdx/search/cdx`)
2. Garbage filtering (images, CSS, oversized URLs)
3. Pattern/priority/confidence scoring (see below)
4. Categorization (Sensitive Files, Sensitive Paths, Authentication, Administrative, File Operations, API Endpoint, IDOR Potential)
5. FFUF/Nuclei target extraction

## 4. Priority & Confidence Scoring ("AI-Powered Filtering")

A rule-based scoring engine (`AdvancedPatternMatcher`) assigns:

- **Priority** (0–100) — how likely a finding is exploitable/interesting
- **Confidence** (0.0–1.0) — how sure the matcher is about the categorization
- **Score** — cumulative weight across matched rules (extension, path keyword, query pattern)

| Priority | Meaning | Examples |
|---|---|---|
| 90–100 | Critical | `.env`, `.sql`, admin panels, IDOR (`?id=123`) |
| 70–89 | High | API endpoints (`/api/`, `/rest/`), CVEs, config files |
| 50–69 | Medium | Standard web assets, login pages |
| 0–49 | Low | Static resources |

> **Honesty note:** this is a deterministic keyword/regex rule set, not a trained ML model. The "AI" branding in the original marketing material overstates what the code does — it's good, useful heuristic scoring, but calling it out accurately in this open-source version avoids misleading contributors and users.

## 5. Custom JS File Discovery

Filters JS URLs down to files that (a) are hosted on the target domain and (b) don't match a static third-party keyword list (`jquery`, `bootstrap`, `angular`, `react`, `vue`, `google`, `facebook`, `cdn`, …). Remaining files get a confidence score boosted by naming heuristics like `app`, `main`, `custom`.

**Gap:** the third-party list is static and will both over- and under-filter on real-world targets. A more robust approach: fetch a snapshot of the file and compare against known library signatures/hashes (e.g. via `retire.js` patterns) rather than filename keyword matching.

## 6. Shodan Integration

Queries `hostname:<target>` against the Shodan Host Search API and extracts:
- Services (IP, port, product, version, org, geo-location)
- CVEs attached to each service, with CVSS-derived severity buckets (Critical/High/Medium/Low)
- Aggregate vulnerability summary counts

**Gap:** single-page query (`limit=50`), no pagination, no retry/backoff on rate limiting (HTTP 429), no local caching — repeated scans of the same target burn API credits unnecessarily.

## 7. Nuclei & FFUF Target Generation

- `nuclei_targets.txt` — deduplicated, scored URLs suitable for `nuclei -l`
- `ffuf_targets.txt` — unique base paths with `/FUZZ` appended, deduplicated by subdomain (fixed in V3.2 to avoid the "431k duplicate FUZZ targets" bug from earlier versions)

**Clarification for users:** the tool does **not** invoke `nuclei` or `ffuf` itself — it produces input files. Run them explicitly:

```bash
nuclei -l vulnrecon_results/<target>/nuclei/nuclei_targets.txt -severity critical,high
ffuf -w vulnrecon_results/<target>/ffuf/ffuf_targets.txt:FUZZ -u FUZZ
```

## 8. Rich Terminal UI

Uses the `rich` library for:
- Startup banner and dependency-check table
- Live progress bars per phase (subdomain discovery, Wayback analysis, JS discovery, Shodan, DNS)
- Result tables (Sensitive URLs, Custom JS Files, New Subdomains, Shodan Services, CVEs, Open Ports)
- Final comprehensive summary table

Falls back to plain `print()` output when `rich` isn't installed (`RICH_AVAILABLE` flag), so the tool degrades gracefully rather than crashing.

## 9. Telegram Notifications

Optional real-time alerts for scan start, per-phase discovery counts, and completion summary. Message length is truncated to Telegram's 4096-character limit.

**Gap:** no retry logic if the Telegram API call fails beyond a single attempt, and errors are printed but not logged to file.

## 10. Timeout & Performance Controls (V3.2+)

Introduced after users reported the JS-discovery phase hanging indefinitely:
- Per-request HTTP timeout reduced 10s → 3s
- Overall JS discovery phase capped (~60s)
- Subdomain checks limited to first 20 for JS discovery
- Concurrency capped at 5 simultaneous subdomain checks

**Gap:** the `--threads` CLI flag (default 20) is not actually wired into an `asyncio.Semaphore` in the reviewed code path — it's accepted but doesn't bound concurrency the way a user would expect.

## 11. Configuration System

Supports both a `config.txt` (`key=value`) file and CLI arguments, with CLI arguments taking precedence. Values are naively type-coerced (bool/int/float/string).

**Gap:** no schema validation — a malformed `config.txt` line is silently ignored (`except: pass`) rather than raising a warning.

## 12. Output Artifacts

Every scan produces a timestamped, per-target directory containing JSON (`comprehensive_results_*.json`) and category-specific text/JSON files (see [README → Output Structure](../README.md#output-structure)).
