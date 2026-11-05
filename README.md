<div align="center">

# Job-Khoja

### Save hours of scrolling through job postings.

*Hyper-personalized job search that filters postings based on what matters to you: role scope, autonomy, management experience, fit with your background, and more.*

**job-khoja** is a free, unlimited job-search engine for coding agents — including a fully local, keyless engine. See [INSTALL_UNLIMITED.md](INSTALL_UNLIMITED.md).
</div>

<img width="3182" height="2160" alt="Job Search digest showing reviewed matches in an agent conversation" src="https://github.com/user-attachments/assets/a3c45a7e-6a93-4afa-86f0-f522c8f8d53c" />

## Challenges with traditional job search

- Important details are often missing from filter fields or buried in job descriptions.
- Reposts and duplicates flood search results with irrelevant and repetitive postings.
- Traditional search requires you to compress a nuanced decision into a few keywords. Results often miss what matters: the work itself, decision-making authority, management experience, and fit with your background.
- After applying filters and getting search results, the real work begins: reading posting after posting to find the few roles you would seriously consider.

## Why the Job Search plugin?

- **Search by fit, not just keywords:** describe your preferences in plain English, including concrete constraints and less searchable qualities such as autonomy, role scope, and access to company leadership.
- **Let your agent do the filtering:** your agent reviews every new posting against your preferences, filters out clear mismatches, and reads promising job descriptions in full. It evaluates the work behind the title, surfacing relevant roles at unfamiliar companies or under titles you might not have thought to search for.
- **Know why each job fits:** get a concise explanation of why each role is a strong, moderate, or weak match, so you can quickly focus on the opportunities that best fit your preferences, background, and interests.
- **Catch new matches early:** schedule your search to run daily, hourly, or on your preferred cadence, so new postings are automatically evaluated as they appear.

## Quickstart

1. [Install Job Search](#installation) for your coding agent.
2. Start a conversation with:

   > **Set up my job search. I'm looking for** a senior product design role, remote in the US, with end-to-end ownership. I don’t want brand-heavy or solo-designer roles. My background is primarily B2B SaaS...

The agent then:
- Checks its prerequisites
- Documents your preferences, and 
- Searches live postings. 

If useful roles are available, it shows only the ones matching your preferences.

A sentence or two is enough to begin. You can also share relevant material, such as a resume, cover letter, or notes from previous applications to use as it evaluates job opportunities.

## What a run looks like

```text
You: Run a search now.

Job Search: Searching for new roles matching your preferences...
Found 42 postings. 9 are new. Reading the promising ones in full...

Here are the first strong matches while I keep reviewing the rest:

• Senior Product Designer — Tidewater Health — Remote (US)
  Owns a care-navigation area end to end; the healthcare mission you're after.

...still reviewing the remaining postings...

Job search digest — 2026-06-05
9 new postings · 2 strong · 2 moderate · 2 weak · 3 filtered out
```

See the [complete sample digest](examples/sample-digest.md).

## What you can ask

| Goal | Example |
|---|---|
| Start | "Set up my job search. I want a senior product design role, remote in the US, with end-to-end ownership. I don’t want brand-heavy or solo-designer roles. My background is primarily B2B SaaS...” |
| Refine preferences | “Make working closely with product and engineering a strong preference” |
| Search now | “Run a search now,” or “check for new jobs.” |
| Schedule searches | “Run my search every morning.” |
| Review a match | “Why is this a strong match?” or “Does this posting fit what I want?” |
| Pause or stop | “Pause my schedule,” or “stop scheduling.” |

## Sources and data dependencies

Job-Khoja gets live postings through its own local engine (see [INSTALL_UNLIMITED.md](INSTALL_UNLIMITED.md)). Onboarding handles the setup:

1. If the engine is missing, the agent offers to install it (copy `bin/agent-data` onto your PATH).
2. No authentication or API key needed — the engine is fully local.

**Currently supported sources:** LinkedIn Jobs, Ashby, Greenhouse, Lever, plus optional api-anything catalog sites (`aa/<site>/<op>`).

## Installation

- **job-khoja (this repo):** follow [INSTALL_UNLIMITED.md](INSTALL_UNLIMITED.md) — installs the free, local engine with no metered API or billing.

Choose your coding agent below. After installation, use the [Quickstart](#quickstart) sentence. Installation succeeded when Job Search opens onboarding and begins the agent-data check.

### Claude Code

Run these commands inside Claude Code:

```text
/plugin marketplace add JustInncash/job-khoja
/plugin install job-khoja@JustInncash
```

Use `/job-search` if you need the command shortcut.

### Codex

Run in your shell, then launch or restart Codex:

```bash
codex plugin marketplace add JustInncash/job-khoja
codex plugin add job-khoja@JustInncash
codex
```

Use `$job-search` if you need the skill shortcut.

### Cursor

Clone the repository, then symlink it into Cursor's local plugins directory:

```bash
git clone https://github.com/JustInncash/job-khoja
ln -s "$(pwd)/job-khoja" ~/.cursor/plugins/local/job-khoja
```

### opencode

Add Job Search to the `plugin` array in your global OpenCode config at
`~/.config/opencode/opencode.json`, or to an `opencode.json` in a project where you want to use it:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": ["job-khoja@git+https://github.com/JustInncash/job-khoja.git"]
}
```

Restart OpenCode, then start with: **“Set up my job search.”** OpenCode installs the git package with
Bun and the plugin registers all seven bundled skills. If your config already has plugins, add this entry
to the existing array rather than replacing it.

See [`.opencode/INSTALL.md`](.opencode/INSTALL.md) for verification, revision pinning, local development,
and troubleshooting.

### Gemini CLI

```bash
gemini extensions install https://github.com/JustInncash/job-khoja
```

### GitHub Copilot CLI

```bash
copilot plugin marketplace add JustInncash/job-khoja
copilot plugin install job-khoja@JustInncash
```

### Factory Droid

```bash
droid plugin marketplace add https://github.com/JustInncash/job-khoja
droid plugin install job-khoja@JustInncash
```

### Pi

```bash
pi install git:github.com/JustInncash/job-khoja
```

For an editable local install, run `pi -e /path/to/job-khoja`.

### Hermes Agent

Copy and paste the following into a new Hermes Agent session:

```text
Retrieve and follow the installation instructions at:
https://raw.githubusercontent.com/JustInncash/job-khoja/main/INSTALL_FOR_HERMES.md
```

After installation, start a new Hermes Agent session and use the Quickstart sentence above.

## Support matrix

<details>
<summary>Compatibility details</summary>

These expectations apply to every listed agent:

| Field | Expected support |
|---|---|
| OS / architecture | macOS and Linux; arm64 and x86_64 |
| Recurring scheduler | `cron` or `launchd`, with a session loop fallback |
| Modes | interactive and background (headless) |

Job Search is compatible with Claude Code, Codex, Cursor, opencode, Gemini CLI, GitHub Copilot CLI, Factory Droid, Pi, and Hermes Agent.

Every run — the one you ask for and the one a schedule starts — reads and judges postings on the model of the session it runs in. Nothing pins a separate model for the search. A recurring schedule is recorded only after the agent runs the real scheduled command once and confirms that it reached agent-data and wrote the workspace.

</details>

If something fails, ask **“why did my last run fail?”** or **“how does my job search agent work?”** The agent reads the run's record and the digest it wrote, both already on your machine, and tells you what stopped the run and what fixes it. To report a bug, copy what it tells you into a [GitHub issue](https://github.com/JustInncash/job-khoja/issues).

## License

[MIT](LICENSE)
---

## Job-Khoja

A standalone, free job-search project for coding agents. It ships its own local engine (`engine/agent-data`, `1.0.0-local`) — no metered API, no external billing.

- **Sources:** LinkedIn (guest), Ashby, Greenhouse, Lever + optional api-anything catalog sites via `aa/<site>/<op>` (e.g. `aa/linkedin/search`). Requires `api-anything` (npm) for that path; set `API_ANYTHING_CLI` to override its location.
- **Install:** see `INSTALL_UNLIMITED.md`.
- **License:** MIT (see [LICENSE](LICENSE)).
