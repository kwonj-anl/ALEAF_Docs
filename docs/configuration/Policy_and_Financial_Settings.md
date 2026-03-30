# Policy and Financial Settings

## Purpose
This page explains two shared input areas in A-LEAF:
- policy settings
- financial and technology-cost settings

These settings are cross-cutting. They affect expansion planning most directly, but parts of them also affect operation, RA, and linked workflows.

The main policy surfaces covered here are:
- `ITC`
- `PTC`
- `RPS`
- `CEGT`
- `CERT`

The main financial-setting surfaces covered here are:
- `ATB Setting`
- `Storage Cost and Performance`

## 1. Policy Settings

## Policy Settings Live In More Than One Place
Policy behavior in A-LEAF is not controlled by a single sheet.

It is assembled from three layers:
1. case-level flags and targets in `Simulation Configuration`
2. zone structure and policy-region mapping from the network configuration workbook
3. technology-level eligibility from `Gen Technology` plus year-value tables such as `ITC` and `PTC`

This is why policy documentation needs to explain how the sheets work together, not just what each sheet contains.

## `ITC` and `PTC`

### What they are
The `ITC` and `PTC` sheets provide year-by-year tax credit values by technology group.

In the code path, these sheets are loaded into technology data during network generation and then used when the selected case has tax credits enabled.

### How they are activated
Two conditions must both be true:
- the case-level flag in `Simulation Configuration` must be on
  - `ITC_Flag`
  - `PTC_Flag`
- the technology row in `Gen Technology` must be eligible
  - `ITC Flag`
  - `PTC Flag`

So the case-level flag turns the policy on for the case, while the technology-level flag decides whether a particular technology can receive it.

### How they are used in the code
`generate_network.jl` attaches `ITC` and `PTC` tables to each technology through `UNITGROUP` lookup.

Then:
- `ITC` is applied primarily as a CAPEX reduction
- `PTC` is applied through generation-based credits over a limited number of years

In `LCO_GTEP.jl`, both ITC and PTC appear in system-cost reporting and objective accounting.

### Important implementation detail
The code clips the tax-credit year lookup into the supported range, so if the modeled year is outside the stored table range, A-LEAF uses the nearest supported year boundary.

## `RPS`

### What it is
`RPS` is the renewable portfolio standard structure used for renewable energy share targets.

### How it is activated
The case-level controls live in `Simulation Configuration`:
- `RPS_Flag`
- `Allow_Alternative_RPS_Compliance_Flag`
- `RPS_Penalty`
- `RPS_Global_Target_Value`

### How it interacts with the network config
RPS targets are built over policy zones. Those policy zones come from the network configuration workbook through:
- `policy_zone_boundary_type`
- `policy_data_resolution_type`

So the same global study can enforce RPS at different regional levels depending on the network configuration.

### How it is used in the code
The network-generation path builds policy-zone targets under:
- `network_data["zone"]["policy"][...]["RPS"]`

Then the optimization model uses those zonal targets in regional RPS constraints.

If alternative compliance is allowed, RPS can use slack with a penalty rather than forcing strict feasibility.

## `CEGT`

### What it is
`CEGT` is the clean energy generation target.

### How it is activated
The case-level controls live in `Simulation Configuration`:
- `Clean_Energy_Generation_Target_EXP_Flag`
- `Clean_Energy_Generation_Target_EXP_Type`
- `Clean_Energy_Generation_Target_OP_Flag`
- `Clean_Energy_Generation_Target_Start_Year`
- `Clean_Energy_Generation_Global_Target_Value`
- `Clean_Energy_Generation_Penalty`

### How it is used
Like RPS, CEGT is built over policy zones from the network configuration.

In the code, CEGT can be enforced in:
- expansion mode
- operation mode

And in expansion mode it can be applied in different styles such as:
- annual
- daygroup

That distinction matters because representative-day weighting is used differently depending on the selected target type.

## `CERT`

### What it is
`CERT` is the carbon emission reduction target structure.

### How it is activated
The main case-level controls live in `Simulation Configuration`:
- `Carbon_Emission_Reduction_Target_Flag`
- `Carbon_Emission_Reduction_Global_Target_Value`
- `Carbon_Emission_Reduction_Target_Start_Year`

### How it interacts with the config workbook
Like other policy targets, CERT uses policy zones from the network configuration workbook.

The optional network config also provides the spatial structure used for:
- policy boundaries
- policy data resolution
- regional carbon-reference accounting

### Important implementation detail
The model tracks both:
- target values
- a reference emission level

That reference is stored in the zonal policy data and is needed because carbon-reduction targets are defined relative to a baseline, not just as an absolute cap.

## Summary of Policy Relationships

The policy settings work together as follows:
- `Simulation Configuration` turns policies on and sets target or penalty values
- the network configuration workbook defines the policy geography
- `Gen Technology` defines technology eligibility or policy participation
- `ITC` and `PTC` provide year-by-year tax-credit values
- the optimization models enforce or price the resulting policy requirements

## 2. Financial Settings

## Why These Settings Matter
The financial side of A-LEAF is not just a cost input table. It is a mapping layer that determines how technology assumptions are populated at runtime.

The two most important shared tables are:
- `ATB Setting`
- `Storage Cost and Performance`

## `ATB Setting`

### What it is
`ATB Setting` maps A-LEAF technology groups to NREL ATB assumptions.

The mapping is keyed by:
- `ATB_Setting_ID`
- `UNITGROUP`

and typically includes:
- `Case`
- `CRP`
- `Tech`
- `TechDetail`
- `Scenario`
- `ATB Year`
- `CAPEX_Scale`

### How it is activated
At the case level, `Simulation Configuration` selects:
- `ATB_Setting_ID`
- `ATB_Year`

Then the model uses `UNITGROUP` to find the matching ATB row for each technology.

### How it is used in the code
The ATB helper functions live in:
- [base_functions.jl](C:/ALEAF/a-leaf/src/core/base_functions.jl)

The main runtime use is in:
- [generate_network.jl](C:/ALEAF/a-leaf/src/network/generate_network.jl)

Depending on the technology row, ATB can supply values such as:
- CAPEX
- fixed O&M
- variable O&M
- fixed charge rate and CRF-related values

### Important implementation detail
ATB is not universally applied to every field automatically. The technology row still decides whether a field is:
- a literal value
- `ATB`
- another external reference such as `ESGC`

So `ATB Setting` is a mapping table, not a blanket replacement of all technology costs.

## `Storage Cost and Performance`

### What it is
`Storage Cost and Performance` is the storage cost and performance table used for ESGC-linked storage assumptions.

It is selected through:
- `ESGC_Setting_ID`

and matched by:
- `UNITGROUP`

### How it is used in the code
The network-generation path looks up ESGC-linked values using:
- selected `ESGC_Setting_ID`
- technology `UNITGROUP`

This affects both technology-level and plant-level storage assumptions.

### Important fields
The exact workbook columns can vary, but the code currently uses this sheet for fields such as:
- round-trip efficiency
- annual energy throughput
- storage CAPEX
- fixed O&M
- variable O&M

In the current implementation, ESGC-linked values are especially important for:
- `BATEFF`
- `AET`
- storage-specific CAPEX
- storage FOM and VOM

### Why this matters operationally
These values do not only affect investment cost. They also affect constraints:
- `BATEFF` affects state-of-charge transitions and reserve coupling
- `AET` affects annual throughput constraints when enabled

So this sheet influences both planning economics and operational/storage feasibility.

For the storage-specific data path from workbook inputs into expansion and operation constraints, see [Storage Modeling Reference](../database/Storage_Modeling_Reference.md).

## ATB and ESGC Together
ATB and ESGC are complementary, not identical.

In practice:
- ATB is the main mapping for broader technology cost assumptions
- ESGC is the main mapping for storage-specific cost and performance assumptions

The `Gen Technology` row decides which source is used for each field.

This means a storage technology may use:
- ATB for some values
- ESGC for others
- literal values for others

## Relationship to `Gen Technology`
The `Gen Technology` sheet is the local contract for whether a technology field is:
- direct input
- ATB-linked
- ESGC-linked
- fuel-linked

So the financial tables do not stand alone. They only matter because the technology definitions reference them.

## Practical Reading Guide
When checking policy and financial settings for a case, it is useful to inspect them in this order:

1. case-level switches in `Simulation Configuration`
2. network configuration for policy geography
3. technology-level eligibility and external-reference flags in `Gen Technology`
4. value tables:
- `ITC`
- `PTC`
- `ATB Setting`
- `Storage Cost and Performance`

If a result looks wrong, common things to check are:
- the case-level policy flag is off
- the technology is not policy-eligible
- the wrong `ATB_Setting_ID` or `ESGC_Setting_ID` was selected
- the `UNITGROUP` name does not match across sheets
- the policy geography is not the one the study intended

## Related Documentation
- [ALEAF Simulation Setting File Reference](../configuration/ALEAF_Simulation_Setting_File.md)
- [Simulation Configuration Reference](../configuration/Simulation_Configuration_Reference.md)
- [Network Configuration Reference](../database/Network_Configuration_Reference.md)
- [Gen Technology Reference](../database/Gen_Technology_Reference.md)
- [Storage Modeling Reference](../database/Storage_Modeling_Reference.md)
- [A-LEAF Documentation](../README.md)
