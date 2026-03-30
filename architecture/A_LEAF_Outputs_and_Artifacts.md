# A-LEAF Outputs and Artifacts

## Purpose
This page explains the main output artifacts produced by A-LEAF and how they are used. The goal is not to document every file name in every workflow, but to show the main categories of saved results and intermediate artifacts that appear across planning, operation, and reliability runs.

## Output Categories
A-LEAF outputs generally fall into five groups:
- model result summaries
- detailed dispatch and system results
- saved intermediate artifacts used by downstream runs
- fixed-input artifacts used for reproducibility
- diagnostic or case-specific reporting outputs

## 1. Model Result Summaries
These are the main saved results that summarize a run.

Typical examples include:
- expansion results
- operation summaries
- reliability metrics
- ELCC or DLOL results

These outputs are usually the first place to look when comparing cases, checking whether a run completed successfully, or extracting high-level metrics for analysis.

## 2. Detailed Dispatch and System Results
These outputs contain more detailed operational information, such as:
- generation by unit, technology, or region
- storage charging, discharging, and state of charge
- transmission flows
- curtailment
- reserve provision or reserve shortfalls
- unserved energy

These outputs are primarily used for:
- operational interpretation
- debugging and validation
- case comparison
- policy and reliability analysis

In reliability workflows, these detailed outputs may exist for:
- reference economic dispatch
- post-contingency dispatch
- sequential dispatch runs

## 3. Saved Intermediate Artifacts
Some outputs are produced mainly so they can be reused by later model runs.

Important examples include:
- saved expansion decisions reused by operation or RA
- saved planning-state data used as predefined inputs
- representative-day selections or grouped-day mappings
- cached data products that let later runs avoid rebuilding the same state

These artifacts are important because A-LEAF supports chained workflows such as:
- GTEP -> operation
- GTEP -> RA
- repeated case studies using the same saved planning result

## 4. Fixed-Input Artifacts for Reproducibility
Some outputs are intended to freeze stochastic or sampled inputs so later runs can reuse the exact same scenario set.

Important examples include:
- fixed risk-sample files for RA
- saved outage-scenario data
- filtered joint-scenario data used to rerun RA with the same scenario set

These artifacts are especially useful when:
- validating changes to the RA model
- comparing two model versions fairly
- holding uncertainty inputs fixed while changing only part of the workflow

## 5. Diagnostic and Case-Specific Outputs
Some outputs are created only for particular workflows or analyses.

Examples include:
- ELCC-specific result tables
- DLOL outputs
- baseline dispatch exports
- case-level debugging outputs
- specialized PSH or policy-study reports

These are usually not needed for every run, but they are important for targeted studies and debugging.

## How Outputs Relate to the Simulation Workflow
The output mix depends on which model families are active in a case:

- Expansion planning runs produce planning decisions and, in many workflows, a saved system state that later operation or RA runs can reuse.
- Operation runs produce dispatch-oriented outputs such as generation, flows, costs, reserves, and shortage metrics.
- Reliability runs produce adequacy metrics, joint-scenario outputs, and optional detailed dispatch outputs for stressed scenarios.

Because one case can enable more than one model family, outputs can be chained:
- one run may produce expansion results first
- then operation or RA may use those results as its input system state

## What To Check First
When reviewing a completed case, the most useful order is usually:
1. high-level summary results
2. whether saved intermediate artifacts were produced as expected
3. detailed operational or reliability outputs
4. any case-specific diagnostic files

When results look wrong, it is often useful to check:
- whether the expected model families actually ran
- whether a predefined expansion input was used
- whether a fixed scenario sample was reused
- whether detailed output flags were enabled for the run mode you expected

## Related Documentation
- [A-LEAF Documentation](../README.md)
- [ALEAF Simulation Setting File Reference](../configuration/ALEAF_Simulation_Setting_File.md)
- [Simulation Configuration Reference](../configuration/Simulation_Configuration_Reference.md)
- [RA Overview](../models/RA/RA_Overview.md)
