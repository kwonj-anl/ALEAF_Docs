# Scenario Reduction and Representative-Day Groups

## Purpose
This page explains how A-LEAF reduces full-year time series into representative days and representative-day groups.

This is a shared concept across the framework:
- GTEP uses representative days to reduce model size
- operation workflows use representative days and day groups for dispatch and reporting
- RA reuses the representative-day structure and maps uncertainty inputs back onto it

The goal of this page is to explain:
- what scenario reduction produces
- how representative days differ from representative-day groups
- how `Reference_Day` is used
- how the reduced structure is consumed by downstream models

## High-Level Idea
Scenario reduction starts from full-year time series and selects a reduced set of days that stand in for the full chronology.

In A-LEAF, the reduced structure has two layers:
- `repdays`: the representative days actually used inside the model
- `repday_groups`: grouped day windows that tie representative days together and preserve grouped-day logic

This distinction matters because:
- many constraints and reports are weighted at the representative-day level
- some workflows, especially grouped-day operation and RA, need to know which representative days belong to the same day group

## How Representative Days Are Created

### 1. Full-year data are prepared for reduction
In `LCO_GTEP.jl`, A-LEAF writes temporary time-series files for scenario reduction using:
- load
- wind
- PV

Both MW values and shape-based variants are prepared, depending on the reduction settings.

### 2. Scenario reduction is run
The reduction logic lives in:
- [scenario_reduction_GTEP.jl](C:/ALEAF/a-leaf/src/util/scenario_reduction_GTEP.jl)

The core workflow produces a reduced day set with:
- selected scenarios or day windows
- probabilities
- grouped-day structure

### 3. The reduced output is loaded into `network_data`
During network generation, A-LEAF either:
- reads a manual representative-day file, or
- runs scenario reduction and loads its output

That happens in:
- [generate_network.jl](C:/ALEAF/a-leaf/src/network/generate_network.jl)

The resulting structure is stored under:
- `network_data["repday_groups"]`
- `network_data["planning_stages"][stage]["repdays"]`

## Manual Representative Days vs Scenario Reduction

A-LEAF supports two main paths:

### Manual representative-day file
If a valid representative-day file is provided and matches the requested number of day groups and days per group, A-LEAF can load it directly.

### Scenario reduction
If the representative-day selection mode is `Scenario Reduction`, or if the manual representative-day input is missing or inconsistent, A-LEAF falls back to scenario reduction.

This is important because the workflow is resilient:
- a manual file can be used when the study wants a fixed reduced calendar
- scenario reduction can generate the structure dynamically when needed

## `repdays` vs `repday_groups`

### `repdays`
`repdays` are the individual representative days used directly in the model.

Each representative day stores fields such as:
- `Day_Group_ID`
- `Day_Group`
- `NumDays_Group`
- `NumDays`
- `Day`
- `Month`
- optionally `Scenario_ID` in stochastic workflows

The most important field for the original calendar mapping is:
- `Day`

This is the original full-year day number, and it is what the reporting code refers to as `Reference_Day`.

### `repday_groups`
`repday_groups` are the grouped windows that collect representative days into one logical day group.

Each group stores fields such as:
- `Start_Day_Id`
- `End_Day_Id`
- `Probability`
- `NumDays_Group`
- `Day_List`
- `Day_Idx_List`

The most important distinction is:
- `Day_List` stores the original calendar days represented by the group
- `Day_Idx_List` stores the representative-day ids inside the internal `repdays` dictionary

This distinction is critical in grouped-day workflows.

## What `Reference_Day` Means
In A-LEAF reporting, `Reference_Day` is the original full-year day number associated with a representative day.

The reporting functions make this explicit:
- [report_result_repdays_OP_GTEP](C:/ALEAF/a-leaf/src/model/LCO_GTEP.jl)
- [report_result_repdays_GTEP](C:/ALEAF/a-leaf/src/model/LCO_GTEP.jl)

In those reports:
- `RepDay_id` is the internal representative-day id
- `Reference_Day` comes from the representative-day field `Day`

This is why the model must distinguish:
- internal representative-day ids
- original calendar-day values

That distinction became especially important for RA, where renewable scenario caches needed to use `Reference_Day`, not just `Day_Idx_List`, to map 8760-hour traces correctly.

## How Weighting Works

### At the day-group level
Scenario reduction produces group probabilities. These are stored on `repday_groups`.

### At the representative-day level
During network-data construction, A-LEAF converts group probabilities into:
- `NumDays_Group`
- `NumDays`

The representative-day-level `NumDays` field is then used throughout the model as the weighting factor for:
- objective terms
- annualized metrics
- policy accounting
- scarcity and ENS reporting

This is why many code paths multiply by:
- `parameter(..., :repdays, "NumDays", d)`

rather than using day-group probability directly.

## How Multi-Day Day Groups Work
Representative-day groups can contain more than one day.

For example, a day group may cover three consecutive calendar days:
- day 4
- day 5
- day 6

In that case:
- `Day_List = [4, 5, 6]`
- `Day_Idx_List` contains the internal representative-day ids for those three days

The representative-day report would show something like:
- `RepDay_id = 1`, `Reference_Day = 4`
- `RepDay_id = 2`, `Reference_Day = 5`
- `RepDay_id = 3`, `Reference_Day = 6`

All three repdays belong to the same `Daygroup_ID`.

This is the pattern you saw in the RA work:
- one day group can contain several representative days
- each representative day still points back to one original calendar day

## How GTEP Uses The Reduced Structure
GTEP uses representative days directly in the optimization model.

The key uses are:
- time-series data are attached to representative days
- objective terms are weighted by `NumDays`
- policy and scarcity accounting use representative-day weights
- grouped-day logic can use `repday_groups` and `Day_Idx_List`

Reporting functions also use both layers:
- `repdays` for day-level weights and reference days
- `repday_groups` for grouped-day interpretation

## How Operation Uses The Reduced Structure
Operational result reporting uses the representative-day and day-group structure to produce dispatch and power-flow outputs that still map back to grouped-day windows.

This matters especially when:
- multiple representative days belong to one day group
- the results need to be interpreted as a grouped operational block rather than isolated days

## How RA Uses The Reduced Structure
RA reuses the representative-day structure built upstream and does not invent a separate calendar representation.

The most important RA uses are:
- outage sampling by representative-day group
- renewable-scenario mapping onto the representative-day structure
- grouped-day execution in sequential and joint-scenario workflows

For RA, the important rule is:
- use `Day_Idx_List` to identify which representative days belong to a group
- use each representative day's `Day` field, that is `Reference_Day`, when mapping annual data back to the reduced horizon

## Practical Reading Guide
When checking a scenario-reduction run, it helps to inspect things in this order:
1. the representative-day selection mode in `Simulation Configuration`
2. the settings in `Scenario Reduction Setting`
3. the resulting `repday_groups`
4. the representative-day report that shows `RepDay_id`, `Daygroup_ID`, and `Reference_Day`

If grouped-day behavior looks wrong, the most useful questions are:
- was a manual representative-day file used or did the model run scenario reduction?
- do `Day_List` and `Day_Idx_List` match the intended grouping?
- does `Reference_Day` match the original day that should be used for time-series lookup?

## Related Documentation
- [A-LEAF Documentation](../README.md)
- [ALEAF Simulation Setting File Reference](../configuration/ALEAF_Simulation_Setting_File.md)
- [Simulation Configuration Reference](../configuration/Simulation_Configuration_Reference.md)
- [Network Data Reference](../database/Network_Data_Reference.md)
- [RA Overview](../models/RA/RA_Overview.md)
