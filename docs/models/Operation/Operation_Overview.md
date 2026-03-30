# Operation Overview

## Purpose
This section documents the standalone operation model in A-LEAF.

In this documentation package, the operation model is treated as a separate model family from GTEP:
- `GTEP` means generation and transmission expansion planning
- `Operation` means the operational dispatch model and its outputs

The operation model can be run:
- as a standalone case
- after an expansion run
- using predefined expansion results

## What The Operation Model Does
The operation model evaluates how a given system configuration is dispatched over representative days and planning stages. Depending on case settings, it can include:
- economic dispatch and commitment behavior
- storage operation
- reserve provision
- large-load and demand-response behavior
- hybrid plant operation
- policy tracking and market outputs

## Key Documents
1. [Operation Execution and Run Modes](./Operation_Execution_and_Run_Modes.md)
2. [Operation Outputs](./Operation_Outputs.md)

## Related Documentation
- [A-LEAF Documentation](../../README.md)
- [ALEAF Simulation Setting File Reference](../../configuration/ALEAF_Simulation_Setting_File.md)
- [Simulation Configuration Reference](../../configuration/Simulation_Configuration_Reference.md)
- [Scenario Reduction and Representative-Day Groups](../../architecture/Scenario_Reduction_and_Repday_Groups.md)
- [Policy and Financial Settings](../../configuration/Policy_and_Financial_Settings.md)
- [Storage Modeling Reference](../../database/Storage_Modeling_Reference.md)
- [Hybrid Resources Reference](../../database/Hybrid_Resources_Reference.md)
- [Large Load and Demand Response Reference](../../database/Large_Load_and_Demand_Response_Reference.md)
