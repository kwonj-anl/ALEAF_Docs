# GTEP Overview

## Purpose
This section documents the generation and transmission expansion planning workflow implemented in A-LEAF.

The current focus is on:
- how the planning horizon is constructed from the workbook settings
- how multi-round expansion runs split and overlap the planning horizon
- what files are exported by the expansion reporting workflow

## What GTEP Does
The GTEP workflow solves a long-term planning problem that can include:
- generation expansion
- generator retirement
- storage duration expansion
- transmission expansion
- representative-day operational dispatch inside each planning stage
- policy, reserve, scarcity, and financial terms in the planning objective

Depending on the case settings, a GTEP case can also trigger:
- an operation run after expansion
- an RA run using the expanded system

In this documentation package, downstream operation and RA workflows are documented separately:
- [Operation overview](../Operation/Operation_Overview.md)
- [RA overview](../RA/RA_Overview.md)

## Key Documents
1. [GTEP Planning Horizon and Multi-Round](./GTEP_Planning_Horizon_and_Multi_Round.md)
2. [GTEP Expansion Outputs](./GTEP_Expansion_Outputs.md)

## Code Areas Cross-Checked
The current GTEP docs are based primarily on:
- [LCO_GTEP.jl](C:/ALEAF/a-leaf/src/model/LCO_GTEP.jl)
- [generate_network.jl](C:/ALEAF/a-leaf/src/network/generate_network.jl)
- [scenario_reduction_GTEP.jl](C:/ALEAF/a-leaf/src/util/scenario_reduction_GTEP.jl)

## Related Documentation
- [A-LEAF Documentation](../../README.md)
- [ALEAF Simulation Setting File Reference](../../configuration/ALEAF_Simulation_Setting_File.md)
- [Simulation Configuration Reference](../../configuration/Simulation_Configuration_Reference.md)
- [Scenario Reduction and Representative-Day Groups](../../architecture/Scenario_Reduction_and_Repday_Groups.md)
- [Policy and Financial Settings](../../configuration/Policy_and_Financial_Settings.md)
- [Operation Overview](../Operation/Operation_Overview.md)
