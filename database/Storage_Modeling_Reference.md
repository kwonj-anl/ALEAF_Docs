# Storage Modeling Reference

This page explains how storage is represented in A-LEAF using the actual workbook surfaces and the storage constraints used in the expansion and operation models.

Storage in A-LEAF is not driven by a single sheet. It is assembled from:
- existing storage records in the network data `plant` sheet
- candidate storage definitions in `Gen Technology`
- storage cost and performance assumptions in `Storage Cost and Performance`
- case-level switches in `Simulation Configuration`

## Data Surfaces

## Existing storage in the network workbook
Existing storage enters the model through the `plant` sheet in the base network workbook, such as:
- `CAP`
- `Charge_CAP`
- `ES_MWh`
- `STOMIN`
- `STOHR_MIN`
- `STOHR_MAX`
- `PMAX`
- `PMIN`
- `FOR`

These fields define the installed power and energy capability of an existing storage plant. In the code path, the network build keeps both power and energy information at the plant level, and `ES_MWh` is carried forward into the model data as the storage energy limit.

Important implementation detail:
- `ES_MWh` is the direct stored-energy quantity for existing storage.
- `STOHR_MAX` is the storage duration field used repeatedly in both planning and RA logic.

## Candidate storage in `Gen Technology`
Candidate storage technologies are defined in `Gen Technology`. The most important storage-related fields are:
- `UNITGROUP`
- `UNIT_CATEGORY`
- `UNIT_TYPE`
- `Charge_CAP`
- `STOMIN`
- `STOHR_MIN`
- `STOHR_MAX`
- `CAPEX`
- `STO_CAPEX`
- `FOM`
- `VOM`
- `FCR`
- `AET`

These fields tell the expansion model:
- whether the technology is storage
- how much charging power it has
- what duration and state-of-charge bounds it uses
- how investment cost and annual throughput are represented

## `Storage Cost and Performance`
The `Storage Cost and Performance` sheet provides storage-specific assumptions keyed by:
- `ESGC_Setting_ID`
- `UNITGROUP`

The actual sheet includes fields such as:
- `Capacity`
- `Duration`
- `RTE`
- `Cycle_Life`
- `Calendar_Life`
- `AET`
- `FOM`
- `VOM`
- year-by-year capital cost columns

In the network build, ESGC-linked storage fields are resolved in `generate_network.jl`. In particular:
- `AET` can be populated from ESGC data
- other storage cost and performance values can also be pulled from ESGC when the technology row uses external references

## Case-level settings that matter
The most important case-level storage controls live in `Simulation Configuration`:
- `storage initialization option`
- `Energy_Stroage_AET_Limit_Flag`
- time-step and representative-day settings that affect intertemporal storage behavior

These settings affect how storage SOC is initialized and whether annual energy throughput limits are enforced.

## How Storage Is Built In The Model

## Network and data preparation
In the shared network build:
- existing storage plants keep explicit `ES_MWh` energy capacity
- candidate storage technologies carry duration and charging data from `Gen Technology`
- ESGC-linked `AET` values are resolved before model construction

In `generate_network.jl`, plant-level storage values are also scaled into the internal per-unit representation, including:
- `ES_MWh`
- `AET`

## Expansion model representation
In the expansion path, storage appears as both investment and operation decisions.

The code explicitly uses:
- `u_new_ESH_iy` for new storage-energy duration investment
- `ES_MWh` for existing stored-energy capability

The expansion model then enforces storage feasibility through these constraint families in [constraints.jl](C:/ALEAF/a-leaf/src/component/constraints.jl):
- `constraint_ES_Charge_Max_idhty`
- `constraint_ES_DisCharge_Max_Sto_UC_idhty`
- `constraint_ES_SOC_Max_idhty`
- `constraint_ES_SOC_Min_idhty`
- `constraint_ES_SOC_Balance_Inter_Hour_idhty`
- `constraint_ES_SOC_Balance_Inter_SubHour_idhty`
- `constraint_ES_SOC_Neutral_idy`
- `constraint_ES_AET_y`

These are called from the expansion and operation model builders in [LCO_GTEP.jl](C:/ALEAF/a-leaf/src/model/LCO_GTEP.jl).

What they mean at a high level:
- charge and discharge limits enforce instantaneous power capability
- SOC min/max enforce energy bounds
- SOC balance constraints link hours and sub-hours
- SOC neutral constraints prevent representative-day storage from creating artificial net energy across a day-group cycle
- AET constraints cap annual energy throughput when the case flag is on

## Operation model representation
The operation model uses the same core storage feasibility logic, but with the operation-specific throughput constraint:
- `constraint_ES_AET_OP_y`

The operation builder also calls the same charge, discharge, SOC, and SOC-neutral constraint families as above, so the storage dispatch logic is consistent between planning and operation runs.

This is why storage behavior depends heavily on:
- representative days
- `NumDays` weighting
- `storage initialization option`

## Important storage assumptions

## SOC initialization
The code supports several initial-SOC assumptions controlled by:
- `storage initialization option`

The constraint logic explicitly checks for:
- `Minimum`
- `Middle`
- `Maximum`

That setting changes the initial energy state used in the SOC balance equations.

## Annual energy throughput
If `Energy_Stroage_AET_Limit_Flag` is true, the model enforces annual throughput using:
- `constraint_ES_AET_y` in expansion
- `constraint_ES_AET_OP_y` in operation

The throughput calculation uses representative-day scaling through `NumDays`, so it is an annualized limit rather than a one-day limit.

## Existing energy vs new energy additions
The code distinguishes between:
- existing `ES_MWh`
- new storage energy represented through expansion decisions

That distinction matters because power additions and energy additions are not always identical in the expansion formulation.

## Outputs To Inspect
The main output docs already list the storage result fields in detail, but the most important operational storage outputs are:
- `chg_idhty`
- `soc_idhty`
- `u_ESE_iy`
- `sto_c_idhty` when storage commitment is active

See:
- [Operation Outputs](./Operation/Operation_Outputs.md)
- [GTEP Expansion Outputs](./GTEP/GTEP_Expansion_Outputs.md)

## Practical Reading Guide
When reviewing storage assumptions for a case, check them in this order:
1. existing storage rows in the network `plant` sheet
2. storage candidate rows in `Gen Technology`
3. the selected `ESGC_Setting_ID` and matching rows in `Storage Cost and Performance`
4. `storage initialization option`
5. `Energy_Stroage_AET_Limit_Flag`

## Related Documentation
- [Gen Technology Reference](./Gen_Technology_Reference.md)
- [Policy and Financial Settings](../configuration/Policy_and_Financial_Settings.md)
- [Network Data Reference](./Network_Data_Reference.md)
- [Operation Outputs](../models/Operation/Operation_Outputs.md)
- [GTEP Expansion Outputs](../models/GTEP/GTEP_Expansion_Outputs.md)
