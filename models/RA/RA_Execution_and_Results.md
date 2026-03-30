# RA Execution and Results

## Reference ED Concept
RA first computes a reference economic dispatch for each retained day group and renewable scenario. This establishes the baseline operating state before post-contingency redispatch.

The reference ED represents the system state under the renewable scenario before outage-driven redispatch is applied. This is why the reference dispatch must be keyed by renewable scenario rather than using a single deterministic baseline.

## Reference ED by Renewable Scenario
Reference ED is scenario-specific. A day group may have multiple reference ED solutions when multiple renewable scenarios are retained after filtering.

This matters because renewable availability changes baseline dispatch, unit commitment exposure, storage position, and congestion patterns in ways that affect the redispatch problem.

## Post-Contingency ED Concept
For each filtered joint scenario, RA applies the outage sample and resolves system redispatch using the matching renewable scenario. This is the core reliability simulation step.

The simulation therefore answers a conditional question: given this outage state and this renewable availability state, how much load cannot be served after redispatch?

## Perfect Foresight vs Sequential
- Perfect foresight ED evaluates redispatch with full visibility of the simulated time horizon.
- Sequential ED evaluates redispatch in a stepwise fashion and is used when sequential operating behavior matters.

Both modes use the same filtered joint scenario structure and the same renewable-scenario-specific reference ED concept. The difference is not in the scenario definition, but in how operations are represented once the scenario is selected.

## What Gets Exported
Depending on the reporting settings, RA can export:
- RA JSON result files
- reference dispatch exports
- post-contingency dispatch exports
- system-level result tables

Reference dispatch outputs are organized by day group and renewable scenario so that scenarios do not overwrite one another. Post-contingency outputs correspond to the simulated joint scenario workload.

## What To Check In Results
When validating an RA run, the main questions are:
- whether the number of filtered joint scenarios is reasonable
- whether reference ED exists for the retained renewable scenarios
- whether post-contingency outputs differ between renewable scenarios when they should
- whether the exported metrics and dispatch outputs are consistent with the selected run mode
