# Dataset Catalogue

## Purpose

This folder contains the synthetic benchmark catalogue used to evaluate MASynReas: 27 scenarios (`DS01` to `DS27`) derived from two reference knowledge graphs.

The catalogue is designed to support the KPI framework defined for the project, especially:
- false positive control
- activation correctness
- `Precision@L2`
- `Recall@L2`
- `F1@L2`
- `Top-1 Diagnosis Accuracy`
- diagnosis ranking quality
- reliability-score calibration
- priority-score calibration
- runtime and SPARQL efficiency
- robustness to missing data
- robustness to noise
- family contribution analysis
- scalability curves

The guiding principle is to build datasets from a small number of coherent reference graphs, then derive scenario datasets by controlled anomaly injection, data removal, or noise addition.

Each scenario folder contains:
- `dataset.ttl`: the scenario graph (Turtle)
- `README.md`: purpose, base graph and mutation profile
- `manifest.json`: base graph, mode, mutation profile, KPI targets and expected summary
- `expected_level1.json`: expected detector outputs
- `expected_level2.json`: expected diagnoses

The whole catalogue can be regenerated with [generate_catalogue.ps1](generate_catalogue.ps1)

## Reference Graphs

### Base A

`Base A` is the main small coherent reference graph.

It serves as the clean control graph for both modes:
- the clean `apriori` control graph
- the clean `aposteriori` control graph

So one same healthy graph will support:
- clean `apriori` tests
- clean `aposteriori` tests
- partial-evidence scenarios
- several integrated anomaly scenarios derived by mutation

Materialized files:
- [Base A README](baseA/README.md)
- [baseA.ttl](baseA/baseA.ttl)

### Base B

`Base B` is a richer graph with more events, changes, tickets, services, and dependencies.

It is intended mainly for:
- more complex integrated scenarios
- ranking scenarios
- runtime measurements
- robustness tests
- scalability experiments

Materialized files:
- [Base B README](baseB/README.md)
- [baseB.ttl](baseB/baseB.ttl)

## Catalogue Structure

The catalogue contains 27 scenarios:
- 3 control / activation scenarios
- 7 integrated `apriori` diagnosis scenarios
- 8 integrated `aposteriori` diagnosis scenarios
- 4 robustness scenarios
- 5 ranking / calibration / scalability scenarios

## Scenario Catalogue

### Control / Activation Scenarios

#### `DS01_clean_baseA`

Mode: `both`

Idea:
- healthy coherent graph derived directly from `Base A`
- no intended anomaly

Main KPI targets:
- false positive control
- activation correctness
- baseline runtime
- traceability sanity check

Materialized files:
- [DS01 README](DS01_clean_baseA/README.md)
- [DS01 graph](DS01_clean_baseA/dataset.ttl)
- [DS01 manifest](DS01_clean_baseA/manifest.json)

#### `DS02_partial_evidence_no_l2_v1`

Mode: `both`

Idea:
- weak or isolated level-1 signals
- not enough convergence for any level-2 diagnosis

Main KPI targets:
- activation correctness
- false positive resistance

#### `DS03_partial_evidence_no_l2_v2`

Mode: `both`

Idea:
- multi-signal scenario
- several weak signals may coexist
- still no coherent anchor or evidence combination should justify a level-2 diagnosis

Purpose:
- second version of the partial-evidence family
- justifies explicit testing of more complex false-positive resistance

Main KPI targets:
- activation correctness
- false positive resistance under multi-signal conditions

### Integrated A Priori Diagnosis Scenarios

#### `DS04_structural_fragility_resource_anchor`

Mode: `apriori`

Idea:
- one resource accumulates missing interface, weak parentage, no management, or similar structural weaknesses

Expected main diagnosis:
- `structural_fragility_diagnoser`

#### `DS05_critical_service_exposure_basic`

Mode: `apriori`

Idea:
- critical application or service with weak redundancy and poor resource coverage

Expected main diagnosis:
- `critical_service_exposure_diagnoser`

#### `DS06_critical_service_exposure_concentrated_service`

Mode: `apriori`

Idea:
- stronger or more concentrated version of `DS05`
- intended specifically to compare the resulting severity score with `DS05`

Expected main diagnosis:
- `critical_service_exposure_diagnoser`

Main KPI targets:
- severity-score comparison
- priority-score comparison

#### `DS07_observability_gap_events_and_changes`

Mode: `apriori`

Idea:
- missing timestamps
- missing related elements
- missing effective or scheduled change times

Expected main diagnosis:
- `observability_gap_diagnoser`

#### `DS08_procedural_unreadiness_basic`

Mode: `apriori`

Idea:
- tickets without procedures
- procedures not linked to resource types
- unscheduled changes

Expected main diagnosis:
- `procedural_unreadiness_diagnoser`

#### `DS09_functional_mapping_gap_application_chain`

Mode: `apriori`

Idea:
- incomplete or inconsistent service-application-resource mapping

Expected main diagnosis:
- `functional_mapping_gap_diagnoser`

#### `DS10_apriori_mixed_two_true_diagnoses`

Mode: `apriori`

Idea:
- one area of the graph should trigger one apriori diagnosis
- another area should trigger a different apriori diagnosis

Expected use:
- multi-diagnosis ranking and family contribution

### Integrated A Posteriori Diagnosis Scenarios

#### `DS11_single_point_of_failure_basic`

Mode: `aposteriori`

Idea:
- incident-related resource is isolated, non-redundant, incomplete-link attached, and high-impact

Expected main diagnosis:
- `single_point_of_failure_diagnoser`

#### `DS12_change_induced_incident_basic`

Mode: `aposteriori`

Idea:
- one change is followed by incident(s) with reinforcing change-related evidence

Expected main diagnosis:
- `change_induced_incident_diagnoser`

#### `DS13_service_cascade_basic`

Mode: `aposteriori`

Idea:
- one technical issue propagates through a shared dependency or module and affects several services

Expected main diagnosis:
- `service_cascade_diagnoser`

#### `DS14_traceability_breakdown_basic`

Mode: `aposteriori`

Idea:
- incidents without tickets
- tickets without linked events
- weak procedural escalation

Expected main diagnosis:
- `traceability_breakdown_diagnoser`

#### `DS15_unstable_component_basic`

Mode: `aposteriori`

Idea:
- repeated events
- bursts
- flapping
- reopened or stale incidents around the same component

Expected main diagnosis:
- `unstable_component_diagnoser`

#### `DS16_application_support_failure_basic`

Mode: `aposteriori`

Idea:
- application anomaly combined with broken or inconsistent support mapping

Expected main diagnosis:
- `application_support_failure_diagnoser`

#### `DS17_local_infrastructure_cluster_basic`

Mode: `aposteriori`

Idea:
- local spatial cluster
- propagation
- synchronous incidents
- local structural weakness

Expected main diagnosis:
- `local_infrastructure_cluster_diagnoser`

#### `DS18_aposteriori_mixed_two_true_diagnoses`

Mode: `aposteriori`

Idea:
- two true diagnosis families coexist in different areas of the graph

Expected use:
- `Precision@L2`
- `Recall@L2`
- ranking quality
- family contribution

### Robustness Scenarios

#### `DS19_missing_data_structural_and_temporal`

Mode: `both`

Idea:
- remove timestamps, related elements, or adjacency information from an otherwise diagnosable scenario

Main KPI targets:
- `MissingDataRobustness`

#### `DS20_missing_procedural_links`

Mode: `aposteriori`

Idea:
- remove some ticket-event or ticket-procedure links from an otherwise coherent anomaly case

Main KPI targets:
- `MissingDataRobustness`
- traceability robustness

#### `DS21_noisy_irrelevant_events`

Mode: `aposteriori`

Idea:
- inject irrelevant extra events, unrelated tickets, or benign extra records

Main KPI targets:
- `NoiseRobustness`
- ranking stability
- SPARQL efficiency under noise

#### `DS22_noisy_duplicate_and_inconsistent_records`

Mode: `both`

Idea:
- duplicate some records
- inject weakly inconsistent but non-decisive links

Main KPI targets:
- `NoiseRobustness`
- false positive sensitivity

### Ranking / Calibration / Scalability Scenarios

#### `DS23_three_diagnoses_ranked_by_urgency`

Mode: `aposteriori`

Idea:
- same dataset contains several true diagnoses with different operational urgency

Main KPI targets:
- `Top-1 Diagnosis Accuracy`
- diagnosis ranking quality
- priority-score calibration

#### `DS24_reliability_calibration_bundle`

Mode: `both`

Idea:
- several diagnosis candidates intentionally built with weak, plausible, and strong evidence levels

Main KPI targets:
- reliability-score calibration
- activation-threshold sanity

#### `DS25_scalability_small`

Mode: `both`

Idea:
- small version of a realistic mixed graph

Main KPI targets:
- end-to-end runtime
- mean runtime per detector
- mean runtime per diagnoser
- SPARQL efficiency
- first point of the scalability curve

#### `DS26_scalability_medium`

Mode: `both`

Idea:
- medium version of the same graph logic as `DS25`

Main KPI targets:
- second point of the scalability curve

#### `DS27_scalability_large`

Mode: `both`

Idea:
- large version of the same graph logic as `DS25` and `DS26`

Main KPI targets:
- third point of the scalability curve

## Notes on Use

### Rule Conformance at Level 1

The main dataset catalogue is not intended to replace detector-level validation fixtures.

For `Rule Conformance @L1`, a separate micro-suite should still be maintained with:
- positive cases
- negative cases
- boundary cases

The present catalogue is mainly intended for:
- integrated level-2 evaluation
- robustness analysis
- runtime analysis
- calibration analysis
- family contribution analysis

### Naming Convention

The current dataset identifiers are stable scenario names.

Scenario folders follow the `DSxx_<short_description>` convention:

```text
datasets/
  DS01_clean_baseA/
  DS02_partial_evidence_no_l2_v1/
  DS03_partial_evidence_no_l2_v2/
  ...
```
