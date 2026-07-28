# data-to-okf

An [Agent Skill](https://agentskills.io) that converts any local folder of
mixed documents — contracts, meeting transcripts, financial exports, a
database, screenshots — into a [Google Open Knowledge Format (OKF) v0.2](https://github.com/GoogleCloudPlatform/knowledge-catalog/blob/main/okf/SPEC.md)
knowledge bundle: a tree of thin markdown "concept" files an AI agent can
traverse deterministically instead of grepping raw binaries every time.

The bundle never copies source files. Every concept points back at the real
file via a `resource: file://...` URI, so the original folder stays the
single source of truth.

## What it does

- Walks a source folder and buckets files by their own top-level subfolder
  structure (no fixed taxonomy to configure).
- Extracts a short markdown summary per file type: `python-docx` for `.docx`,
  `pypdf` for `.pdf`, `openpyxl` for `.xlsx`, live `duckdb` CLI introspection
  for `.duckdb` files, stdlib `csv` for CSV folders.
- Skips secrets, `.DS_Store`, and video files by default.
- Generates OKF-conformant `index.md`/`log.md` files and validates the result.

See [`SKILL.md`](SKILL.md) for the full agent-facing instructions and
[`scripts/`](scripts/) for the two bundled Python scripts
(`generate_okf_bundle.py`, `validate_okf_bundle.py`).

## Installing

This repo follows the [Agent Skills specification](https://agentskills.io/specification),
so any [compatible client](https://agentskills.io/clients) — Claude Code,
Claude, Cursor, GitHub Copilot, OpenAI Codex, Gemini CLI, VS Code, OpenCode,
Roo Code, and dozens more — can load it directly once it's placed (or
symlinked) in the location that client scans for skills. The general
pattern is the same everywhere: **clone this repo into your client's skills
directory, keeping the folder name `data-to-okf`.**

```bash
git clone https://github.com/rajivmehtaflex/data-to-okf.git data-to-okf
```

Per-client install paths:

| Client | Where to put/link it |
|---|---|
| **Claude Code** (personal, all projects) | `~/.claude/skills/data-to-okf/` |
| **Claude Code** (this project only) | `<project-root>/.claude/skills/data-to-okf/` |
| **Claude.ai** | Upload as a skill in Settings → Capabilities, or use the [skill-creator](https://github.com/anthropics/skills) upload flow |
| **Cursor** | See [cursor.com/docs/context/skills](https://cursor.com/docs/context/skills) for its skills folder |
| **Gemini CLI** | See [geminicli.com/docs/cli/skills](https://geminicli.com/docs/cli/skills/) |
| **OpenAI Codex** | See Codex's skills docs — check the [client showcase](https://agentskills.io/clients) entry for the current path |
| **VS Code / GitHub Copilot / others** | Check the corresponding entry on the [Agent Skills client showcase](https://agentskills.io/clients) for that client's exact skills directory and setup steps |

A quick way to confirm it loaded: ask your agent to list its available
skills (in Claude Code, run `/skills`) and look for `data-to-okf`.

### One-time setup after installing

The bundled scripts need a few Python packages (and, optionally, the
`duckdb` CLI if you'll be bundling folders containing `.duckdb` files):

```bash
python3 -m venv .okf-venv && source .okf-venv/bin/activate
pip install python-docx openpyxl pyyaml pypdf
```

You (or your agent) can also just ask the agent to set this up — see the
sample prompts below.

## Sample prompts

Once installed, just talk to your coding agent normally — you don't need to
name the skill or run the scripts yourself. Try any of these:

> Bundle `~/Desktop/ClientY` into a knowledge bundle I can point an agent at.

> I have a folder of contracts, invoices, and a database export at
> `~/Projects/Acme/data` — can you turn it into an OKF knowledge base so an
> agent can query it without me re-explaining the folder every time?

> Turn `./docs-raw` into an agent-consumable knowledge bundle. Call the
> output folder `docs-okf`.

> We added a bunch of new PDFs to `~/Desktop/ClientY` this week — can you
> refresh the `ClientY-okf` bundle?

> Make this folder of meeting notes and spreadsheets AI-readable.

The agent will ask for (or infer) the source folder and a destination name,
install the small set of Python dependencies if needed, generate the
bundle, validate it, and report back what got included and what was
excluded (secrets, videos, etc.) — see [`SKILL.md`](SKILL.md) for the exact
workflow it follows.

## Validating conformance

```bash
pip install skills-ref
agentskills validate .
```

## License

Apache 2.0 — see [LICENSE](LICENSE).
