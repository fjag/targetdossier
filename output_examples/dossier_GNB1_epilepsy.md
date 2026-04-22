# TargetDossier Report — GNB1 / epilepsy

Generated: 2026-04-21T00:00:00Z
Papers retrieved: 5 | Processed: 2 | Records verified: 2 | Dropped: 3

## PARAMETERS
  Target: GNB1  Aliases: none
  Disease: epilepsy
  Year range: 2021–2026 (default 5-year window)  |  Top-k: 2
  Retrieval: SQL primary + semantic fallback

---

## EVIDENCE BY CLASS

### functional_perturbation  (1 paper, 1 record)
  - Sensory regulation of absence seizures in a mouse model of Gnb1 encephalopathy
    (2022, mouse — thalamocortical circuits) — In Gnb1 K78R/+ mutant mice, sensory
    stimulation significantly increases spike-wave discharge (SWD) frequency and
    duration; chemogenetic activation of thalamocortical cells further enhances SWD
    count (P<0.05, N=6), while reticular thalamic cells show opposite ictal activity
    patterns, establishing a thalamocortical circuit mechanism for absence seizure
    generation in GNB1 encephalopathy.  [positive]
    "Furthermore, chemogenetic activation of TC cells leads to the enhancement of SWD in Gnb1 mice."
    doi:10.1101/2022.04.28.489780

### clinical_evidence  (1 paper, 1 record)
  - GNB1 Encephalopathy: Clinical Case Report and Literature Review
    (2024, human — brain/neuronal Gβγ–GIRK signalling) — GNB1 encephalopathy, caused
    by pathogenic GNB1 variants, manifests with developmental delay, dystonia, and
    epilepsy in a significant fraction of ~68 documented worldwide cases. This case
    (first in Lithuania) presents an inherited Asp76Asn variant (autosomal dominant,
    Asp76–Ile80 hotspot) with mild phenotype and no epileptiform EEG, highlighting
    variable epileptic expressivity. Treatment remains symptomatic; ethosuximide and
    DBS are highlighted as candidates.  [positive]
    "GNB1 encephalopathy is a rare genetic disease caused by pathogenic variants in the G Protein Subunit Beta 1 (GNB1) gene, with only around 68 cases documented worldwide."
    doi:10.3390/medicina60040589  PMID:38674235

---

## TRIANGULATION
  Classes with support:  functional_perturbation, clinical_evidence
  Classes absent:        human_genetics, expression_localisation, pharmacology_tools,
                         safety_essentiality, structure_druggability, mechanism_pathway
  Converging direction:  positive (GNB1 pathogenic variants → epileptic/seizure activity
                         across both animal model and human clinical evidence)

## GAPS AND NEXT ACTIONS
- No human_genetics evidence retrieved — expand year window to include Petrovski et al.
  (2016) and Hemati et al. (2018) de novo mutation papers, or search ClinVar/OMIM directly.
- No expression_localisation evidence retrieved — check GTEx for GNB1 expression across
  epilepsy-relevant brain regions (thalamus, cortex).
- No pharmacology_tools evidence retrieved — ethosuximide (GIRK blocker) cited in both
  papers as a candidate treatment; search Paperclip for dedicated GNB1/GIRK pharmacology
  studies.
- No safety_essentiality evidence retrieved — check gnomAD for GNB1 constraint scores
  (pLI/LOEUF) to assess haploinsufficiency.
- No structure_druggability evidence retrieved — Gβ1 structural data exists; search for
  cryo-EM or crystallography studies of Gβ1–Gα or Gβ1–GIRK complexes.
- No mechanism_pathway evidence retrieved — re-run with --aliases 'Gbeta1,GIRK,Gβγ' to
  capture GIRK channel signalling papers that do not mention GNB1 by gene symbol.

## CAVEATS
  - Paperclip indexes papers only; absence here does not mean absence of evidence.
  - Publication bias likely inflates positive findings.
  - Evidence counts are not evidence strength; weight by independence and design.
  - 3 records dropped: 1 SKIP (semantic false-positive, no GNB1 in content),
    2 YEAR_FILTER (pub_date outside 2021–2026 default window).
  - The two year-filtered papers (Colombo et al. 2019, Pires et al. 2020) may contain
    relevant evidence; consider re-running with --from 2016 to include them.

---
Powered by Paperclip / gxl.ai
