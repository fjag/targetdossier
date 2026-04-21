# TargetDossier

A Claude Code slash command that builds a structured, per-evidence-class
literature evidence profile for a therapy target using the Paperclip MCP corpus.

## Hard constraints

- **Paperclip-only.** All retrieval via `mcp__paperclip__paperclip`. No web
  search, no external databases. If Paperclip is unreachable, stop immediately.
- **Verbatim quotes required.** Every evidence record must include a
  `supporting_quote` copied character-for-character from fetched paper text.
  The coordinator verifies this before accepting any record. Records that fail
  verification are dropped and logged — never silently discarded.
- **Coordinator fetches and extracts.** All Paperclip calls and evidence
  extraction happen in the coordinator. The extraction rules in
  `paper-extractor.md` are applied inline — there is no separate subagent.
- **No silent pass-through.** Target normalisation must be confirmed by the
  user before retrieval begins.
- **JSON to file only.** Markdown report → stdout + `evidence_<target>.md`.
  Full JSON → `evidence_<target>.json` only, never stdout.

## Retrieval architecture

SQL is the primary retrieval mechanism (`abstract_text ILIKE '%TARGET%'`).
Semantic `search` is used as a fallback to catch aliases and body-text-only
mentions. Evidence class is assigned during extraction from paper content —
not inferred from which query retrieved the paper.

## Files

- `docs/ideas/targetdossier-spec.md` — full specification
- `.claude/commands/targetdossier.md` — coordinator slash command
- `.claude/agents/paper-extractor.md` — extraction rules applied inline by the coordinator

## MCP setup (one-time)

```
claude mcp add --transport http paperclip https://paperclip.gxl.ai/mcp
```

Verify with `claude mcp list` before the first run.

## Security notes

- **Input validation.** The coordinator validates `--target`, `--disease`,
  `--top-k`, and year arguments before any queries run. Inputs containing SQL
  metacharacters (quotes, semicolons) are rejected to prevent query breakage.
- **Prompt injection via paper content.** Fetched paper text (titles, abstracts,
  section content) flows into the coordinator's context. A crafted paper could
  attempt to inject instructions. Mitigated by the fact that the coordinator has
  no privileged actions beyond writing two output files; there are no external
  API calls, no credential access, and no shell execution paths.

## Known limitations

- **Date filtering is approximate.** The `pub_date` column stores values like
  `September_2025`, not ISO dates. SQL date comparisons fail silently.
  Use `pub_date ILIKE '%2025%'` for year filtering. The `--year` flag on
  `search` is a ranking hint, not a hard cutoff.
- **`--top-k` is a per-class target, not a total cap.** The coordinator pulls
  up to `top_k × 8` papers, distributes them across classes (capped at
  `top_k` per class), and processes at most `top_k × 8` total. A run with
  `--top-k 5` may process up to 40 papers if all 8 classes have coverage,
  or as few as 5 if the corpus is sparse.
- **Output files land in the working directory.** There is no `--output-dir`
  flag. Files are written to wherever the Claude Code session is running.
  Move them after the run if needed.
- **Paperclip indexes 8M+ papers but is not exhaustive.** Absence of a record
  in the output does not mean absence of evidence in the literature.
