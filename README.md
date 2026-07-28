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

## Using this skill

This repo follows the [Agent Skills specification](https://agentskills.io/specification) —
any compatible agentic coding tool can load it directly. To install manually,
clone (or copy) this folder into whatever skills directory your agent client
scans (e.g. `~/.claude/skills/data-to-okf/` for Claude Code).

Quickstart (once installed):

```bash
python3 -m venv .okf-venv && source .okf-venv/bin/activate
pip install python-docx openpyxl pyyaml pypdf

python3 <skill-dir>/scripts/generate_okf_bundle.py --source "<source folder>" --dest "<bundle folder>"
python3 <skill-dir>/scripts/validate_okf_bundle.py "<bundle folder>"
```

## Validating conformance

```bash
pip install skills-ref
skills-ref validate .
```

## License

Apache 2.0 — see [LICENSE](LICENSE).
