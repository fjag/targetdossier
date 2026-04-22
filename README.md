# TargetDossier

![Built with Claude Code](https://img.shields.io/badge/built_with-Claude_Code-blueviolet?logo=anthropic&logoColor=white)
![MCP](https://img.shields.io/badge/MCP-Model_Context_Protocol-green)
![License](https://img.shields.io/badge/license-MIT-green)
![Agentic](https://img.shields.io/badge/workflow-agentic-orange)
![Powered by Paperclip](https://img.shields.io/badge/powered_by-paperclip.gxl.ai-blue)

A Claude Code slash command that builds a structured literature evidence profile
for a therapy target. Given a gene symbol and optional disease, it queries the
Paperclip biomedical literature corpus (8M+ full-text papers), extracts verified
evidence records across eight evidence classes, and produces a machine-readable
JSON profile plus a human-readable markdown report.

---

## Prerequisites

- [Claude Code](https://claude.ai/code) (CLI, desktop app, or IDE extension)
- A Paperclip account — sign up at [paperclip.gxl.ai](https://paperclip.gxl.ai)

---

## Setup

**1. Clone this repository into your project:**

```bash
git clone https://github.com/fjag/targetdossier.git
cd targetdossier
```

This gives you the slash command (`.claude/commands/targetdossier.md`) and the
extraction rules (`.claude/agents/paper-extractor.md`) that Claude Code needs
to run `/targetdossier`. Open the folder in Claude Code before continuing.

**2. Add the Paperclip MCP server (one-time):**

```bash
claude mcp add --transport http paperclip https://paperclip.gxl.ai/mcp
```

**2. Authenticate:**

Start Claude, enter `/mcp`, and select **Authenticate** under the paperclip server.

**3. Verify it is connected:**

```bash
claude mcp list
# paperclip: https://paperclip.gxl.ai/mcp (HTTP) - connected
```

For more details see [paperclip.gxl.ai](https://paperclip.gxl.ai).

---

## Usage

```
/targetdossier --target SYMBOL [--disease LABEL] [--top-k N] [--from YEAR] [--to YEAR] [--aliases A,B,C]
```

| Argument | Required | Default | Description |
|---|---|---|---|
| `--target` | yes | — | HGNC gene symbol, e.g. `GNB1` |
| `--disease` | no | — | Disease label, e.g. `epilepsy` |
| `--top-k` | no | 5 | Max papers per evidence class |
| `--from` / `--to` | no | 5-year window | Publication year range |
| `--aliases` | no | — | Comma-separated extra search terms |

**Examples:**

```
/targetdossier --target GNB1 --disease epilepsy

/targetdossier --target PCSK9 --disease hypercholesterolemia --top-k 10

/targetdossier --target LRRK2 --disease "Parkinson's disease" --from 2020 --to 2025 --aliases "leucine-rich repeat kinase 2"
```

The tool will ask you to confirm search parameters before any queries run. You
can add aliases or cancel at that point.

---

## Output

Two files are written to the current working directory after each run:

- `dossier_{target}_{disease}.md` — human-readable report, also printed to stdout
- `dossier_{target}_{disease}.json` — full structured profile with all verified records

The report covers eight evidence classes:

| Class | What it captures |
|---|---|
| `human_genetics` | GWAS, rare variant, eQTL, Mendelian randomisation |
| `functional_perturbation` | CRISPR, knockout, knockdown, overexpression |
| `expression_localisation` | RNA/protein expression, single-cell, tissue specificity |
| `pharmacology_tools` | Inhibitors, agonists, probes, selectivity data |
| `clinical_evidence` | Clinical trials, patient cohorts, biomarkers |
| `safety_essentiality` | Adverse effects, essentiality screens, PheWAS |
| `structure_druggability` | Crystal structures, cryo-EM, binding/allosteric sites |
| `mechanism_pathway` | Pathway, signalling, protein interactions |

Each record includes: canonical claim, direction of effect, supporting verbatim
quote (verified against fetched paper text), species, study design, and quality
flags.

---

## Estimated cost per run

**Paperclip MCP** is currently free.

**Claude tokens** (via your Claude Code plan or API key):

| Setting | Papers processed | Approx. input tokens | Approx. cost (API) |
|---|---|---|---|
| `--top-k 3` (light) | up to 24 | ~75K | ~$0.23 |
| `--top-k 5` (default) | up to 40 | ~125K | ~$0.38 |
| `--top-k 10` | up to 80 | ~250K | ~$0.75 |
| `--top-k 20` (heavy) | up to 160 | ~500K | ~$1.50 |

Token estimates assume ~3,000 tokens of fetched text per paper (meta.json +
grep output + 2–3 section excerpts). Actual cost varies with paper length and
corpus density. Output tokens (~5–10K per run) add roughly $0.10–0.15.

**Claude Code Pro/Max subscribers:** runs are covered by the subscription;
no per-run charge.

**API pricing reference:** Claude Sonnet 4.6 — $3/MTok input, $15/MTok output
(verify current pricing at [anthropic.com/pricing](https://www.anthropic.com/pricing)).

---

## Known limitations

- **Date filtering is approximate.** The `--from`/`--to` flags filter by year
  string matching, not strict cutoffs. Papers slightly outside the window may
  appear; papers at the boundary may be missed.
- **Paperclip corpus coverage.** 8M+ papers from bioRxiv, medRxiv, and PMC.
  Absence of a result does not mean absence of evidence — database-native
  sources (Open Targets, gnomAD, ChEMBL, PDB) are represented only via papers
  that cite them.
- **Output files land in the working directory.** Move them after the run if
  needed; there is no `--output-dir` flag.
- **Positive-result bias.** Published literature over-represents positive
  findings. Track `direction_of_effect: none` and `failed_to_replicate`
  records explicitly rather than discarding them.

See `CLAUDE.md` for full architectural constraints and `docs/ideas/targetdossier-spec.md`
for the complete specification.
