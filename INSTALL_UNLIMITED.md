# Job-Khoja — Installation

This is a standalone, free job-search project. It ships its own local engine, so there is no metered API, no monthly cap, and no API key.

| Feature | job-khoja |
|---|---|
| Billing / limits | **Unlimited — no billing, no monthly cap, no API key** |
| Engine | Local (`engine/agent-data`, `1.0.0-local`) |
| Sources | LinkedIn, Ashby, Greenhouse, Lever + optional api-anything catalog sites |

## Core design

- **Agent workflow** — setup → brief → search → judge → digest → schedule.
- **Workspace contract** — `config.yaml`, `preferences.md`, `jobs.jsonl`, `runs/`, `reports/`, registry.
- **CLI contract** — `agent-data whoami | docs | call <listing> search-jobs | call <listing> get-posting`, stable envelopes, stable fields, stable error shape (`retryable`, `request_id`, `param`).
- **Sources** — LinkedIn, Ashby, Greenhouse, Lever, plus the
  optional free remote boards `remoteok`, `weworkremotely` and `remotive`
  (best-effort) when added to `search.sources`. Any site in the **api-anything
  catalog** is available as a source `aa/<site>/<op>` (e.g.
  `aa/hacker-news/search`); catalog ops are learned from public pages. Indian
  portals Naukri, Indeed, Shine and TimesJobs are not yet in the catalog —
  learn them once with `api-anything learn <site>` (browser, on demand), and
  they become callable through the same `aa/<site>/<op>` source with no further
  changes.
- **Run/close semantics** — same scripts, same records (`complete`/`blocked`, `run_health`), same digest format.

## Engine

The `agent-data` CLI on PATH is backed by a local engine in
[`engine/`](engine/) that speaks a stable wire contract and:

- needs **no API key** (`whoami` → `api_key_set: true`),
- has **no quota** — no `403 insufficient_credits` can ever occur,
- fetches from the same sources directly: LinkedIn guest search + company
  ATS boards (registry in [`engine/sources.json`](engine/sources.json)),
- caches board dumps locally (6h) and throttles politely.

## Install

```sh
# put the launcher on PATH (idempotent)
mkdir -p ~/bin
cp bin/agent-data ~/bin/agent-data
chmod +x ~/bin/agent-data
# add ~/bin to PATH in ~/.bashrc if not present:
#   export PATH="$HOME/bin:$PATH"
agent-data whoami          # => api_key_set: true
```

The installed `job-search` plugin (or this project used as a plugin) then runs
unchanged against the unlimited engine.

## Extending coverage (ATS boards)

Company boards live in `engine/sources.json` (`{ashby: [...], greenhouse: [...], lever: [...]}`).
To add boards:

```sh
# edit engine/manifest.json, then:
python3 engine/agent-data probe-registry --manifest engine/manifest.json --out engine/sources.json
```

The probe verifies each board's public API and keeps only live ones.

## Extra commands (project enhancements)

The engine adds these subcommands beyond the base CLI surface:

- `agent-data export-matches --workspace <ws> --out <file.csv> [--run-id <id>] [--format csv|xlsx]`
  exports every judged posting with 20 columns: match band, reasoning, salary
  normalized, company snapshot (board slug, board URL, open-role count), and
  repost flag.
- `agent-data probe-registry --manifest m.json --out sources.json` — verify and
  merge new company boards into `engine/sources.json`.
- `agent-data discover-slugs [--source all|greenhouse|ashby|lever]` — harvest
  candidate board slugs from the public web archive, verify live, and merge.

Enrichment applied at export time: **salary normalization** (₹/USD bands),
**company snapshot** (board slug + URL + open-role count from the cached board),
**repost/duplicate detection** (fuzzy title+company grouping, `is_repost` /
`repost_of` columns).

These features live **only in this project** — the engine is fully local.

## Known constraints (honest)

- LinkedIn access uses the guest endpoint; if LinkedIn tightens bot-blocking,
  the engine returns a clean error and the run closes normally — the workflow
  never fakes data.
- The metered original indexes more LinkedIn results and more ATS boards than
  any free client can enumerate. Judgment quality and workflow are identical;
  breadth is a function of `engine/sources.json` + LinkedIn page depth
  (`MAX_LINKEDIN_PAGES`).
