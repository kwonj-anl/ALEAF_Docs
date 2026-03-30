# Network Configuration Reference

This page explains the optional network configuration workbook used by A-LEAF and, in particular, the role of the `Network Setting` sheet.

The most important idea is:
- the base network workbook defines the physical system
- the config workbook defines how that system is grouped, bounded, and interpreted for modeling

The `Network Setting` sheet is the control layer for that process. It tells A-LEAF:
- what geographic footprint to model
- what spatial resolution to use for the network itself
- what spatial resolution to use for reserve zones, policy zones, and regional overlays

## When This Workbook Is Used
The config workbook is optional. It is used only when `Network_Configuration_File_ID` is set to a real value in `Simulation Configuration`.

If the config workbook is missing, blank, or `NA`, A-LEAF:
- keeps the original network topology
- keeps buses at their original resolution
- builds one system-wide planning reserve zone
- builds one system-wide reserve zone
- builds one system-wide policy zone
- uses neutral system-wide defaults for cost scaling and capacity credit

So this workbook is not required for every run, but it is required whenever the study needs flexible network construction.

## What `Network Setting` Controls

The `Network Setting` sheet is a compact set of switches that determine how the physical network is turned into the modeled network.

In the `config_USA_Base.xlsx` reference file, the sheet includes these major groups:

### Regional model setting
- `network_boundary_type`
- `network_boundary_level`
- `regional_aggregation_resolution_type`
- `regional_aggregation_resolution_level`
- `subregional_aggregation_resolution_type`
- `subregional_aggregation_resolution_level`

### Reserve setting
- `planning_reserve_zone_boundary_type`
- `planning_reserve_zone_boundary_level`
- `operating_reserve_zone_boundary_type`
- `operating_reserve_zone_boundary_level`
- `operating_reserve_data_resolution_type`
- `operating_reserve_data_resolution_level`

### Policy setting
- `policy_zone_boundary_type`
- `policy_zone_boundary_level`
- `policy_data_resolution_type`
- `policy_data_resolution_level`

### Additional regional overlays
- `regional_fuel_zone_resolution_type`
- `regional_fuel_zone_resolution_level`
- `regional_resource_cost_scaling_resolution_type`
- `regional_resource_cost_scaling_resolution_level`
- `regional_capacity_credits_resolution_type`
- `regional_capacity_credits_resolution_level`
- `regional_resource_supply_curve_resolution_type`
- `regional_resource_supply_curve_resolution_level`

### Build flags
- `generate_networkdata_flag`
- `network_reduction_flag`

These settings do not contain the mapping data themselves. Instead, they tell A-LEAF which parts of the other config sheets to use.

## Relationship With Other Sheets In The Config Workbook

`Network Setting` only works because the other config sheets provide the actual geographic hierarchy and regional records.

### `network_resolution_level`
This sheet defines the ordered hierarchy of spatial levels.

In `config_USA_Base.xlsx`, the hierarchy is:
- Level 1 = `BA`
- Level 2 = `State`
- Level 3 = `ISO`
- Level 4 = `Interconnection`
- Level 5 = `Country`

This sheet matters because the numeric `*_level` fields in `Network Setting` point into this hierarchy.

For example:
- `network_boundary_level = 3`
- `regional_aggregation_resolution_level = 3`

means A-LEAF should use the level-3 records, which correspond to `ISO`.

### `sub_area_mapping`
This sheet is the translation table between spatial levels.

In the USA config, each row maps one fine-resolution area across the hierarchy, for example:
- `BA`
- `State`
- `ISO`
- `Interconnection`
- `Country`

This table is used repeatedly in the code to:
- remap original buses into aggregated buses
- determine the region membership of each aggregated bus
- identify which data region belongs to a reserve or policy zone
- recover all lower-level areas that roll up into a higher-level zone

This is the key sheet that makes the network configurable rather than hard-coded.

### `sub_area_list`
This sheet defines the list of sub-areas that can become modeled buses after aggregation.

It works together with:
- `regional_aggregation_resolution_type`
- `subregional_aggregation_resolution_type`
- `sub_area_mapping`

to build the final aggregated bus list.

### `Network Data Level <n>`
These sheets contain the regional records for each spatial level.

For example, `Network Data Level 3` in the USA config represents ISO-level records and includes fields such as:
- `Region_ID`
- `Region_Name`
- `Model`
- `Subregional_Aggregation`
- `RA_ELCC_Calculation_Flag`
- regional resource limits
- regional capacity credits
- regional CAPAX scaling

These sheets serve two purposes:
1. they define which regions are actually modeled at a given level
2. they store regional overlays tied to that level

This is why `Network Setting` uses both a `type` and a `level`. The level selects which `Network Data Level <n>` sheet to read, and the type tells A-LEAF which geographic field from the hierarchy to interpret at that level.

### `RPS`, `CEGT`, and `CERT`
These sheets provide regional policy targets.

`Network Setting` determines:
- what the policy-zone boundaries are
- what level policy data should be read at

Then these sheets provide the values for those zones.

## How Flexible Network Construction Works

The flexible construction logic can be understood as a sequence:

1. Start from the physical network in the base workbook.
2. Use `network_boundary_type` and `network_boundary_level` to decide what geographic footprint is in the modeled system.
3. Use `regional_aggregation_resolution_type` and `regional_aggregation_resolution_level` to decide the main modeled network resolution.
4. Use `subregional_aggregation_resolution_type` and `subregional_aggregation_resolution_level` to define the lower-level grouping that rolls up into each modeled bus.
5. Use `sub_area_mapping` to connect those levels to the original physical areas.
6. Use `Network Data Level <n>` sheets to decide which regions are modeled and what regional overlay data applies.
7. Build separate zone structures for:
- planning reserve
- operating reserve
- policy
- resource supply curves
- cost scaling
- capacity credit

The result is that the same physical network workbook can support different modeled systems simply by changing the config workbook selections.

## How Different Layers Can Use Different Resolutions

One important feature of `Network Setting` is that the network, reserve zones, policy zones, and overlay data do not all need to use the same geography.

For example, the code allows:
- the modeled network to be aggregated at ISO level
- operating reserve requirements to be read at BA level
- policy zones to be enforced at State level
- capacity credit to be looked up at ISO level
- cost scaling to be applied at State level

This flexibility is why the config workbook has separate controls for:
- network aggregation
- reserve boundaries and reserve data
- policy boundaries and policy data
- fuel, cost, supply-curve, and capacity-credit resolution

## Concrete Example: `config_USA_Base.xlsx`

The USA base config provides a good example of how the pieces fit together.

### Example settings from `Network Setting`
- `network_boundary_type = ISO`
- `network_boundary_level = 3`
- `regional_aggregation_resolution_type = ISO`
- `regional_aggregation_resolution_level = 3`
- `subregional_aggregation_resolution_type = ISO`
- `subregional_aggregation_resolution_level = 3`
- `planning_reserve_zone_boundary_type = ISO`
- `operating_reserve_zone_boundary_type = ISO`
- `operating_reserve_data_resolution_type = BA`
- `policy_zone_boundary_type = State`
- `policy_data_resolution_type = State`
- `regional_capacity_credits_resolution_type = ISO`
- `regional_resource_cost_scaling_resolution_type = State`

### What this means in practice
- The modeled network footprint is defined using ISO-level regional records.
- The network itself is aggregated at ISO level.
- Planning reserve zones are ISO-level.
- Operating reserve zones are ISO-level, but the reserve requirement data can still be tied to BA-level data regions.
- Policy targets are enforced at State level, not ISO level.
- Capacity credit is looked up at ISO level.
- Cost scaling is applied at State level.

So even though the physical network comes from one base workbook, the config workbook tells A-LEAF to combine:
- ISO-level network representation
- BA-level reserve data mapping
- State-level policy structure
- ISO-level capacity-credit structure

That is the core of flexible network construction in A-LEAF.

## Practical Interpretation Rules

When reading `Network Setting`, it helps to interpret each pair of fields this way:

- `*_type` answers:
  what geographic label is being used, such as `BA`, `State`, `ISO`, or `Interconnection`?

- `*_level` answers:
  which `Network Data Level <n>` sheet and hierarchy level provide the records for that choice?

Then ask:
- what is being controlled by this pair?
  network boundary, aggregation, reserve, policy, cost scaling, capacity credit, or resource supply curve

## What To Check First When Debugging

If the network configuration does not behave as expected, check in this order:

1. whether `Network_Configuration_File_ID` is actually set in `Simulation Configuration`
2. whether the selected config workbook exists
3. whether the `network_resolution_level` hierarchy matches the intended geography
4. whether `sub_area_mapping` contains the needed cross-level mappings
5. whether the right `Network Data Level <n>` sheets contain modeled regions with `Model = true`
6. whether the reserve, policy, and overlay resolution fields are aligned with the intended study design

Common symptoms of a bad configuration include:
- the run collapsing to one system-wide zone
- policy targets being applied at the wrong geography
- missing or unexpected regional capacity-credit values
- resource-limit or cost-scaling behavior that does not match the intended regional design

## Related Documentation
- [Network Data Reference](./Network_Data_Reference.md)
- [ALEAF Simulation Setting File Reference](../configuration/ALEAF_Simulation_Setting_File.md)
- [Simulation Configuration Reference](../configuration/Simulation_Configuration_Reference.md)
- [A-LEAF Documentation](../README.md)
