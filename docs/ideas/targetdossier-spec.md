# TargetDossier — Specification

Turn a free-text therapy target and optional disease into a structured,
per-evidence-class evidence profile suitable for pipeline decision-making,
sourced entirely from the Paperclip MCP literature corpus.

---

## 1. Input

```json
{
  "target": {
    "symbol": "string (HGNC preferred, e.g. PCSK9)",
    "aliases": ["string"]
  },
  "disease": {
    "label": "string | null"
  },
  "time_window": { "from": "YYYY", "to": "YYYY" },
  "top_k": 5
}
```

User confirms before retrieval begins. Missing disease is acceptable.

---

## 2. Retrieval architecture

### Primary — SQL
```sql
SELECT id, doi, title, pub_date, authors, abstract_text
FROM documents
WHERE abstract_text ILIKE '%{target}%'
[AND abstract_text ILIKE '%{disease}%']
ORDER BY pub_date DESC
LIMIT {top_k * 8}
```

Add year filtering when `--from` / `--to` are supplied:
```sql
AND pub_date >= '{from}-01-01' AND pub_date <= '{to}-12-31'
```
`pub_date` is a proper `date` column — standard comparison operators work correctly.

Run for main symbol and each alias. Dedup by DOI across all results.

### Fallback — semantic search
```
search "{target} {disease}" -n {top_k}
search "{alias} {disease}" -n {top_k}
```

Applied after SQL to catch papers where the symbol appears in body text
only. Results merged and deduped into the master list.

### Why SQL-primary, not search-primary

SQL gives precise gene-symbol matching across abstracts, which semantic search
can miss for niche or rare targets. As an example, a validation probe on GNB1
(2026-04-21): `search "GNB1 CRISPR knockout"` returned zero GNB1-specific
papers — all three results were generic CRISPR methodology papers. By contrast,
`sql WHERE abstract_text ILIKE '%GNB1%'` returned 5 relevant papers
with DOIs populated. Semantic search does not respect gene symbol specificity
for niche targets, but remains useful as a fallback for aliases and body-text-only mentions.

### Total cap
`min(SQL + search results after dedup, top_k * 8)`.
Default top_k=5 → max 40 papers per run.

---

## 3. Per-paper extraction

### 3a. Fetch (coordinator only)
For each retained paper:
1. `cat /papers/{id}/meta.json`
2. `grep -i "{target}" /papers/{id}/content.lines`
3. `ls /papers/{id}/sections/`
4. Read Abstract + Results + Methods (or equivalent). Use `head -80` on
   sections >300 lines.

### 3b. Extract (coordinator, inline)
The coordinator applies the extraction rules from `.claude/agents/paper-extractor.md`
directly — no subagent is spawned. All reasoning is over the text fetched in 3a only.

### 3c. Quote verification (coordinator)
`supporting_quote` must appear verbatim in the text fetched in 3a.
Records failing verification are dropped and logged as `QUOTE_UNVERIFIABLE`.
Records with `supporting_quote: null` are logged as `NO_QUOTE`.
Neither type enters the aggregation.

---

## 4. Per-paper evidence record schema

```json
{
  "paper": {
    "doi": "string | null",
    "pmid": "string | null",
    "title": "string",
    "year": "integer",
    "venue": "string | null",
    "authors_first_last": "string"
  },
  "evidence_class": "human_genetics | functional_perturbation | expression_localisation | pharmacology_tools | clinical_evidence | safety_essentiality | structure_druggability | mechanism_pathway",
  "system": {
    "species": "human | mouse | rat | zebrafish | cell_line | organoid | computational | other",
    "tissue_or_cell": "string | null",
    "disease_context": "string | null"
  },
  "claim": {
    "text": "1-3 sentence paraphrase",
    "direction_of_effect": "target_up_disease_up | target_up_disease_down | target_down_disease_up | target_down_disease_down | none | unclear",
    "effect_size": "string | null",
    "uncertainty": "string | null",
    "sample_size": "integer | null"
  },
  "study_design": {
    "type": "observational | interventional | computational | review | meta_analysis",
    "independent_cohort": "boolean | null",
    "replication_status": "original | replicated | failed_to_replicate | unknown"
  },
  "quality_flags": {
    "preregistered": "boolean | null",
    "blinded": "boolean | null",
    "code_available": "boolean | null",
    "data_available": "boolean | null",
    "retraction_or_concern": "boolean | null"
  },
  "supporting_quote": "verbatim sentence from paper text"
}
```

---

## 5. Aggregation schema

```json
{
  "target": "string",
  "disease": "string | null",
  "generated_at": "ISO-8601",
  "total_papers_retrieved": 0,
  "total_papers_processed": 0,
  "total_records_verified": 0,
  "total_records_dropped": 0,
  "per_class": {
    "<class>": {
      "n_papers": 0,
      "support_count": 0,
      "contradict_count": 0,
      "direction_consistency": 0.0,
      "human_fraction": 0.0,
      "records": []
    }
  },
  "triangulation": {
    "classes_with_support": [],
    "classes_absent": [],
    "converging_direction": "string"
  },
  "gaps_and_next_actions": [],
  "caveats": [],
  "dropped_log": []
}
```

`direction_consistency` = fraction of directional records in the class
agreeing with the modal direction.

---

## 6. Evidence class definitions

See `.claude/agents/paper-extractor.md` for the authoritative class definitions
and extraction rules applied during each run.

---

## 7. Output

Two files written to the working directory:

- `dossier_{target}_{disease}.json` — full aggregation + all verified records
- `dossier_{target}_{disease}.md` — human-readable report (also printed to stdout)

