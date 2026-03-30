# GTEP Operation Execution and Run Modes

## Purpose
This page explains how the GTEP operation model is run in A-LEAF and how the supported operation run modes differ.

The descriptions in this page are based on:
- [LCO_GTEP.jl](C:/ALEAF/a-leaf/src/model/LCO_GTEP.jl)

## What The Operation Model Does
The GTEP operation model evaluates dispatch over the reduced representative-day structure for each planning stage.

It can be used in three main ways:
- as a standalone operation run without prior expansion results
- as an operation follow-on after an expansion run
- as an operation run using predefined expansion results loaded from an external file

In all three cases, the model is organized by:
- planning stage
- representative-day group
- day, hour, and time indices inside the reduced chronology

## Main Run Modes

### 1. Standalone operation run
This path uses:
- `execute_LC_GTEP_operation_model`

It is used when:
- `Run_operation_flag == true`
- the run does not depend on a newly solved expansion case in the same workflow

In this mode:
- A-LEAF generates operation network data directly from the workbook inputs
- no expansion decisions are fixed from a preceding expansion solution
- the operational model evaluates dispatch on the configured planning stages and day groups

### 2. Operation after expansion
This path uses:
- `execute_LC_GTEP_operation_model_after_expansion_run`

It is used when:
- an expansion run has just been solved
- the study wants to run the operation model on the expanded system

In this mode:
- A-LEAF extracts expansion decisions from the solved expansion result
- those decisions are reused when building the operation network data
- renewable investment totals by bus are also recorded for later use in network-data generation

This is the most common coupled workflow when the study wants both:
- long-term expansion decisions
- operational results under the chosen expansion plan

### 3. Operation using predefined expansion data
This path uses:
- `execute_LC_GTEP_operation_model_using_predefined_expansion_data`

It is used when:
- the expansion result is not being solved in the current run
- the study wants to reuse saved expansion data from an earlier run

In this mode:
- expansion decisions are loaded from external data
- the operation model is built against that fixed expansion state

## Core Execution Flow
The operation workflow is conceptually:
1. build or load the expansion state to use
2. generate operation network data for each planning stage
3. build the operation model for each day group
4. solve the operation subproblems
5. collect duals if needed
6. export JSON and CSV reporting outputs

## How Planning Stages Enter The Operation Model
The operation workflow still uses the planning stages built for GTEP.

For each stage `y`:
- A-LEAF generates stage-specific operation `network_data`
- stores stage-specific references in `OP_data_record_for_reporting`
- solves each configured operation day group for that stage

So the operation model is not a single one-year dispatch run. It is a stage-by-stage operational evaluation over the study horizon.

## How Representative-Day Groups Enter
For operation runs, day groups come from:
- `Simulation Configuration["NDAY_Groups_OP"]`

For each stage:
- A-LEAF loops over operation day groups
- each day group maps to a block of representative days through `repday_groups`
- results are stored under:
  - year
  - day-group id

This is why the operation JSON and reporting outputs are organized as:
- planning stage
- day group
- reduced day/hour/time detail

## Serial vs Distributed Operation Solves

### Serial operation solve
This path uses:
- `build_solve_LC_GTEP_operational_model`

It solves day groups sequentially inside each planning stage.

### Distributed operation solve
This path uses:
- `build_solve_LC_GTEP_operational_model_distributed`

It is activated when:
- `Simulation Setting["run_operation_in_parallel_flag"] == true`

In this mode:
- each planning stage still runs separately
- day-group subproblems are distributed across available workers

The reporting outputs are the same; only the solve orchestration changes.

## What Gets Fixed From Expansion
When expansion results are available, operation runs use:
- fixed generator expansion decisions
- fixed retirement decisions
- fixed storage-duration decisions

These are passed into the operational build through:
- `result_LC_GTEP_expansion`
- `expansion_fix = true`

This lets the operation model evaluate dispatch on the expanded system rather than re-optimizing the long-term decisions.

## Important Implementation Detail
Before the operation subproblem is built, A-LEAF disables expansion-only controls in the operational instance:
- `multi_round_solution_process_flag = false`
- `transmission_expansion_flag = false`

That means the operation run uses the expanded network and asset state, but does not perform a new transmission expansion optimization inside the operation solve.

## LP vs Non-LP Operation Models
The operation path supports both:
- LP-style operation runs
- non-LP operation runs with a second fixed-integer pass for dual recovery

The code checks the operation model type through:
- `model_type_OP`

If the model is not LP:
- A second model instance with fixed integer variables is used to recover dual values after the main solve

## Operation Outputs
The operation export entry point is:
- [export_LC_GTEP_result_OP](C:/ALEAF/a-leaf/src/model/LCO_GTEP.jl)

It produces:
- an optional JSON output
- operation dispatch, market, policy, demand-response, power-flow, summary, and representative-day CSV outputs

Those files are documented in:
- [Operation Outputs](./Operation_Outputs.md)

## Relationship To Expansion Outputs
The operation reports are designed to be comparable with the expansion reports:
- similar dispatch and market file structures
- similar tech-summary and system-summary outputs
- similar representative-day reporting

The key difference is that the operation outputs reflect:
- fixed expansion decisions
- operation-only outcomes

not the integrated expansion objective.

## Practical Reading Guide
If you are reviewing an operation run, check:
1. whether the run is standalone, after expansion, or predefined-expansion
2. whether operation was run serially or in parallel
3. how many stages are included
4. how many operation day groups are configured
5. whether expansion decisions were fixed into the run

## Related Documentation
- [Operation Overview](./Operation_Overview.md)
- [Operation Outputs](./Operation_Outputs.md)
- [GTEP Overview](../GTEP/GTEP_Overview.md)
- [GTEP Expansion Outputs](../GTEP/GTEP_Expansion_Outputs.md)
- [Scenario Reduction and Representative-Day Groups](../../architecture/Scenario_Reduction_and_Repday_Groups.md)
