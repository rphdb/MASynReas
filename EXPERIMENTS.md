# Experiments

These are the experiments reproducible with the scripts in this repository. The full evaluation, including the 27-scenario synthetic catalogue and the comparisons with an LLM detector and classifier, is in the [final report](reports/final_report.pdf).

All experiments were run on **noria-0.2**, the reference knowledge graph provided by Orange for network diagnosis (3,152 RDF triples covering the four facets of the [NORIA-O](https://github.com/Orange-OpenSource/noria-ontology) ontology), loaded in a local Virtuoso SPARQL endpoint (`localhost:8890`).

The reference incident is a software failure on resource `RES_TOY_as2` (L2 table overflow), which cuts access to application `APP_TOY_PFS01` and propagates to `RES_TOY_srv1`.

| Experiment | Script | Raw results |
|---|---|---|
| 1. Family complementarity | `complementarity_table.py` | `complementarity_results.json` |
| 2. Ablation: added value of each MAS layer for the LLM | `ablation_study.py` | `ablation_results.json` |
| 3. Sequential vs parallel runtime | `baseline_monoagent.py` | `baseline_results.json` |

---

## 1. Family complementarity

**Question:** are the four detector families redundant, or does each one see anomalies the others miss?

### A priori mode (23 detectors, ~10 s)

| Family | Active agents | Detections | Detects entities no other family sees |
|---|:---:|:---:|:---:|
| Structural | 2 / 10 | 17 | yes |
| Dynamic | 1 / 3 | 10 | yes |
| Functional | 2 / 7 | 7 | yes |
| Procedural | 1 / 3 | 2 | yes |

Every family contributes unique detections. Most active detectors:
- `missing_interface_detector` (structural): 14 resources without a network interface
- `event_without_related_element_detector` (dynamic): 10 events without a related element
- `inconsistent_service_hierarchy_detector` (functional): 4 inconsistent service hierarchies
- `ticket_without_assigned_procedure_detector` (procedural): 2 tickets without a procedure

**Synergy cases.** Three applications (`APP_TOY_DC01`, `APP_TOY_DC02`, `APP_TOY_MON01`) are flagged both by a structural detector (`application_without_support`) and by a functional one (`application_without_resource`). Without level 2 these remain two independent alerts; `structural_fragility_diagnoser` merges them into a single diagnosis.

### A posteriori mode (31 detectors, ~12 s)

| Family | Active agents | Detections |
|---|:---:|:---:|
| Structural | 0 / 7 | 0 |
| Dynamic | 0 / 12 | 0 |
| Functional | 7 / 9 | 19 |
| Procedural | 0 / 3 | 0 |

On noria-0.2, only the functional family fires in a posteriori mode: the trouble tickets in this graph mainly impact the application and service layer. The other families are exercised by the synthetic scenarios in [`datasets/`](datasets/README.md).

---

## 2. Ablation study: what does each layer add to the LLM?

**Question:** if an LLM is available, what is the multi-agent system for?

> This section reports a local CPU run of the ablation with `llama3.1:8b`, reproducible with `ablation_study.py`. The [final report](reports/final_report.pdf) presents a later GPU run with Mistral-7B-Instruct-v0.3 (3 runs per condition, slightly different metric definitions and the graph rather than the ticket text as baseline input), where conformity rises from 0.31 (graph only) to 0.53 (+ level 1) and 0.78 (+ level 1 + level 2).

### Protocol

Three conditions on the same scenarios; only the LLM input changes.

| | Condition A | Condition B | Condition C |
|---|---|---|---|
| **LLM input** | Trouble ticket text (what a human operator sees) | Level-1 detector outputs (JSON) | Level-1 outputs + level-2 diagnoses |
| **Pipeline** | LLM only | Level 1 → LLM | Level 1 + 2 → LLM |

- **Model:** `llama3.1:8b` via Ollama (local, CPU)
- **Runs:** 5 per scenario and condition (~25 minutes in total)
- **Metrics** (computed automatically):
  - *Hallucination rate*: share of entity names in the answer that do not exist in the scenario (lower is better)
  - *Conformity*: share of ground-truth elements (target entity, diagnosis type, severity) found in the answer (higher is better)
  - *Consistency*: similarity between the 5 runs (higher is better)
  - *Severity match*: 1 if the ground-truth severity is reproduced, else 0

### Scenarios

| ID | Description | Ground truth |
|---|---|---|
| S1 | `RES_TOY_as2` triggers 3 independent structural signals | single point of failure, CRITICAL |
| S2 | Change `CR_2022_001` corroborated by a dynamic and a procedural signal | change-induced incident, HIGH |
| S3 | Incidents without tickets and tickets without events | traceability breakdown, HIGH |
| S4 | `RES_TOY_as2` accumulates a priori governance weaknesses | structural fragility, HIGH |
| S5 | `RES_TOY_as2` is both a SPOF and affected by change `CR_2022_001` | single point of failure, CRITICAL |

### Results (averages over 5 scenarios × 5 runs)

| Condition | Hallucination ↓ | Conformity ↑ | Consistency ↑ | Severity match ↑ |
|---|:---:|:---:|:---:|:---:|
| A: LLM only | 0.00 | 0.60 | **0.68** | **1.00** |
| B: Level 1 → LLM | 0.05 | 0.53 | 0.57 | 0.60 |
| **C: Level 1 + 2 → LLM** | **0.00** | **0.87** | 0.66 | **1.00** |

### Per scenario (hallucination / conformity / consistency / severity match)

| Scenario | A: LLM only | B: Level 1 → LLM | C: Level 1 + 2 → LLM |
|---|:---:|:---:|:---:|
| S1 SPOF | 0.00 / 0.67 / 0.84 / 1 | 0.25 / 1.00 / 0.44 / 1 | 0.00 / 1.00 / 0.54 / 1 |
| S2 Change-induced | 0.00 / 0.67 / 0.52 / 1 | 0.00 / 0.33 / 0.82 / 0 | 0.00 / 0.67 / 0.82 / 1 |
| S3 Traceability | 0.00 / 0.33 / 0.80 / 1 | 0.00 / 0.33 / 0.52 / 1 | 0.00 / 0.67 / 0.62 / 1 |
| S4 Structural fragility | 0.00 / 0.67 / 0.70 / 1 | 0.00 / 0.67 / 0.51 / 1 | 0.00 / 1.00 / 0.70 / 1 |
| S5 Cross-family | 0.00 / 0.67 / 0.53 / 1 | 0.00 / 0.33 / 0.56 / 0 | 0.00 / 1.00 / 0.63 / 1 |

### Analysis

- **The correlation layer is what makes the LLM reliable.** Condition C reaches the best conformity (0.87, vs 0.60 for the LLM alone), with no hallucination and the correct severity in every scenario. It is the only condition reaching full conformity on S1, S4 and S5.
- **Raw, uncorrelated facts can mislead the LLM.** Condition B scores *below* the LLM alone: on S2 and S5 it predicts CRITICAL instead of HIGH (conformity 0.33), and on S1 it finds the right answer but invents an entity. The level-1 outputs mention `RES_TOY_as2` in several independent files (isolated, no redundancy, high impact, change-linked); without correlation, the LLM focuses on the most visible resource and over-diagnoses. The level-2 diagnosis (e.g. *"change CR_2022_001, dual corroboration, severity HIGH"*) reframes it correctly.
- **The cross-family scenario shows the largest gap** (S5: conformity 1.00 with level 2 vs 0.33 without), precisely where several families produce competing signals.
- **Consistency.** The LLM alone is slightly more consistent (0.68) because the vague ticket text yields similarly vague answers; condition B is the least consistent (0.57); condition C combines high conformity with similar consistency (0.66).

**Takeaway:** the MAS does not replace the LLM; it filters, structures and correlates the knowledge-graph facts first, so that the LLM only verbalizes a diagnosis that is already established and traceable.

Scope: 5 hand-built scenarios on one reference graph, with automatic string-based metrics. These results are indicative rather than a full benchmark.

---

## 3. Sequential vs parallel runtime

`baseline_monoagent.py` runs every level-1 detector one after another in a single process (no JaCaMo) and compares the total time with the slowest single detector, i.e. the best case for parallel agents.

| Mode | Detectors | Sequential | Slowest detector | Theoretical speedup |
|---|:---:|:---:|:---:|:---:|
| A priori | 23 | 4.9 s | 0.39 s | 12.6× |
| A posteriori | 31 | 6.5 s | 0.35 s | 18.7× |

---

## Reproducing the experiments

Prerequisites: Virtuoso on `localhost:8890` with noria-0.2 loaded, and Ollama with `llama3.1:8b` pulled (for the ablation). On Windows, keep the `-X utf8` flag.

```bash
# 1. Run all level-1 detectors (both modes)
python -X utf8 run_all_detectors.py both

# 2. Complementarity table
python -X utf8 complementarity_table.py

# 3. Ablation study (~25 minutes)
python -X utf8 ablation_study.py

# 4. Sequential baseline
python -X utf8 baseline_monoagent.py both
```

Outputs: per-detector JSON files in `results/<family>/<mode>/`, diagnoses in `results/level2/<mode>/`, and the three result files listed at the top of this page.
