---
name: paper-extractor
description: Extraction instructions for the TargetDossier pipeline. NOT a spawnable subagent — the coordinator applies these rules inline after fetching paper text. The `.claude/agents/` location is retained so the rules are versioned with the project.
---

# Paper Extraction Instructions

These rules are applied by the coordinator inline — not by a subagent. After
fetching paper text via Paperclip (meta.json, grep, section files), the
coordinator applies the schema and rules below directly, then verifies the
supporting_quote before accepting the record.

## Grounding rules

1. **Fetched text only.** Claims must be grounded in the paper's meta.json,
   grep output, and section files retrieved in Step 3a. Do not use training-data
   knowledge to fill gaps.
2. **supporting_quote must be verbatim.** Copy character-for-character from
   the fetched text. Do not paraphrase, do not combine sentences.
3. **If no verbatim supporting sentence can be found, set supporting_quote to
   null.** A null record is correct. The coordinator will drop it. A fabricated
   quote is a critical failure — the coordinator verifies every quote.

## Input available during extraction

- `meta.json` output — doi, pmid, title, pub_date, authors, source
- `grep -i "{target}"` output — line-numbered lines mentioning the target
- Section file contents — Abstract, Results, Methods (or equivalent)

## What to return

Return ONE evidence record in this exact format:

```
EVIDENCE_RECORD
  paper:
    doi: <from metadata, or null>
    pmid: <from metadata, or null>
    title: <from metadata>
    year: <from metadata>
    venue: <journal or source from metadata, or null>
    authors_first_last: <"First Author Surname ... Last Author Surname">

  evidence_class: <one of the 8 classes below>

  system:
    species: <human | mouse | rat | zebrafish | cell_line | organoid | computational | other>
    tissue_or_cell: <string or null>
    disease_context: <string or null>

  claim:
    text: <1–3 sentence paraphrase of the specific finding about the target>
    direction_of_effect: <target_up_disease_up | target_up_disease_down | target_down_disease_up | target_down_disease_down | none | unclear>
    effect_size: <string or null>
    uncertainty: <string or null>
    sample_size: <integer or null>

  study_design:
    type: <observational | interventional | computational | review | meta_analysis>
    independent_cohort: <true | false | null>
    replication_status: <original | replicated | failed_to_replicate | unknown>

  quality_flags:
    preregistered: <true | false | null>
    blinded: <true | false | null>
    code_available: <true | false | null>
    data_available: <true | false | null>
    retraction_or_concern: <true | false | null>

  supporting_quote: "<VERBATIM sentence copied from GREP MATCHES or SECTIONS>"
```

## Evidence class definitions

Assign the single best-fitting class based on the paper's primary contribution:

- `human_genetics` — GWAS, rare variant, eQTL colocalization, Mendelian
  randomisation, loss-of-function burden in human populations
- `functional_perturbation` — CRISPR, knockout, knockdown, overexpression,
  RNAi, conditional knockout, rescue experiments in any model system
- `expression_localisation` — RNA or protein expression levels, single-cell
  profiling, tissue specificity, spatial distribution
- `pharmacology_tools` — small molecule inhibitors, agonists, antagonists,
  chemical probes, selectivity characterisation
- `clinical_evidence` — clinical trials, patient cohort studies, biomarker
  studies, patient stratification
- `safety_essentiality` — adverse effects, essentiality screens, PheWAS,
  on-target toxicity, knockout lethality phenotypes
- `structure_druggability` — crystal structure, cryo-EM, binding site
  characterisation, allosteric site, structural modelling
- `mechanism_pathway` — pathway analysis, protein-protein interactions,
  signalling context, mechanistic studies

When a paper spans multiple classes, pick the one most central to the
finding about the target.

## direction_of_effect rules

Assign a directional value ONLY when the Results or conclusion text explicitly
states a direction with respect to both the target and the disease/phenotype:

- Target activity/expression ↑ AND disease/phenotype worsens → `target_up_disease_up`
- Target activity/expression ↑ AND disease/phenotype improves → `target_up_disease_down`
- Target activity/expression ↓ AND disease/phenotype worsens → `target_down_disease_up`
- Target activity/expression ↓ AND disease/phenotype improves → `target_down_disease_down`
- Paper does not address disease context → `none`
- Direction stated but ambiguous or contradictory → `unclear`

Do NOT infer direction from the abstract if the Results section states
something different. Results section takes precedence.

## supporting_quote rules

- Copy exactly from GREP MATCHES or SECTIONS. Character-for-character.
- Pick the single most informative sentence about the target's role or effect.
- Do not combine two sentences. Do not rephrase.
- If the only mentions are in a reference list (e.g. "[Smith et al. showed X]"):
  set `supporting_quote: null`.
- If no sentence about the target can be found anywhere in the provided text:
  set `supporting_quote: null`.

## Drop conditions

If the paper should be dropped, return this instead of a full record:

```
EVIDENCE_RECORD
  supporting_quote: null
  drop_reason: "<one of: paper does not mention target in provided text | target appears only in references | paper is off-topic>"
```
