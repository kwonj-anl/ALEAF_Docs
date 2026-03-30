# Hybrid Resources Reference

This page explains how hybrid resources are represented in A-LEAF using the actual `hybrid` sheet in the network workbook and the explicit hybrid constraints used in the expansion and operation models.

In the current implementation, a hybrid resource is modeled as a linked pair:
- a main generating component
- an energy-storage component

The hybrid workbook row provides the shared linkage and interconnection information, while component technology definitions are pulled from `Gen Technology` through `UNITGROUP`.

## Data Surface: `hybrid` Sheet
The base network workbook includes a dedicated `hybrid` sheet. The actual fields include:
- `PLANT_NAME`
- `bus_ID`
- `bus_name`
- `RetireYear`
- `Online_Year`
- `PLANT_ORIS_ID`
- `UNITGROUP`
- `UNIT_CATEGORY`
- `UNIT_TYPE`
- `CAP`
- `INTERCON_LIM`
- `PMAX`
- `PMIN`
- `FOR`
- `Hybrid_Gen`
- `Hybrid_Gen_CAP`
- `Hybrid_ES`
- `Hybrid_ES_CAP`
- `Grid_Charge`

The fields that matter most structurally are:
- `Hybrid_Gen`
- `Hybrid_Gen_CAP`
- `Hybrid_ES`
- `Hybrid_ES_CAP`
- `INTERCON_LIM`
- `Grid_Charge`

These define which generator technology and storage technology belong to the hybrid, how large each component is, and whether the storage side can charge from the grid.

## How The Code Builds A Hybrid Resource
Hybrid processing is explicit in [generate_network.jl](C:/ALEAF/a-leaf/src/network/generate_network.jl):
- `update_hybrid_plant_technology_data!`
- `add_hybrid_plant_to_LFL!`

The workflow is:
1. find the component technology rows in `gen_technology` by matching `UNITGROUP`
2. copy those technology rows into hybrid component records
3. overwrite component capacities with `Hybrid_Gen_CAP` and `Hybrid_ES_CAP`
4. add separate plant entries for the generator side and storage side
5. mark those plant entries with:
- `hybrid_type = "GEN"` or `hybrid_type = "ES"`
- `hybrid_ID`

This means a hybrid is not treated as one opaque plant. It is expanded into linked model objects with shared metadata.

## Important implementation details

## Separate generator and storage members
The network build creates separate plant entries for:
- the hybrid generator member
- the hybrid storage member

The storage member also gets:
- `Charge_CAP`
- `ES_MWh`

with `ES_MWh` computed from the component charge capacity and duration.

## Interconnection is shared
The `INTERCON_LIM` field on the hybrid row is used in the hybrid interconnection constraints. This is the mechanism that couples the generator and storage sides at the site level.

## Grid charging is optional
The `Grid_Charge` field is read directly by:
- `constraint_hybrid_ES_Charge_from_Grid_idhty`

If `Grid_Charge` is false, the storage side cannot freely charge from the grid.

## Expansion and Operation Constraints
The hybrid formulation is explicit in the model builders and [constraints.jl](C:/ALEAF/a-leaf/src/component/constraints.jl).

The key hybrid constraint families are:
- `constraint_hybrid_ES_Charge_from_Grid_idhty`
- `constraint_hybrid_ES_Charge_Max_idhty`
- `constraint_hybrid_ES_Charge_Max_Sto_UC_idhty`
- `constraint_hybrid_ES_SOC_Balance_Inter_Hour_idhty`
- `constraint_hybrid_ES_SOC_Neutral_idy`
- `constraint_hybrid_onsite_gen_thermal_cap_ldhty`
- `constraint_hybrid_inter_connection_limit_injection_ldhty`
- `constraint_hybrid_inter_connection_limit_withdraw_ldhty`

These are called directly from the expansion and operation model construction in [LCO_GTEP.jl](C:/ALEAF/a-leaf/src/model/LCO_GTEP.jl).

What they mean at a high level:
- the hybrid ES charge constraints limit how the storage side charges
- the hybrid ES SOC constraints track storage state over time
- the onsite generation constraint limits how hybrid generation contributes
- the interconnection constraints enforce a shared site-level injection and withdrawal limit

## Relationship to Time Series
Hybrid generation is currently treated like PV in the fixed-profile renewable balance logic. In [constraints.jl](C:/ALEAF/a-leaf/src/component/constraints.jl), hybrid resources follow the `pv_shape` path.

That is an important modeling simplification:
- hybrid generation is not given a separate time-series category
- it is currently mapped through the PV-style profile logic

## Relationship to Large Flexible Load
The code also supports hybrid-style onsite generation and storage attached to large flexible load entries. That linkage is built through:
- `add_hybrid_plant_to_LFL!`

So hybrid logic is used in two places:
- the dedicated `hybrid` plant sheet
- hybrid-enabled large-load entries in the `demand` sheet

## Outputs To Inspect
The operation reporting path includes hybrid fields such as:
- `hybrid_chg_idhty`
- `hybrid_type`
- `Hybrid_Gen`
- `Hybrid_Gen_CAP`
- `Hybrid_ES`
- `Hybrid_ES_CAP`

See:
- [Operation Outputs](./Operation/Operation_Outputs.md)

## Practical Reading Guide
When checking a hybrid resource, inspect it in this order:
1. the row in the network `hybrid` sheet
2. the matching `UNITGROUP` rows in `Gen Technology`
3. `Hybrid_Gen_CAP` and `Hybrid_ES_CAP`
4. `INTERCON_LIM`
5. `Grid_Charge`

## Related Documentation
- [Network Data Reference](./Network_Data_Reference.md)
- [Gen Technology Reference](./Gen_Technology_Reference.md)
- [Operation Outputs](../models/Operation/Operation_Outputs.md)
- [RA Execution and Results](../models/RA/RA_Execution_and_Results.md)
