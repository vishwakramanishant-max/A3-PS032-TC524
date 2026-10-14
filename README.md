# AIORI 3 — RPKI Path & Reachability Analysis

Team Debugs | A3-PS032-TC524 | Challenge PS-032 

A measurement-analysis pipeline that correlates timestamped RPKI origin-validation state changes with paired traceroute and reachability data, and classifies forwarding paths as **changed**, **unchanged** or **inconclusive**, while preserving ambiguous and missing data.

## Key Features

- **Timestamp Alignment:** Converts every dataset to UTC, flags naive, epoch and offset timestamps, and warns when a dataset does not overlap the event window.
- **Paired Before/After Windows:** Compares measurements around each transition, with a settle-time guard so BGP convergence does not pollute the result.
- **Path Classification:** Labels each vantage point as changed (reroute, loop, reachability loss or gain), unchanged, or inconclusive.
- **Missing Data Preserved:** An empty observation is never counted as a failure. Thin or unstable data is reported as inconclusive.
- **Silent-Hop Tolerance:** Builds a consensus AS-level path per window and treats unresponsive hops as wildcards.
- **Sample Counts Everywhere:** Every event and every result category reports how many vantage points and traceroutes support it.
- **Event Timelines and Case Studies:** Per-event timelines and a per-vantage-point table are generated automatically.
- **Correlation, Not Proof:** Every report states that a path change near an RPKI state change does not prove an operator deployed ROV.

## Architecture

The pipeline follows a layered design:

1. **Input & Alignment Layer:** Loads RPKI transition logs, traceroutes and reachability data, then normalises and checks timestamps.
2. **Analysis Layer:** Splits observations into before, guard and after windows, builds consensus paths, and classifies each vantage point.
3. **Reporting Layer:** Produces per-event summaries, sample counts, timelines and machine-readable outputs.

## Ethics & Safety

The analysis is fully offline. It makes no live ROA changes, sends no probes, and has no measurement-credit dependency. Results are reported as correlation only, and missing data and sample sizes are shown openly.

## Presentation

- 📄 Presentation (PDF): `docs/presentation/[file name].pdf`
- 🎥 YouTube Intro: [WATCH]([paste YouTube link here])

## Team

| Jasmeet | Sakec | Team lead: problem understanding and solution design |
| Nishant | Sakec | Data engineer: analysis pipeline |
| Uma | Sakec | Results analyst: case studies, report and demo |

## Repository Layout

```
src/        pipeline.py, generate_sample_data.py
data/       events.csv, traceroutes.jsonl, reachability.csv
results/    summary.md, events.json, paths.csv
docs/       PSEUDOCODE.md, presentation/, video/
```

## Quick Start (Python 3.9+, no dependencies)

```bash
python3 src/generate_sample_data.py   # synthetic demo data -> data/ (skip when using real data)
python3 src/pipeline.py               # -> results/summary.md, events.json, paths.csv
```

Useful options: `--window-min 30`, `--guard-after-min 5`, `--min-samples 3`.

**Input formats:** `events.csv` (event_id, prefix, origin_asn, from_state, to_state, transition_time), `traceroutes.jsonl` (`{event_id, vp, dst, ts, hops, reached}`), `reachability.csv` (event_id, vp, ts, ok; empty = missing).

## Project Status

Developed for Challenge PS-032: Path and Reachability Analysis During RPKI State Changes. The prototype currently runs end to end on **synthetic data**. Classification thresholds are heuristics and must be validated and tuned on the organiser-supplied events before drawing conclusions.
