# bounded_risk_score Pipeline Reference

## System Overview

**Service**: `bounded_risk_score` (Lytx driver safety scoring)  
**AWS Region**: `us-west-2`  
**Parameter Store Root**: `/lytx_bounded_risk_score`  
**Data Root**: `/mnt/data/168_companies_data/{company_id}/`

## File Paths by Data Type

| Data Type | Path Pattern |
|-----------|-------------|
| Safety events | `{company_id}/Safety/year={Y}/month={MM}/{company_id}_{start}_to_{end}_Safety.csv` |
| Trip data | `{company_id}/Trips_DeviceTripsUnified/year={Y}/month={MM}/{start}_to_{end}_tripinformation_DeviceTripsUnified.csv` |
| Behavior event scores | `{company_id}/BehaviorEventScore/year={Y}/month={MM-1}/{company_id}_{prev_start}_to_{end}_behaviorEventScore.csv` |
| Severity output | `Severity_and_Saturator/{company_id}/year={Y}/month={MM}_{MM-1}/Severity_broad_categories/{grouping_preset}/` |
| Driver scores | `Companywise_DriverScores/{company_id}/year={Y}/month={MM}/{variant}/{start}_to_{end}.csv` |

**Note**: BehaviorEventScore uses a 2-month lookback window — the file month is one month prior to the evaluation month.

## Modules and Their Roles

| Module | Responsibility |
|--------|---------------|
| `aws_utils` | AWS Parameter Store client init |
| `config` | Load run config (mode, grouping, exclusions) |
| `main` | Orchestration: company loop, parallelization, S3 upload |
| `kitchen_sink` | Worker count optimization |
| `Data_download_Stats` | Check/download Safety, Trips, BehaviorEvent files |
| `ETL_data` | Load and transform raw CSV files |
| `broad_category_severity` | Compute conditional probability severity scores |
| `Severity_Calculation` | Severity dict lookup (company-set vs. global) |
| `halfway_bad_calculation` | Intermediate risk score calculation |
| `driver_score_bounded` | Final driver score computation and saving |
| `aggregated_analysis` | Industry-wide aggregation across all companies |

## Experiment Variants (expected in every company-month run)

1. `CleanKms_Normalisation`
2. `TaskScore_Normalisation_Median`
3. `TaskScore_Normalisation_Mean`
4. `UnCleanKms_Normalisation`
5. `TotalKms_Normalisation`
6. `NoNormalisation`
7. `NoNormalisation_LRSaturator`
8. `NoSaturator`

If any variant is missing from the save sequence, that month's output is incomplete.

## Severity Modes

- **Company Set severity** (`Use company Set severity for company id: X`): company-specific severity weights
- **Group severity** (default): uses the group's pooled severity; triggered by `get_severity_normalised_score_Conditional_probability_groupCompanies`

## Parallelization

- Uses month-level parallelization with `kitchen_sink.calculate_optimal_workers`
- Workers selected as `min(workers_by_cpu, workers_by_memory, 5)` typically
- Companies processed sequentially; months within a company processed in parallel

## Normal Performance Baselines

| Operation | Typical Duration |
|-----------|-----------------|
| Safety file load (large company ~25k events) | 0.8–1.2s |
| Safety file load (small company ~4k events) | 0.15–0.35s |
| Per-month severity calculation | ~0.6s |
| Per-experiment variant save | ~0.07–0.15s |
| Full company run (9 months, 8 variants) | ~10–20s |
| Aggregated analysis | ~1s |

## Known Patterns / Gotchas

- `S3 upload is disabled` — expected in standalone/dev mode, flag if seen in prod
- BehaviorEventScore file month is always `eval_month - 1` (1-month lookback)
- `GT_behavior_toberemoved` list in config is labeled `toberemoved` — verify it's intentional
- Safety event counts typically: large companies 20k–35k/month, small companies 3k–7k/month
- `aggregate_driver_risk_score` only runs if `Aggregated Analysis is enabled` in config
