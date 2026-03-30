# Network Data Reference

## Purpose
This page explains the two network-related input workbooks used by A-LEAF:
- the base network data workbook
- the optional network configuration workbook

The base network workbook defines the physical system that is being modeled. The optional configuration workbook defines how that physical system is grouped, aggregated, and mapped into planning, reserve, policy, and resource zones.

## Required vs Optional Inputs

### Base network data workbook
The base network data workbook is required for normal A-LEAF runs. It is selected through `Network_Data_File_ID` in the `Simulation Configuration` sheet and is resolved to a file named:

`network_<test_system_name>_<Network_Data_File_ID>.xlsx`

This workbook provides the core power-system data that all model families use.

### Network configuration workbook
The network configuration workbook is optional. It is selected through `Network_Configuration_File_ID` in the `Simulation Configuration` sheet and is resolved to a file named:

`config_<test_system_name>_<Network_Configuration_File_ID>.xlsx`

If this field is blank, missing, or `NA`, A-LEAF runs without a configuration workbook.

This is an important distinction:
- the base network workbook is the required physical-system input
- the config workbook is an optional overlay for aggregation, zoning, and region mapping

## How A-LEAF Uses These Files

### Without a configuration workbook
If no config workbook is provided, A-LEAF keeps the original network topology and synthesizes minimal region metadata. In that fallback mode:
- each bus remains at its original resolution
- plants, hybrids, and demand stay mapped to their original buses
- the whole system is treated as one planning reserve region, one reserve region, and one policy region
- resource, cost-scaling, and capacity-credit lookups fall back to neutral system-wide defaults

This means a config workbook is not required to run the model, but it is required if the study needs:
- regional aggregation
- multiple planning or reserve zones
- policy-region mapping
- regional resource supply curves
- regional capacity-credit data
- other zone-level overlays

### With a configuration workbook
If a config workbook is provided, A-LEAF uses it to:
- define the network boundary being modeled
- map original buses into aggregated regions
- create planning, reserve, and policy zones
- attach regional policy targets and resource-supply information
- define the regional data areas used for time-series aggregation and capacity-credit lookup

In other words, the base workbook defines the physical system, while the config workbook defines how that system is interpreted geographically and institutionally.

## 1. Base Network Data Workbook

The base network workbook provides the main physical-system tables. In the current code path, A-LEAF reads these sheets directly:
- `bus`
- `plant`
- `branch`
- `demand`
- `hybrid`

### `bus`
Defines the network nodes and their core attributes.

Typical contents include:
- bus identifiers
- load at each bus
- bus names
- basic location or mapping fields used later in aggregation

This sheet is the starting point for both physical topology and zonal aggregation.

### `plant`
Defines existing generation and storage plants already in the system.

Typical contents include:
- plant identifiers
- bus assignment
- capacity
- technology labels
- plant-specific overrides

Plants with zero capacity are removed during network-data preparation.

For storage-specific behavior grounded in the `plant` sheet and the storage constraints, see [Storage Modeling Reference](./Storage_Modeling_Reference.md).

### `branch`
Defines transmission elements between buses.

Typical contents include:
- from and to buses
- electrical parameters
- thermal limits
- cost fields used in transmission-related workflows

This sheet is used for both operations and expansion-planning transmission representations.

### `demand`
Defines flexible or special demand-side objects that are represented separately from fixed bus load.

This is used in workflows that distinguish between base load and modeled demand-side resources.

In the current database and code path, this is also the dedicated input surface for large flexible load and demand-response entries. See [Large Load and Demand Response Reference](./Large_Load_and_Demand_Response_Reference.md).

### `hybrid`
Defines hybrid plant structures and their component mapping.

This sheet is used to build linked generator-storage representations and to add hybrid components into the plant-level model structures.

See [Hybrid Resources Reference](./Hybrid_Resources_Reference.md) for the dedicated hybrid fields and the associated model constraints.

## 2. Optional Network Configuration Workbook

The optional configuration workbook provides the region-mapping and zoning layer that sits on top of the physical network.

For a detailed explanation of how `Network Setting` drives flexible network construction and how it works with the other config sheets, see [Network Configuration Reference](./Network_Configuration_Reference.md).

In the current code path, A-LEAF reads these key sheets from the config workbook:
- `Network Setting`
- `sub_area_list`
- `network_resolution_level`
- `sub_area_mapping`
- `Network Data Level <n>` sheets
- `CEGT`
- `CERT`
- `RPS`

### `Network Setting`
Defines the main configuration choices for the network overlay.

Important concepts include:
- network boundary type and level
- regional aggregation resolution
- subregional aggregation resolution
- regional mapping behavior

This sheet drives the high-level logic for how the physical network is grouped.

It is the control layer for:
- network boundary selection
- modeled network aggregation
- reserve-zone construction
- policy-zone construction
- regional overlay resolution for supply curves, cost scaling, and capacity credit

### `sub_area_list`
Defines the list of sub-areas that can appear in the configured network representation.

This sheet is used when building the aggregated bus structure.

### `network_resolution_level`
Defines the hierarchy of spatial resolution levels used in the model.

This helps A-LEAF interpret relationships such as:
- original bus or county level
- balancing area level
- ISO level
- state level

### `sub_area_mapping`
Defines the mapping between original network areas and the configured regional hierarchy.

This is one of the most important configuration tables because it allows A-LEAF to:
- remap buses into aggregated regions
- determine regional membership
- identify the data region used for time-series and policy lookup
- create planning, reserve, and policy zones

### `Network Data Level <n>`
These sheets provide boundary and region records for each configured network level.

They are used when selecting:
- what part of the network is in scope
- which regions are modeled
- how aggregation boundaries are interpreted

### `CEGT`, `CERT`, and `RPS`
These sheets provide policy-target data tied to the configured regional structure.

They are used to build policy-zone targets for:
- clean energy generation targets
- carbon-emission reduction targets
- renewable portfolio standards

## 3. Zonal Structures Built From Network Inputs

Once the physical network and optional config data are loaded, A-LEAF constructs several zonal structures under `network_data["zone"]`.

The most important zone families are:
- `planning_reserve`
- `reserve`
- `policy`
- `resource_supply_curve`
- `cost_scaling`
- `capacity_credit`

These are important because they are where the optional config workbook has the largest downstream effect.

### Planning reserve zones
Used in planning-reserve and adequacy-style constraints.

### Reserve zones
Used in operating-reserve formulations.

### Policy zones
Used for policy targets such as RPS, clean-energy, and carbon-related tracking.

### Resource supply curve zones
Used to define regional build limits and supply curves.

### Cost scaling zones
Used to apply locational scaling to selected investment costs.

### Capacity credit zones
Used when capacity credit is stored regionally and technologies reference a zonal lookup instead of a literal value.

## 4. Relationship to Time-Series Data

The network inputs and the time-series inputs are tightly connected.

The network structure determines:
- which buses or regions receive load data
- how renewable data is aggregated
- which regional identifiers are used for time-series allocation

In the configured-network path, the config workbook helps determine the data-region mapping used to:
- aggregate local renewable shapes
- map policy and adequacy data
- connect zonal structures to timeseries tags or regional IDs

## 5. Relationship to Model Families

All major A-LEAF model families use the shared network data foundation:

- Expansion planning uses the network topology, zonal mappings, technology overlays, and representative-day data to build planning models.
- Production cost simulation uses the same network and zone structures to build operational dispatch and reserve formulations.
- Reliability assessment reuses the generated network state and then adds outage and renewable-uncertainty logic on top of it.

Because the same network-data build is reused across model families, changes to the base network workbook or config workbook can affect all downstream models.

## Practical Reading Guide

When reviewing a case, it is useful to check the network inputs in this order:
1. confirm `Network_Data_File_ID`
2. confirm whether `Network_Configuration_File_ID` is blank, `NA`, or a real config id
3. if a config workbook is used, check `Network Setting`, `sub_area_mapping`, and the relevant `Network Data Level` sheet first
4. then inspect how that structure interacts with policy, reserve, and resource-limit settings

If a run behaves as one single system-wide region when you expected multiple zones, the first thing to check is whether a valid network configuration workbook was actually selected.

## Related Documentation
- [A-LEAF Documentation](../README.md)
- [ALEAF Simulation Setting File Reference](../configuration/ALEAF_Simulation_Setting_File.md)
- [Network Configuration Reference](./Network_Configuration_Reference.md)
- [Simulation Configuration Reference](../configuration/Simulation_Configuration_Reference.md)
- [Gen Technology Reference](./Gen_Technology_Reference.md)
- [Storage Modeling Reference](./Storage_Modeling_Reference.md)
- [Hybrid Resources Reference](./Hybrid_Resources_Reference.md)
- [Large Load and Demand Response Reference](./Large_Load_and_Demand_Response_Reference.md)
