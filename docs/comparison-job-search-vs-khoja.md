# Comparison: agent-data/job-search vs JustInncash/job-khoja

**Scope:** both GitHub projects ship the same `job-search` skill (7 skills, qualitative judge + digest), but as two different systems overall. This document compares every dimension the two repos differ on.

*Upstream:* `https://github.com/agent-data/job-search` (org `agent-data`, maintainer boganeyj)
*Vendor:* `https://github.com/JustInncash/job-khoja` (single-maintainer, `@JustInncash`)

---

## Bottom line

| | agent-data/job-search | JustInncash/job-khoja |
|---|---|---|
| What it is | upstream open-source **skill** | **plugin/package** that wraps the same skill + a local engine |
| Billing | metered agent-data API (100/mo free tier) | **free, unlimited, local** (no API key, no quota, no billing) |
| Sources | LinkedIn, Ashby, Greenhouse, Lever (4) | same + optional api-anything catalog sites (remoteok, weworkremotely, remotive, hacker-news; Indian portals learnable but not in-catalog) |
| License | MIT | MIT |
| Stars / forks | 58 / 4 | 1 / 0 |
| Latest commit | Aug 13, 2026 | Nov 5, 2026 (this week) |

They are **not alternatives** — job-khoja **is** job-search, delivered in a free, local-first package. The comparison below is "what runs around the skill."

---

## 1. Identity & ownership

**agent-data/job-search**
- GitHub org `agent-data`; single maintainer `boganeyj`.
- 727 commits on `main` (last commit Aug 13, 2026 — 3 months old).
- Distributed as a skill you install **per coding agent** (plugin marketplace, `codex plugin add`, `copilot plugin install`, `gemini extensions install`, opencode plugin, symlink, Pi).
- README title: "Job Search" — turns any coding agent into a job-search assistant.

**JustInncash/job-khoja**
- Single repo, single maintainer (`@JustInncash`).
- 10 commits on `master` (last commit 15 minutes ago — active, being patched today).
- Distributed as the **package** `job-khoja` (plugin marketplace `job-khoja@JustInncash`). The repo itself is the product.
- Recent commits: removed all "metered/upstream" framing from docs/CI, removed personal names from CONTRIBUTORS.md, added "privacy and verification standing rules" to skills.

---

## 2. What ships (file structure)

Both repos contain the **same 7 skills** and the same documents: `AGENTS.md`, `ARCHITECTURE.md`, `CHANGELOG.md`, `CLAUDE.md`, `CONTRIBUTING.md`, `COPILOT.md`, `GEMINI.md`, `LICENSE`, `README.md`, `TESTING.md`, `TODOS.md`, `pytest.ini`, `examples/`.

Differences:
- **skill repo** also has: `.claude/agents/`, `INSTALL_FOR_HERMES.md`, `.claude-plugin/`, `.codex-plugin/`, `.cursor-plugin/`, `.factory-plugin/`, `.opencode/`, `gemini-extension.json`, `package.json`, `plugin.yaml`, plus per-harness plugin manifests. Documentation is `docs/`.
- **khoja** additionally ships: `bin/` (launcher `agent-data`), `engine/` (local engine + `sources.json` + `manifest.json`), `docs/`, and a **cleaner** set of harness manifests. Documentation is `docs/`; the engine is a self-contained `engine/agent-data` CLI (`1.0.0-local`).

The khoja repo **adds**, not replaces, the file set: a local engine binary + extra CLI subcommands + richer export tooling.

---

## 3. Engine & billing (the real delta)

**agent-data/job-search**
- Runs against the **agent-data CLI**, a metered, third-party service (agent-data.dev).
- Needs an API key (from agent-data). Free tier: **100 calls/month**. Typical run ~10–30 calls.
- No local engine; the skill shells out to the metered agent-data service.
- Source-of-truth contract lives in `skills/agent-data-reference/SKILL.md`; runner executes against that CLI.

**JustInncash/job-khoja**
- Ships its **own local engine** `engine/agent-data` (version `1.0.0-local`). No metered API, no API key, no quota, no external billing.
- Pinned contract: `agent-data whoami | docs | call <listing> search-jobs | call <listing> get-posting` — stable envelopes, stable fields, stable error shape (`retryable`, `request_id`, `param`).
- `engine/sources.json` = the company-board registry (LinkedIn guest + Ashby/Greenhouse/Lever).
- Runs **completely offline/billing-free**; the whole skill set executes on your machine.
- **Extra engine commands beyond the base CLI** (only in khoja): `export-matches` (CSV/XLSX, 20 columns), `probe-registry`, `discover-slugs`.
- Enrichment applied at export: **salary normalization** (₹/USD bands), **company snapshot** (board slug + URL + open-role count), **repost/duplicate detection** (fuzzy title+company grouping → `is_repost` / `repost_of`).

---

## 4. Sources & coverage

| | agent-data/job-search | JustInncash/job-khoja |
|---|---|---|
| Supported sources | LinkedIn, Ashby, Greenhouse, Lever (4) | same 4 **plus** optional api-anything catalog sites |
| LinkedIn access | via agent-data API | guest endpoint (guest search), throttled politely, 6h cache |
| Company boards | via agent-data API | registry in `engine/sources.json` (Ashby/Greenhouse/Lever) |
| Optional catalog sites | — | `remoteok`, `weworkremotely`, `remotive`, `aa/<site>/<op>` (e.g. `aa/linkedin/search`); requires `api-anything` (npm) for the `aa/` path |
| Indian portals | not in-scope list | **Naukri, Indeed, Shine, TimesJobs are learnable but not yet in catalog** — run `api-anything learn <site>` once (browser, on demand); they then become callable as `aa/<site>/<op>` with no further changes |
| Coverage breadth | a function of the agent-data service's own coverage | a function of `engine/sources.json` + LinkedIn page depth (`MAX_LINKEDIN_PAGES`) |

Note the khoja source set is **larger but more explicit and local**: you can see exactly which boards are registered and can add boards by editing `engine/manifest.json` + `probe-registry`.

---

## 5. Judgment core (identical)

Both repos use the **same 5 skills** and the same qualitative judgment:
- `job-search` (home/interactive onboarding)
- `job-search-run` (headless pass)
- `job-search-runbook` (runtime contracts)
- `job-preference-interview` (brief capture)
- `evaluate-job-fit` (judge against prose preferences)

**Output:** HTML-style prose in chat, qualitative bands **only** (strong/moderate/weak), with reasoning citing posting → brief. **No numeric fit scores, no budget/cost config knobs** — enforced by `scripts/philosophy_guard.py` CI (fails the build if numeric scores or budget fields leak into shipped artifacts).

The two repos **do not differ in how a posting is judged** — that is the same skill.

---

## 6. Digest & output artifact

Both produce `reports/<date>-digest.md` with identical format: grouped by match strength, formatted run-health, counts (new / strong / moderate / weak / filtered), usage-first records. Example:

```
Job search digest — 2026-06-05
9 new postings · 2 strong · 2 moderate · 2 weak · 3 filtered out
```

khoja **adds** `agent-data export-matches` for CSV/XLSX export with the enriched fields above.

---

## 7. Scheduling & automation

Both share the same contract:
- `cron` or `launchd`, with a **session-loop fallback** (in-session retry while a session is open).
- Reccuring schedule recorded **only after** the agent runs the real scheduled command once and confirms it reached agent-data and wrote the workspace.
- If a run fails, ask "why did my last run fail?" or "how does my job search agent work?" — the agent reads the run record + digest (already on your machine) and tells you what stopped it and what fixes it.
- **khoja's honest constraint**: the engine's breadth is bounded by `engine/sources.json` + LinkedIn page depth; if LinkedIn tightens bot-blocking, the engine returns a clean error and the run closes normally (never fakes data).

---

## 8. Keys, cost & metering

**agent-data (upstream)**
- Needs an agent-data API key.
- Free tier 100/mo; run spends ~10–30 calls (metered).
- A run that fails auth/closing writes `runs/<run_id>.json` with `close_state: blocked` + `run_health: degraded` + a digest naming the missing key + exact command to set one.

**khoja**
- **No API key, no quota, no billing.** `agent-data whoami` → `api_key_set: true` from a local binary.
- The "100-call free tier" is entirely out of scope; no metering exists.
- Same `blocked`/digest-error-surfacing for a missing engine, but there is **no credit/budget awareness at all**.

---

## 9. Reliability, CI & tests

**agent-data**
- `pytest.ini` + `tests/` (fake-shim test suite, no live API calls — credit-free, side-effect-free).
- No CI.yaml in the README listing.

**khoja**
- CI (`.github/workflows/ci.yml`) runs: `pytest` (fake-shim, no live calls), a **philosophy guard** (`scripts/philosophy_guard.py` — fails CI on numeric fit-scores, budget/credit config fields, or unverified "actual charge" claims), a **doc_lint** (doc-drift check), and **release-integrity** (plugin manifests + harness metadata coverage).
- The khoja repo puts **privacy/verification standing rules** into the skills (latest commit).
- `TESTING.md` + `CONTRIBUTING.md` both canonicalize the "local-only, no metered account" stance.

**Verdict on tooling:** khoja is strictly ahead — it adds CI + philosophy guard + richer engine commands; the upstream repo is the simplest possible layout (skill + docs, no CI).

---

## 10. Workspace & privacy

Both place the workspace at `~/.job-search/`, which is **outside the repo** and **deny-mapped** (a `.gitignore` with `*` + `!.gitignore`, i.e. deny-all at the source). Nothing personal (brief, judged postings, run logs, digests) is committed or uploaded anywhere.

khoja additionally:
- hardens `CONTRIBUTORS.md` (removed personal names, added verification standing rules);
- removed all meta references to the "metered original" / "upstream";
- leaves the repository **permanently free of personal data** (synthetic examples only).

---

## 11. Harness support

Both target the same 9 agents: Claude Code, Codex, Cursor, opencode, Gemini CLI, GitHub Copilot CLI, Factory Droid, Pi, Hermes Agent. Both ship manifest files for the major harnesses. **khoja also ships a `bin/agent-data` launcher** that is idempotent (`mkdir -p ~/bin`, copy, chmod, PATH) and a Hermes-specific `INSTALL_FOR_HERMES.md`.

---

## 12. Versioning & community

| | agent-data | khoja |
|---|---|---|
| Version | per skill / repo (see `plugin.yaml`) | engine `1.0.0-local`, plugin `0.9.0` |
| Stars / forks | 58 / 4 | 1 / 0 |
| Last commit | Aug 13, 2026 | Nov 5, 2026 (active) |
| Community | upstream org, more eyes | single-maintainer, fast-moving, actively patched |

---

## 13. What each is best at

**agent-data/job-search**
- Best if you want the **upstream, widest-reaching** skill with a large community and many forks, hosted under the `agent-data` org, with a **well-known metered API** and established onboarding.
- No local engine cost/benefit decision to make.

**JustInncash/job-khoja**
- Best if you want **free, unlimited, local, no-key, no-billing** job search with **richer output** (export-matches JSON/CSV/XLSX, salary normalization ₹/USD, company snapshot, repost detection), an **explicit board registry** you can add to, and an **active CI/guard** that enforces privacy and stops metered/upstream framing from leaking into docs.
- Strongly recommended when you want to keep your search strictly on your machine (career-sensitive PII).

---

## 14. Gaps vs. work wanted

For the user's current workflow (fresher QA, Kolkata/India, Selenium/Java/Postman/SQL/Jira, remote ok):

- **Indian portals (Naukri, Indeed, Shine, TimesJobs)** are not yet in the catalog of either repo; they're learnable via `api-anything learn <site>` (browser, on demand) but not pre-loaded. Downstream this is identical — the skill itself is harness-agnostic and covers both.
- **Local-first** (khoja) wins for privacy; **metered API** (upstream) is the only reason a run exceeds the free tier and there is no local engine.
- The upstream repo is the **canonical reference implementation**; the khoja engine is a **drop-in local replacement** that runs the identical skill set.

---

## 15. Recommendation for shipping

1. Keep `job-search` (upstream) as the **reference** — it is the canonical, community-backed skill and the document of record for the 7-skill layout, the run contract, and the digest format.
2. Ship `job-khoja` as the **free, unlimited, local-first package** your own plugin builds on; it is the version that actually runs for you with **no metered account, no billing, and richer export**.
3. If you want to align khoja's source set with your own pipeline, add the **Indian portals** (Naukri/Indeed/Shine/TimeJobs) through `api-anything learn <site>` and register them in `engine/sources.json` — but that is an **engine extension**, not a skill change.
4. Both repos are MIT and both judge identically — no conflict in using them together for comparison while shipping khoja locally.

---

*Generated from live repository reads (agent-data/job-search, JustInncash/job-khoja) and the local job-khoja checkout. Nothing in the repo that ships is personal data; all shipped examples are synthetic.*
