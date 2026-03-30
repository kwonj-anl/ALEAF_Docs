# Simulation Configuration Reference

The `Simulation Configuration` sheet is the case-level control surface for A-LEAF. Each case is stored in its own column, and each column determines:
- which models run
- which input data variants are used
- how representative days are selected
- which dispatch, policy, and reliability options apply

This sheet is one of the most important parts of the workbook because it controls both simulation sequencing and case-specific scenario definitions.

## Sheet Structure
The sheet is organized as a case matrix:
- column `A` stores a broad category label
- column `B` stores the setting name
- columns `C`, `D`, `E`, and beyond define individual simulation cases

Each case column is interpreted as one case configuration. A case is activated when its `Run_Flag` is enabled.

## How A Case Is Interpreted

### `Run_Flag`
Controls whether a case is executed.

### `Case_ID`
Defines the case name used in outputs and logging.

### Model Run Flags
The main run-sequencing controls are:
- `Run_expansion_flag`
- `Run_operation_flag`
- `Run_RA_flag`

These determine whether the case runs:
- expansion planning
- production cost / operation simulation
- reliability assessment

The same case can enable more than one model family, which is how linked workflows are defined. In the codebase these flags are used directly to decide whether A-LEAF runs expansion first, reuses saved expansion outputs, or launches operation and RA from an existing planning state.

## Parameter Groups

## 1. Case Definition

### `Run_Flag`
Turns the case on or off.

### `Case_ID`
Assigns the case label used in outputs and scenario naming.

## 2. Network Data File Selection
These fields determine which network and representative-day inputs are used for the case.

### `Network_Data_File_ID`
Selects the base network data variant.

### `Network_Configuration_File_ID`
Selects the network-configuration variant, such as regional aggregation or topology configuration.

### `Repday_File_ID`
Selects a pre-defined representative-day file when a manual or precomputed representative-day input is used.

### `Repday_Selection_Mode_EXP`
Controls how representative days are selected for the expansion model. This setting is consumed when the expansion model builds or loads representative-day groups.

### `Repday_Selection_Mode_OP`
Controls how representative days are selected for the operation model.

### `Repday_Selection_Mode_RA`
Controls how representative days are selected for the reliability model.

## 3. Expansion Model Settings

### `Run_expansion_flag`
Turns the expansion model on or off for the case.

### `Dispatch_Mode_in_EXP`
Selects the expansion-model operational representation, for example:
- `Economic Dispatch`
- `Unit Commitment`

This is not just a reporting label. It changes which variables, constraints, and commitment logic are built in the expansion model.

### `transmission_expansion_flag`
Enables or disables transmission expansion decisions in the expansion model.

### `operating_reserve_modeling_option`
Controls how reserves are represented. Common options include:
- aggregated
- individual

This field is used directly inside reserve constraints and reserve-shortage formulations.

### `include_dispatch_ramping_flag`
Turns dispatch ramping constraints on or off.

### `dispatch_ramping_modeling_option`
Controls how ramping is represented when ramping is enabled. In practice this selects the ramping formulation when ramping is active and dispatch is not handled through full unit commitment logic.

### `NDAY_Groups`
Number of representative day groups used by the expansion model.

### `NDAYS_in_Single_Group`
Number of days in each representative day group used by the expansion model.

These two fields are part of the actual representative-day construction workflow, not just documentation metadata.

### `Load_Shed_in_EXP_Flag`
Controls whether load shedding is allowed in the expansion model.

### `OR_Shortage_in_EXP_Flag`
Controls whether operating reserve shortage is allowed in the expansion model.

## 4. Stochastic Expansion Settings
These fields apply only when stochastic expansion is enabled.

### `Stochastic_Expansion_Flag`
Turns stochastic expansion on or off.

### `Stochastic_File_ID`
Selects the stochastic scenario file bundle.

### `Stochastic_Load_Flag`
Enables stochastic load treatment.

### `Stochastic_Wind_Ons_Flag`
Enables stochastic onshore wind treatment.

### `Stochastic_PV_Flag`
Enables stochastic PV treatment.

### `Num_Sto_Scenarios`
Number of stochastic scenarios used in the expansion model.

## 5. Operation Model Settings

### `Run_operation_flag`
Turns the operation / production-cost model on or off.

### `Use_predefined_expansion_data_for_OP_flag`
If enabled, operation runs use saved expansion outputs instead of the expansion state from the current run. This is one of the main sequence-control switches for chained workflows.

### `predefined_expansion_data_file_name_for_OP`
Specifies the saved expansion-output file used by the operation model when predefined expansion data is enabled.

### `Dispatch_Mode_in_OP`
Selects the operation-model dispatch style, such as:
- `Economic Dispatch`
- `Unit Commitment`

Like the expansion dispatch mode, this field directly changes which operational model is built.

### `NDAY_Groups_OP`
Number of representative day groups used in operation simulation.

### `NDAYS_in_Single_Group_OP`
Number of days in each operation-model representative day group.

### `Load_Shed_in_OP_Flag`
Controls whether load shedding is allowed in operation simulation.

### `OR_Shortage_in_OP_Flag`
Controls whether reserve shortage is allowed in operation simulation.

## 6. Reliability Assessment Case Settings

### `Run_RA_flag`
Turns the reliability assessment model on or off.

### `Use_predefined_expansion_data_for_RA_flag`
If enabled, RA uses saved expansion outputs instead of the current in-memory planning state. This is the RA-side sequence-control switch for using a saved planning result as the system state under study.

### `predefined_expansion_data_file_name_for_RA`
Specifies the saved expansion-output file used when predefined expansion input is enabled for RA.

### `update_CAPCRED_in_each_round_of_Expansion_Flag`
Controls whether capacity-credit information is updated in each expansion round. This matters when expansion and RA are linked through multi-round workflows.

### `NDAY_Groups_RA`
Number of representative day groups used in reliability assessment.

### `NDAYS_in_Single_Group_RA`
Number of days in each RA representative day group.

These settings interact with the `RA Setting` sheet, which controls how the RA simulation is executed once the case has selected the RA run itself.

## 7. Planning Design Overrides

### `load_increase_rate_value`
Case-level load-growth override used in planning calculations.

### `enforce_min_reserve_margin_flag`
Turns planning reserve margin enforcement on or off.

### `planning_reserve_margin_type`
Specifies how the reserve margin is interpreted. The code distinguishes at least `maximum` and `minimum` style formulations.

### `planning_reserve_margin_value`
Defines the reserve-margin target value used directly in reserve-margin constraints.

## 8. Storage Operation

### `contingency_reserve_min_duration_value`
Defines the minimum duration requirement for contingency reserve.

### `storage initialization option`
Controls initial storage state assumptions, for example:
- minimum
- middle
- maximum

This setting is read directly by storage state-of-charge initialization logic.

## 9. Financial Options and Linked Tables

### `Fuel_ID`
Selects the fuel-price timeseries variant for the case.

### `ESGC_Setting_ID`
Selects the storage cost/performance setting used by the case.

### `ATB_Year`
Specifies the ATB year used for technology assumptions.

### `ATB_Setting_ID`
Selects the ATB mapping configuration used to assign cost/performance assumptions to technologies.

These fields connect the case definition to other workbook tables such as:
- `ATB Setting`
- `Storage Cost and Performance`

## 10. Market Parameters

### `VOLL`
Value of lost load used directly in objective-function penalty terms for unserved energy in operation and RA.

### `RegRSP`
Regulation reserve shortage penalty coefficient used in the objective when reserve shortfalls are allowed.

### `SRSP`
Spinning reserve shortage penalty coefficient used in the objective when reserve shortfalls are allowed.

### `NSRSP`
Non-spinning reserve shortage penalty coefficient used in the objective when reserve shortfalls are allowed.

### `FLEXRSP`
Flexibility reserve shortage penalty coefficient used in the objective when reserve shortfalls are allowed.

## 11. Reliability / Scarcity Caps in Expansion
These fields control optional annual adequacy caps in expansion planning.

### `Total_ENS_MWh_Cap_Flag`
Turns the annual total ENS cap on or off.

### `Total_ENS_MWh`
Annual total ENS cap value.

### `ENS_Hours_Cap_Flag`
Turns the ENS-hours cap on or off.

### `ENS_Hours`
ENS-hours cap value.

### `Max_ENS_MWh_Cap_Flag`
Turns the maximum single-period ENS cap on or off.

### `Max_ENS_MWh`
Maximum ENS cap value.

## 12. Policy and Regulation

### Tax Credit and Carbon Tax

#### `ITC_Flag`
Enables investment tax credit treatment.

#### `PTC_Flag`
Enables production tax credit treatment.

#### `CTAX`
Carbon tax value used in dispatch and planning calculations.

### Renewable Portfolio Standard

#### `RPS_Flag`
Turns the RPS requirement on or off.

#### `Allow_Alternative_RPS_Compliance_Flag`
Allows alternative compliance treatment when RPS is active.

#### `RPS_Penalty`
Penalty used when RPS compliance is soft rather than hard.

#### `RPS_Global_Target_Value`
Global RPS target value.

### Clean Energy Generation Target

#### `Clean_Energy_Generation_Target_EXP_Flag`
Enables clean-energy target treatment in expansion planning.

#### `Clean_Energy_Generation_Target_EXP_Type`
Controls how the expansion target is applied, such as:
- annual
- daygroup

#### `Clean_Energy_Generation_Target_OP_Flag`
Enables clean-energy target treatment in operation simulation.

#### `Clean_Energy_Generation_Target_Start_Year`
Year when clean-energy target enforcement starts.

#### `Clean_Energy_Generation_Global_Target_Value`
Target value for clean-energy generation.

#### `Clean_Energy_Generation_Penalty`
Penalty used when the clean-energy target is soft.

### Carbon Emission Reduction Target

#### `Carbon_Emission_Reduction_Target_Flag`
Enables the carbon reduction target.

#### `Carbon_Emission_Reduction_Global_Target_Value`
Global carbon reduction target value.

#### `Carbon_Emission_Reduction_Target_Start_Year`
Year when carbon reduction target enforcement starts.

## 13. Regional Resource Limits

### `Regional_resource_limits_flag`
Turns regional resource-limit enforcement on or off.

### `Resource_limit_level_value`
Selects the regional resource-limit level, such as:
- low
- high

### `Regional_CAPAX_scailing_flag`
Controls regional scaling treatment for capacity expansion limits.

## 14. Time-Series Data ID Selectors
These fields select the timeseries variants used by the case.

### `Load_File_ID`
Load timeseries selector.

### `Wind_Ons_File_ID`
Onshore wind timeseries selector.

### `PV_File_ID`
PV timeseries selector.

### `Temperature_File_ID`
Temperature timeseries selector.

These selectors are important because they can change both operations and reliability outcomes without changing the underlying network data.

## 15. Hydro Options

### `Hydro_Budget_File_ID`
Hydro budget input selector.

### `Hydro_Value_File_ID`
Hydro water-value input selector.

### `Hydro_Flexibility_Flag`
Turns hydro flexibility treatment on or off. This field is read directly by hydro operating constraints.

### `Hydro_Flexibility_Percent`
Hydro flexibility level used when hydro flexibility is enabled.

### `Hydro_Budget_Flag`
Turns hydro budget constraints on or off.

### `Reservoir_Hydro_Operation_Option`
Defines how reservoir hydro is represented operationally, for example through day-group or annual budget treatment. This field selects between different hydro budget formulations in the model.

## 16. Other Case-Level Settings

### `Energy_Stroage_AET_Limit_Flag`
Turns annual energy-throughput limits for storage on or off. This field activates storage throughput constraints based on technology `AET`.

### `External_Constraints_ID`
Selects which external constraint set applies to the case.

### `HFREQ`
Controls the hour-to-hour time-step spacing used in ramping, storage state transitions, and other intertemporal constraints.

### `FIVEMIN`
Enables sub-hourly 5-minute treatment in formulations that support it.

### `FIVEMIN_OP`
Enables sub-hourly 5-minute treatment for the operation model.

### `PD`
System peak-demand parameter used in planning calculations. It is updated from the network data and then used in reserve-margin and peak-demand calculations.

## How This Sheet Interacts With Other Sheets
`Simulation Configuration` is the bridge between the high-level workbook settings and the actual model run. It interacts directly with:
- `Simulation Setting`
- `Planning Design`
- `RA Setting`
- `RA Scenarios`
- `Scenario Reduction Setting`
- `ATB Setting`
- `Storage Cost and Performance`
- `ITC`
- `PTC`
- `External Constraints`

## Practical Reading Guide
When reviewing or creating a case:
1. Start with `Run_Flag` and `Case_ID`.
2. Check which model families are enabled.
3. Confirm the representative-day settings for the active model family.
4. Check the input-data selectors for load, fuel, wind, PV, hydro, and ATB/storage assumptions.
5. Review policy and reserve settings.
6. For RA cases, verify that the case-level RA settings and the `RA Setting` sheet are aligned.

## Related Documentation
- [ALEAF Simulation Setting File Reference](./ALEAF_Simulation_Setting_File.md)
- [A-LEAF Documentation](../README.md)
- [RA Overview](../models/RA/RA_Overview.md)
