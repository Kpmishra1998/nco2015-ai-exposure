# NCO-2015 3-digit AI Exposure Crosswalk

AI occupational exposure scores mapped to India's National Classification of
Occupations (NCO-2015) at the 3-digit level, for merging with PLFS microdata.

**File:** `nco2015_ai_exposure_3digit.csv` — 130 rows, one per NCO-2015
3-digit occupation group.

## Construction

Chain: US SOC (6-digit) → ISCO-08 minor group (3-digit) → NCO-2015 (3-digit).

1. **SOC → ISCO-08.** The official BLS crosswalk between SOC-2010 and ISCO-08
   (developed for the SOC Policy Committee, 2012), as distributed in the
   `eworx-org/iscoCrosswalks` R package, already aggregated to ISCO minor
   groups (1,021 SOC–ISCO pairs; 839 SOC-2010 codes; 130 ISCO minor groups).
2. **ISCO-08 → NCO-2015.** Identity at 3 digits: NCO-2015 was constructed by
   the DGE&T to align with ISCO-08; division groups correspond one-to-one to
   ISCO minor groups. No further mapping required.
3. **Aggregation within ISCO group.** Where several SOC codes map to one ISCO
   minor group, scores are averaged, both (a) weighted by BLS OEWS May 2021
   national employment (`*_w` columns) and (b) unweighted (`*_uw` columns,
   for robustness). OEWS is SOC-2018-coded; employment is carried to the
   SOC-2010 frame via O*NET's published 2010→2018 crosswalk, splitting
   equally where one 2018 code has multiple 2010 parents.

## Exposure measures

| Prefix | Source | Basis | Interpretation |
|---|---|---|---|
| `AIOE` | Felten, Raj & Seamans (2021), *SMJ* | SOC-2010, ability-based | General AI exposure (standardised; higher = more exposed) |
| `gpt_beta_human` | Eloundou et al. (2024), *Science* | O*NET-SOC 2019, task-based, human raters | Share of tasks where LLM + tooling halves time (0–1) |
| `gpt_beta_model` | Eloundou et al. (2024) | Same, GPT-4-rated | As above, model-annotated |

`z_*` columns are z-standardised across the 130 NCO groups (unweighted).
8-digit O*NET detail codes were averaged to 6-digit SOC before crosswalking.

## Diagnostics

- Spearman rank correlations between weighted measures: 0.90–0.93,
  consistent with cross-index correlations reported for the EU (Bruegel WP
  06/2024: 0.78–0.82 between AIOE and the ILO index).
- Face validity: most exposed groups are 264 (authors/journalists),
  212 (mathematicians/statisticians), 422 (client information workers);
  least exposed are 634, 931, 911 (subsistence fishers, construction
  labourers, domestic cleaners).
- Missing: AIOE absent for 011/021/031 (armed forces; typically excluded
  from PLFS analysis) and 951 (street services). Eloundou absent for
  011/021 only.

## Merging with PLFS

PLFS reports NCO-2015 at 3 digits in the occupation variable for usual and
current weekly status. Zero-pad to 3 characters and merge on `nco2015_3d`.
Codes appearing in PLFS but not here are India-specific additions within the
same minor-group structure or coding errors; in practice coverage of urban
graduate workers should exceed 99% of weighted employment — report the
match rate in your data appendix.

## Caveats to acknowledge in the paper

1. **Task-content transferability.** Scores describe US task bundles; the
   same NCO code may have a different task mix in India (Carbonero et al.
   2023). Mitigation: show robustness across all three indices, and to
   dropping IT occupations (groups 133, 251, 252, 351, 352) where the
   India–US task mismatch is most plausible.
2. **US employment weights.** The within-ISCO-group weighting uses US, not
   Indian, employment shares (Indian shares at SOC detail do not exist by
   definition). The `_uw` columns bound this choice.
3. **Exposure ≠ adoption.** These are potential-exposure measures in the
   standard tradition; interpret results as incidence under counterfactual
   adoption.
4. **Aggregation noise.** 3-digit assignment averages over heterogeneous
   detailed occupations; classical measurement error attenuates gradients,
   biasing against finding group differences.

## Replication

`build_nco_exposure.py` rebuilds the CSV from public sources:

- github.com/AIOE-Data/AIOE (`AIOE_DataAppendix.xlsx`)
- github.com/openai/GPTs-are-GPTs (`data/occ_level.csv`,
  `data/national_May2021_dl.csv`)
- github.com/eworx-org/iscoCrosswalks (BLS ISCO-08↔SOC-2010 crosswalk;
  O*NET 2010→2018 SOC crosswalk)

Dependencies: `pandas`, `numpy`, `pyreadr`, `openpyxl`. Place the three
repositories (cloned) and the AIOE workbook under `raw/` and run
`python3 build_nco_exposure.py`.

## To add next (not in this build)

- **ILO 2025 refined GenAI index** (Gmyrek et al., ILO WP 140): supplementary
  data at ilo.org provides ISCO-08 4-digit scores and exposure gradients —
  merge at 4 digits, aggregate to 3 as above. This is the index most
  defensible for developing-country application.
- **Webb (2020) AI patent exposure** for a pre-GenAI placebo.
