# NCO-2015 3-digit AI Exposure Crosswalk

AI occupational exposure scores mapped to India's National Classification of
Occupations (NCO-2015) at the 3-digit level, for merging with PLFS microdata.

## Files

| File | Contents |
|---|---|
| `nco2015_ai_exposure_3digit.csv` | 130 rows, one per NCO-2015 3-digit group: AIOE, GPT exposure (human- and model-rated), and ILO GenAI index |
| `webb_exposure_nco2015_3digit.csv` | 126 rows: Webb (2020) patent-based AI, software, and robot exposure |
| `ilo_wp140_tableA1.tsv` | ILO WP 140 4-digit scores, transcribed from Appendix Table A1 |
| `build_nco_exposure.py` | Rebuilds the SOC-based columns from public sources |
| `add_ilo_index.py` | Aggregates the ILO 4-digit scores and appends them |

Merge key in both CSV files: `nco2015_3d`.

## Construction

Chain: US SOC (6-digit) → ISCO-08 minor group (3-digit) → NCO-2015 (3-digit).

1. **SOC → ISCO-08.** The official BLS crosswalk between SOC-2010 and ISCO-08
   (developed for the SOC Policy Committee, 2012), as distributed in the
   `eworx-org/iscoCrosswalks` R package, aggregated to ISCO minor groups
   (1,021 SOC–ISCO pairs; 839 SOC-2010 codes; 130 ISCO minor groups).
2. **ISCO-08 → NCO-2015.** Identity at 3 digits: NCO-2015 was constructed by
   the DGE&T to align with ISCO-08, and its 3-digit groups correspond one to
   one to ISCO minor groups.
3. **Aggregation within ISCO group.** Where several SOC codes map to one ISCO
   minor group, scores are averaged, (a) weighted by BLS OEWS May 2021
   national employment (`*_w` columns) and (b) unweighted (`*_uw` columns).
   OEWS is SOC-2018-coded; employment is carried to the SOC-2010 frame via
   O*NET's published 2010→2018 crosswalk, split equally where one 2018 code
   has multiple 2010 parents. 8-digit O*NET detail codes are averaged to
   6-digit SOC before crosswalking.

## Exposure measures

| Prefix | Source | Basis | Interpretation | Groups scored |
|---|---|---|---|---|
| `AIOE` | Felten, Raj & Seamans (2021), *SMJ* | SOC-2010, ability-based | General AI exposure | 126 |
| `gpt_beta_human` | Eloundou et al. (2024), *Science* | O*NET-SOC 2019, task-based, human raters | Share of tasks where an LLM with tooling halves completion time (0–1) | 128 |
| `gpt_beta_model` | Eloundou et al. (2024) | As above, GPT-4-rated | As above, model-annotated | 128 |
| `ilo_score` | Gmyrek et al. (2025), ILO WP 140 | ISCO-08 4-digit, task-based | Mean GenAI automation score (0–1) | 127 |
| `webb_ai`, `webb_software`, `webb_robot` | Webb (2020) | Overlap between patent text and O*NET task descriptions | Exposure to AI, software, and robots | 126 |

`z_*` columns are z-standardised (unweighted) across the NCO groups each
measure scores, so each has mean 0 and SD 1 at the occupation level.

**ILO index.** Scores are native to ISCO-08 at 4 digits (Table A1 of WP 140)
and are aggregated to 3-digit minor groups by unweighted mean (Bruegel WP
06/2024 convention); no SOC crosswalk is needed. Additional columns:
`ilo_sd_uw` (mean within-occupation task-score SD), `ilo_max_gradient`
(highest exposure gradient of any constituent 4-digit occupation),
`ilo_high_grad_share` (share of constituent 4-digit occupations in Gradients
3–4), and `n_isco4_ilo` (constituent occupation count).

**Webb index.** Webb's occupation-level exposure percentiles are attached to
O*NET-SOC codes and passed through the same SOC → ISCO-08 → NCO-2015 chain
and employment weighting as the other US-based measures. Because the index
is built from AI patents that predate large language models, it measures
exposure to AI broadly, not to generative AI. It is included as a robustness
check on how an exposure index is constructed.

## Coverage

- AIOE and Webb: absent for 011, 021, 031 (armed forces) and 951 (street
  services).
- Eloundou et al.: absent for 011 and 021.
- ILO: absent for 011, 021, 031 (armed forces are excluded from WP 140 for
  lack of task descriptions); all 427 published 4-digit occupations matched.

## Diagnostics

- Spearman rank correlations: 0.90–0.93 between the two US-based weighted
  measures; 0.81–0.83 between each US-based measure and the ILO index,
  close to the 0.78–0.82 range Bruegel WP 06/2024 reports for the EU.
- Most exposed groups: 264 (authors and journalists), 212 (mathematicians
  and statisticians), 422 (client information workers). Least exposed: 634,
  931, 911 (subsistence fishers, construction labourers, domestic cleaners).

## Merging with PLFS

PLFS rounds from 2021-22 onward, including the redesigned 2025 round, record
occupation in NCO-2015 at 3 digits; zero-pad to 3 characters and merge on
`nco2015_3d`. The 2017-18 to 2020-21 rounds use NCO-2004 and require an
ISCO-88 → ISCO-08 correspondence before merging.

## Caveats

1. **Task-content transferability.** Scores describe US task bundles; the
   same NCO code may bundle different tasks in India (Carbonero et al.,
   2023).
2. **US employment weights.** Within-group weighting uses US employment
   shares; the `_uw` columns bound this choice.
3. **Exposure is not adoption.** These are potential-exposure measures and
   describe incidence under counterfactual adoption.
4. **Aggregation noise.** 3-digit assignment averages over heterogeneous
   detailed occupations, which tends to attenuate group differences.

## Replication

`build_nco_exposure.py` rebuilds the SOC-based columns and `add_ilo_index.py`
appends the ILO columns. Sources:

- github.com/AIOE-Data/AIOE (`AIOE_DataAppendix.xlsx`)
- github.com/openai/GPTs-are-GPTs (`data/occ_level.csv`,
  `data/national_May2021_dl.csv`)
- github.com/eworx-org/iscoCrosswalks (BLS ISCO-08 ↔ SOC-2010 crosswalk;
  O*NET 2010 → 2018 SOC crosswalk)
- Webb (2020) occupation-level exposure scores, as published with the paper

Dependencies: `pandas`, `numpy`, `pyreadr`, `openpyxl`. Place the cloned
repositories and the AIOE workbook in a `raw/` folder and run
`python3 build_nco_exposure.py`, then `python3 add_ilo_index.py`.

## Provenance of the ILO index

The ILO did not publish a machine-readable file with WP 140. The 4-digit
scores in `ilo_wp140_tableA1.tsv` were transcribed from Appendix Table A1 of
the working paper (PDF). The official score file can be requested from the
ILO Research Department.
