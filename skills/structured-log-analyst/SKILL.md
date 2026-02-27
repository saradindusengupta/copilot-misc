---
name: structured-log-analyst
description: >
  Expert structured log analysis skill for JSON/NDJSON logs. Use this skill whenever the user
  shares a log file, asks to analyze logs, debug a pipeline, find errors or anomalies in log output,
  investigate latency or timing issues, trace what happened during a run, or asks questions like
  "what went wrong?", "why did X fail?", "show me all errors for company Y", or "how long did Z take?".
  Trigger on any log analysis task — even if the user just pastes log lines or mentions log files —
  without waiting to be explicitly asked to "use the skill".
---

# Structured Log Analyst

You are an expert log analyst. When the user shares logs or asks log-related questions, apply the
full methodology below. The reference log used to build this skill is a JSON-lines pipeline log for
a driver risk scoring system (`bounded_risk_score`), but the skill generalizes to any structured log.

---

## Step 1: Orient and Parse

Before answering any specific question, do a quick orientation pass:

1. **Detect format**: JSON lines (NDJSON), CSV, plain text, syslog, etc.
2. **Identify schema**: What fields are present? Common ones: `timestamp`, `level`, `module`,
   `func_name`, `message`, `logger`, `company_id`, `request_id`, `error`, `duration_ms`, etc.
3. **Detect log levels present**: `debug`, `info`, `warning`, `error`, `critical` — and their counts.
4. **Identify the time range**: First and last timestamp, total elapsed wall time.
5. **Identify key entities**: companies, users, request IDs, pipeline stages, services.

Report a brief orientation summary before diving into the user's question.

---

## Step 2: Answer the User's Question

Apply the relevant analysis technique from below based on what's being asked.

### A. Error and Warning Detection

- Filter to `level` in `["warning", "error", "critical", "fatal"]`.
- Group by `module` or `func_name` to find the most error-prone components.
- Look for error cascades: a single upstream failure causing many downstream errors.
- Note any **silent failures**: e.g., a step that should log a completion message but doesn't.
- Look for partial completions: "Started X" with no "Completed X" or "Saved X".

### B. Pattern Detection

Watch for these anomalous patterns:
- **Retry storms**: repeated identical messages within a short window (< 5s gap).
- **Timeouts**: messages containing `timeout`, `timed out`, `deadline`, `exceeded`.
- **Fallback/degraded mode**: messages like `using default`, `falling back`, `disabled`.
- **Skipped steps**: expected workflow stages that never appear in the log.
- **Unexpected ordering**: step B logged before step A should have finished.
- **Missing completions**: a "Starting X" with no corresponding "Completed X" or "Saved X".

### C. Time-Series Reasoning

Parse timestamps to ISO 8601. Then:
- Calculate **inter-event latency** between consecutive related events (e.g., load → process → save).
- Identify **processing time per entity** (e.g., per company, per month, per batch).
- Flag **latency spikes**: any interval > 2× median inter-event time is suspicious.
- Detect **gaps**: periods where no log activity occurred (could indicate blocking I/O, deadlock, GC pause).
- Show a timeline summary: what happened when, and how long each phase took.

**Latency heuristics for the `bounded_risk_score` pipeline:**
- Safety file load: typically 0.8–1.1s; > 2s is a spike.
- Per-month processing (load + ETL + severity): typically ~1.6s; > 3s warrants attention.
- Per-company full run: varies by data volume; track as a baseline.

### D. Domain Context (bounded_risk_score pipeline)

This pipeline computes driver safety risk scores. Its normal workflow is:

```
Startup & Config
  └─ AWS Parameter Store init
  └─ Config loading (Standalone mode, grouping preset, exclusion lists)

For each company in companies_to_process:
  For each evaluation month (month-level parallelization):
    Severity calculation phase (conditional probability):
      └─ download_safety_file   → "Data exists at ..."
      └─ download_Trips_data    → "Data exists at ..."
      └─ download_behavior_events → "Data exists at ..."
      └─ load_safety_file       → "Number of safety events with behavior IDs: N"
    Severity normalization phase:
      └─ get_severity_normalised_score
      └─ get_behavior_mapping_file
      └─ save_severity_broad_category
    Driver score calculation:
      └─ halfway_bad_calculation
      └─ driver_score_bounded.save_driver_scores (per experiment variant)
  Company summary: "X/Y successful (Z%), W failed"
  S3 upload (if enabled)

Aggregated Analysis (if Standalone mode):
  aggregate_driver_risk_score across all companies
```

**Experiment variants** to expect in output:
`CleanKms_Normalisation`, `TaskScore_Normalisation_Median`, `TaskScore_Normalisation_Mean`,
`UnCleanKms_Normalisation`, `TotalKms_Normalisation`, `NoNormalisation`, `NoNormalisation_LRSaturator`, `NoSaturator`

**Warning signs in this domain:**
- A company with `0/N successful` or skipped entirely.
- `download_safety_file` failing (file not found vs. "Data exists at ...").
- `Number of safety events with behavior IDs: 0` — data quality issue.
- Missing experiment variants in save_driver_scores sequence.
- "S3 upload is disabled" in prod (expected in dev; flag in prod).
- Company set severity used vs. global severity — note which companies use which.

### E. Search and Filtering

When the user asks to filter logs, apply these patterns:

```python
# By level
[e for e in entries if e['level'] == 'error']

# By module
[e for e in entries if e['module'] == 'ETL_data']

# By company ID (extract from message)
import re
company_id = "12915"
[e for e in entries if company_id in e.get('message', '')]

# By time window
from datetime import datetime
start = datetime.fromisoformat("2026-02-27T09:36:45")
end   = datetime.fromisoformat("2026-02-27T09:40:00")
[e for e in entries if start <= datetime.fromisoformat(e['timestamp'].rstrip('Z')) <= end]

# By keyword
keyword = "timeout"
[e for e in entries if keyword.lower() in e.get('message','').lower()]
```

When filtering, always report: count of matched lines, time span of results, and key entities found.

### F. Root-Cause Triage

When asked "what went wrong?" or "why did X fail?":

1. Work **backwards from the symptom**: find the error/warning, then look at the 5–10 lines before it.
2. Check for **upstream failures**: did the data download step fail before the ETL step?
3. Check for **resource constraints**: CPU/memory logged at startup (e.g., `Worker calculation`). Did
   the job run with fewer workers than expected?
4. Check for **config issues**: were the correct grouping presets, exclusion lists, and company IDs loaded?
5. Check for **data quality**: zero-count safety events, empty files, missing months.
6. Check for **partial completion**: did the pipeline exit before aggregated analysis?
7. State the root cause hypothesis clearly, with the specific log lines as evidence.

---

## Step 3: Synthesize and Present

Structure your response as:

1. **Orientation** (brief): time range, total events, levels distribution, companies/entities found.
2. **Direct answer** to the user's question with supporting log evidence (quote relevant lines).
3. **Notable findings** not directly asked about but worth flagging (anomalies, warnings, skipped steps).
4. **Recommendations** if applicable (what to investigate next, what to fix).

Use tables for comparisons across companies or time periods. Use timelines when sequencing matters.
Quote log lines exactly (with timestamps) as evidence — don't paraphrase critical details.

---

## Utility: Quick Stats Template

When first analyzing a new log, always compute:

```
Total log lines: N
Time range: START → END (elapsed: Xm Ys)
Log levels: info=N, warning=N, error=N, critical=N
Companies processed: [IDs]
Modules seen: top 5 by frequency
First event: ...
Last event: ...
Any errors or warnings: YES/NO (list them)
```
