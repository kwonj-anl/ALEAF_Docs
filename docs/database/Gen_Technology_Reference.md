# Gen Technology Reference

The `Gen Technology` sheet is the master technology-definition table in A-LEAF. Each row describes one technology option, and the columns control how that technology is represented in:
- expansion planning
- production cost simulation
- reliability assessment
- ELCC and other post-processing workflows

This sheet is one of the most important input tables in the workbook because it defines technology identity, cost structure, operating behavior, reliability treatment, and investment options.

## How To Read The Sheet
The sheet is organized by technology row and grouped by column categories. The category headers in the workbook indicate the main logic blocks:
- Category
- Dispatch and Commitment Options
- Capacity and Energy
- Financial
- Technical Characteristics
- Market and Policy
- Reliability
- Investment Options
- Resource Limits
- ETC

In practice, the sheet should be read as a technology contract:
- the left side identifies what the technology is
- the middle columns define how it behaves operationally and financially
- the right side defines planning, policy, and reliability behavior

## Technology Identity
Each row represents one technology definition.

Important identity fields include:

### `Tech_ID`
Unique technology identifier used inside the workbook and model data structures.

### `UNITGROUP`
Core technology group name used throughout A-LEAF. This is one of the most important fields because it links the technology row to:
- ATB mappings
- storage cost and performance settings
- policy tables
- RA capacity credit and ELCC logic
- resource-limit logic

In practice, `UNITGROUP` is the main cross-sheet join key. Consistent naming here is required for technology costs, storage assumptions, policy lookup, and several RA/ELCC workflows to work correctly.

### `UNIT_CATEGORY`
High-level technology class, such as:
- thermal
- hydro
- storage
- nuclear
- renewable

This affects both model logic and reporting groupings.

### `UNIT_TYPE`
More specific technology type within the broader category.

### `FUEL`
Primary fuel or resource label used for cost, emissions, and policy mapping.

## Dispatch and Commitment Options
These fields control whether and how the technology participates in operational decision-making.

Typical parameters in this group include:
- commitment requirement
- minimum generation level
- startup and shutdown behavior
- reserve participation
- ramping representation
- charge/discharge treatment for storage

This group determines whether a technology behaves like:
- a fully dispatchable committed unit
- a simple dispatchable resource
- a variable renewable profile
- an energy-limited storage asset

## Capacity and Energy
These fields define the physical scale and energy characteristics of the technology.

Typical parameters include:
- nameplate capacity
- charge capacity
- minimum operating level
- storage duration
- energy volume limits

For storage technologies, this group works together with the `Storage Cost and Performance` sheet and the selected ESGC setting.

## Financial
These fields control investment and operating-cost assumptions.

Important concepts in this group include:
- CAPEX
- fixed O&M
- variable O&M
- fuel cost source
- startup and no-load cost
- cost values imported from `ATB` or fuel-price data

Important implementation detail:
- some cost fields are literal values
- some use external references such as `ATB`, `ESGC`, `Fuel`, `Fuel-Regional`, or `Water Value`

That means this sheet often defines whether a parameter is hard-coded locally or linked to another table.

## Technical Characteristics
These fields describe the engineering behavior of the technology.

Typical examples include:
- heat rate
- ramp rate
- minimum up/down-style operating restrictions
- round-trip efficiency
- annual energy throughput
- emissions factors
- lifetime

These parameters determine how the technology behaves in dispatch, commitment, planning, and emissions calculations.

## Market and Policy
This group controls how the technology interacts with policy and incentive settings.

Important examples include:
- ITC eligibility
- PTC eligibility
- market participation assumptions
- policy-related flags

These fields work with:
- `ITC`
- `PTC`
- `Simulation Configuration`

to determine whether a technology receives policy treatment in a particular case.

The case-level `ITC_Flag` and `PTC_Flag` enable the policy generally, while the technology-level flags determine whether a specific technology is eligible to receive it.

## Reliability
These fields are especially important for RA and ELCC-related workflows.

Important examples include:
- forced outage rate or reliability parameters such as `RA_FOR`
- capacity credit definition such as `CAPCRED`
- ELCC participation flags such as `ELCC_Flag`

### `CAPCRED`
This field is used to define capacity accreditation logic. In practice it may be:
- a direct numeric value
- a string key that maps to a capacity-credit lookup in the network data

Both planning and RA read this field. When it is a string, the model resolves it through regional capacity-credit data before calculating UCAP.

### `ELCC_Flag`
Controls whether the technology is eligible for ELCC-related workflows.

### `RA_FOR`
Used in the reliability workflow to determine the forced outage representation for the technology in RA.

This reliability block is one of the key links between the technology table and the RA model.

## Investment Options
These fields control whether and how the technology can be built, retired, or expanded.

Typical examples include:
- minimum and maximum investment
- integrality
- retirement eligibility
- expansion limits

These parameters are central to expansion planning because they define whether the model can invest in the technology and under what bounds.

## Resource Limits
These fields connect technologies to regional or system-level build constraints.

Important examples include:
- `Resource_Limit_Flag`
- `Resource_Limit_ID`
- locational scaling flags

These fields work together with:
- regional resource-limit data
- simulation-configuration resource-limit settings

to restrict where and how much of a technology can be built.

## Interaction With Other Sheets
The `Gen Technology` sheet is heavily connected to the rest of the workbook.

### `ATB Setting`
Maps `UNITGROUP` values to ATB technology assumptions, cost cases, and ATB years.

ATB-linked technologies can pull CAPEX, FOM, and in some cases VOM from the selected ATB configuration.

### `Storage Cost and Performance`
Provides storage-specific cost and performance data, usually keyed by `UNITGROUP` and ESGC setting.

This is also where ESGC-linked fields such as storage cost, `BATEFF`, and `AET` can be sourced when the technology row uses `ESGC` as the reference value.

### `ITC` and `PTC`
Provide year-specific tax credit values for eligible technologies.

### `Simulation Configuration`
Controls which ATB setting, ESGC setting, policy flags, and case options apply to the technologies in a specific run.

### `RA Setting` and `RA Scenarios`
Use reliability and ELCC-related fields in this sheet to determine which technologies participate in RA, ELCC, and renewable adequacy treatment.

## Why `UNITGROUP` Matters
`UNITGROUP` is one of the most important fields in the entire workbook because it acts as the common join key across many tables. If a technology is not named consistently across workbook sheets, it can break:
- ATB mapping
- storage-performance lookup
- ITC/PTC policy application
- ELCC eligibility
- resource-limit logic
- RA renewable and capacity-credit logic

## Practical Reading Guide
When reviewing a technology row, it is useful to check the fields in this order:
1. identity: `Tech_ID`, `UNITGROUP`, `UNIT_CATEGORY`, `UNIT_TYPE`, `FUEL`
2. operational behavior: dispatch, commitment, ramping, and reserve participation
3. physical behavior: capacity, energy, efficiency, emissions
4. cost sources: literal values vs external references such as `ATB` or `Fuel`
5. reliability and policy fields
6. investment and resource-limit controls

## What Should Be Documented Next
This page documents the sheet at the column-group level. The next useful layer, if needed later, would be:
- a more detailed field-by-field reference for the highest-impact columns
- a focused reference for storage technologies
- a focused reference for policy-eligible renewable technologies

## Related Documentation
- [ALEAF Simulation Setting File Reference](../configuration/ALEAF_Simulation_Setting_File.md)
- [Simulation Configuration Reference](../configuration/Simulation_Configuration_Reference.md)
- [RA Overview](../models/RA/RA_Overview.md)
