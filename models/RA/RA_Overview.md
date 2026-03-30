# RA Overview

## Purpose
The Reliability Assessment (RA) model evaluates system reliability under generator outage uncertainty and renewable availability uncertainty. It uses representative days, dispatch-based redispatch modeling, and scenario filtering to estimate reliability outcomes such as unserved energy and loss-of-load risk.

## What RA Is Used For
RA is used to answer questions such as:
- whether the modeled system can maintain adequacy under forced outage conditions
- how renewable availability assumptions change reliability outcomes
- how reliability differs across buses, day groups, and the system as a whole
- how candidate resources affect reliability in ELCC-style studies

## Run Modes
RA supports the following main uses:
- Perfect-foresight post-contingency ED
- Sequential post-contingency ED
- ELCC studies that reuse the RA machinery
- DLOL reporting derived from RA outputs

The perfect-foresight and sequential modes share the same scenario framework and metric framework. The main difference is how redispatch is simulated once a contingency scenario is selected.

## Key Inputs
The main inputs to RA are:
- `ALEAF_setting`
- `RA_setting`
- network and representative-day data from the model instance
- the `RA Scenarios` sheet for renewable scenarios
- outage samples and filtered joint scenarios when fixed-risk inputs are provided

The representative-day structure and the generator/resource data prepared for RA determine the simulation horizon and the assets that are exposed to outage and renewable uncertainty.

## Key Outputs
RA produces:
- `RA_solutions`
- `RA_metrics`
- `filtered_joint_scenario_map`
- optional dispatch and system exports
- optional ELCC and DLOL outputs when those modes are enabled

The main reliability outputs are interpreted at annual, day-group, regional, and systemwide levels depending on the metric.

## High-Level Flow
1. Build the RA-ready network and representative-day structure.
2. Parse renewable scenarios and load renewable timeseries.
3. Generate outage samples and combine them with renewable scenarios.
4. Filter the joint scenario set to determine the scenarios that will be simulated.
5. Run reference ED for each retained renewable scenario and day group.
6. Run post-contingency ED for each filtered joint scenario.
7. Aggregate annual and day-group reliability metrics and export results.

## Representative Days and Renewable Scenarios
RA runs on representative-day groups rather than the full 8760-hour chronology inside the optimization model. Renewable scenario timeseries are first mapped from annual data to the representative-day structure. This matters in two places:
- filtering, because renewable availability affects the generation margin used to identify critical scenarios
- simulation, because both the reference ED and the post-contingency ED must use the renewable scenario associated with the joint scenario being solved

In the current design, reliability is evaluated over joint scenarios that combine outage uncertainty with renewable availability uncertainty.

## Recommended Reading Order
1. [RA Scenarios and Data](./RA_Scenarios_and_Data.md): outage samples, renewable scenarios, joint scenarios, fixed-risk inputs, and core RA data objects
2. [RA Settings and Scenarios Reference](./RA_Settings_and_Scenarios_Reference.md): worksheet-level explanation of the `RA Setting` and `RA Scenarios` sheets
3. [RA Execution and Results](./RA_Execution_and_Results.md): reference ED, post-contingency ED, run modes, and exported results
4. [RA Metrics, ELCC, and DLOL](./RA_Metrics_ELCC_and_DLOL.md): reliability metrics, weighting, ELCC, and DLOL


