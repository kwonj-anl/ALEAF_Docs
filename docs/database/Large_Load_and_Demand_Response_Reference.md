# Large Load and Demand Response Reference

This page explains how large flexible load and demand-response style resources are represented in A-LEAF using the dedicated `demand` sheet in the network workbook and the explicit large-load constraints used in the expansion and operation models.

In the current codebase, these resources are often referred to as:
- large load
- large flexible load
- LFL
- demand response

The `demand` sheet is the dedicated network-data surface for them.

## Data Surface: `demand` Sheet
The base network workbook includes a dedicated `demand` sheet. The actual fields include:
- `PLANT_NAME`
- `bus_ID`
- `bus_name`
- `Online_Year`
- `UNITGROUP`
- `UNIT_CATEGORY`
- `UNIT_TYPE`
- `CAP`
- `INTERCON_LIM`
- `PMAX`
- `PMIN`
- `FOR`
- `Integer_Flag`
- `Daily_DR_Limit_MWh`
- `Num_DR_Segments`
- `Pct_MW_1` to `Pct_MW_5`
- `Price_1` to `Price_5`
- `Hybrid_Gen`
- `Hybrid_Gen_CAP`
- `Hybrid_ES`
- `Hybrid_ES_CAP`
- `Grid_Charge`

The sample entries in the North America database include resources such as:
- `LFL_DataCenter`
- `LFL_CriptoMining`

## What These Fields Mean

## Core load fields
- `CAP` defines the modeled size of the large load.
- `INTERCON_LIM` defines the interconnection limit used in both injection and withdrawal constraints.
- `PMAX` and `PMIN` define the operating range relative to the load structure.
- `FOR` provides the outage-style availability parameter carried with the demand object.

## Demand-response segmentation
- `Daily_DR_Limit_MWh` limits how much daily demand response is available.
- `Num_DR_Segments` defines how many price/quantity segments are active.
- `Pct_MW_1` to `Pct_MW_5` define the quantity share of each segment.
- `Price_1` to `Price_5` define the segment prices.

This is the main mechanism that gives large flexible load an economic curtailment structure rather than treating it as fixed inflexible load.

## Optional onsite hybrid support
The `demand` sheet also includes:
- `Hybrid_Gen`
- `Hybrid_Gen_CAP`
- `Hybrid_ES`
- `Hybrid_ES_CAP`
- `Grid_Charge`

Those fields allow a large-load entry to have onsite generation and storage, using the same component-matching logic as the hybrid plant workflow.

## How The Code Uses Large Load
Large-load logic is built from the `demand` sheet in the shared network build, and then explicit variables and constraints are added in the expansion and operation formulations.

The key model-side variables include:
- `lfl_g_G_LFL_ldhty`
- `lfl_g_G_Grid_ldhty`
- `lfl_g_G_ES_ldhty`
- `lfl_g_ES_LFL_ldhty`
- `lfl_g_ES_Grid_ldhty`
- `lfl_chg_Grid_ES_ldhty`
- `lfl_soc_ldhty`

These represent:
- load served by onsite generation
- load served by the grid
- power routed between large load and storage
- onsite storage SOC

## Expansion and Operation Constraints
The large-load formulation is explicit in [constraints.jl](C:/ALEAF/a-leaf/src/component/constraints.jl). The key constraint families are:
- `constraint_LFL_power_balance_ldhty`
- `constraint_LFL_Limit_ldhty`
- `constraint_LFL_inter_connection_limit_injection_ldhty`
- `constraint_LFL_inter_connection_limit_withdraw_ldhty`
- `constraint_LFL_segment_bound_lsdhty`
- `constraint_LFL_segment_relation_lsdhty`

If the large load has onsite storage, the formulation also uses:
- `constraint_lfl_onsite_ES_SOC_cap_ldhty`
- `constraint_lfl_onsite_ES_SOC_balance_ldhty`
- `constraint_lfl_onsite_ES_SOC_neutral_ldhty`

These are called directly from the expansion and operation model builders in [LCO_GTEP.jl](C:/ALEAF/a-leaf/src/model/LCO_GTEP.jl).

What they mean at a high level:
- power-balance constraints allocate how the large load is served
- interconnection constraints cap site import and export
- segment constraints enforce the demand-response block structure
- onsite-storage constraints track any storage paired with the load

## How The Objective Uses It
The operation and expansion objective paths include explicit large-load demand-response cost accounting. In [LCO_GTEP.jl](C:/ALEAF/a-leaf/src/model/LCO_GTEP.jl), the objective comments and reporting blocks label this as:
- `LFL demand response cost`

That means the segment prices in the `demand` sheet are not passive metadata. They directly affect optimization outcomes.

## Relationship to Hybrid Modeling
Large-load entries can include onsite generation and storage using the hybrid component fields. The supporting linkage is built in:
- `add_hybrid_plant_to_LFL!`

So the `demand` sheet is both:
- the large-load data surface
- the onsite hybrid-load surface

## Outputs To Inspect
The GTEP operation outputs include a dedicated demand-response result family with fields such as:
- `UNITGROUP`
- `UNIT_CATEGORY`
- `UNIT_TYPE`
- `CAP`
- `INTERCON_LIM`
- `Daily_DR_Limit_MWh`
- `Num_DR_Segments`
- `lfl_g_G_LFL_lt`
- `lfl_g_G_Grid_lt`
- `lfl_g_G_ES_lt`
- `lfl_g_ES_LFL_lt`
- `lfl_g_ES_Grid_lt`
- `lfl_chg_Grid_ES_lt`
- `lfl_soc_lt`

See:
- [Operation Outputs](./Operation/Operation_Outputs.md)

## Practical Reading Guide
When reviewing a large-load or demand-response entry, check it in this order:
1. the row in the network `demand` sheet
2. `CAP` and `INTERCON_LIM`
3. `Daily_DR_Limit_MWh` and `Num_DR_Segments`
4. segment shares `Pct_MW_1` to `Pct_MW_5`
5. segment prices `Price_1` to `Price_5`
6. any onsite hybrid fields

## Related Documentation
- [Network Data Reference](./Network_Data_Reference.md)
- [Hybrid Resources Reference](./Hybrid_Resources_Reference.md)
- [Operation Outputs](../models/Operation/Operation_Outputs.md)
