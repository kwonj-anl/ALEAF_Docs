# ALEAF Simulation Setting File Reference

`ALEAF_Simulation_Setting_USA.xlsx` is the main user-facing control file for A-LEAF. It defines:
- what system is being studied
- which model families are run
- how the simulation is sequenced
- what technology, policy, financial, and reliability assumptions apply
- which scenario tables and external inputs are used

This document explains the structure of the workbook and the purpose of the main sheets and parameters.

## How To Read This Reference
The workbook contains two main types of sheets:
- setting-style sheets, where each row is a named parameter with a value and note
- table-style sheets, where each row is an entity or scenario record

This reference documents:
- parameter-level meaning for the setting-style sheets
- structural meaning of the table-style sheets and their key columns

## Workbook Structure
The `ALEAF_Simulation_Setting_USA.xlsx` workbook currently contains these sheets:
- `Simulation Setting`
- `Planning Design`
- `Gen Technology`
- `RA Setting`
- `Simulation Configuration`
- `RA Scenarios`
- `Scenario Reduction Setting`
- `ATB Setting`
- `Storage Cost and Performance`
- `ITC`
- `PTC`
- `CPLEX Setting`
- `HiGHS Setting`
- `Raw Materials`
- `Gen Technology Raw Materials`
- `External Constraints`
- `ATB Params`

## 1. `Simulation Setting`
This sheet defines model-wide simulation behavior and global numerical settings.

### Test System

#### `test_system_name`
Defines the base test system name used to locate data, organize outputs, and label the run.

### Topology Setting

#### `network_model_generation_method`
Controls how the network model is generated. This determines whether A-LEAF builds the system through its internal network-generation workflow or uses another configured approach.

#### `generation_representation_option`
Controls whether generation is represented as:
- individual plants
- aggregated technology blocks

This setting affects both model size and the level of operational detail.

#### `generation_parameter_source_option`
Controls where generator parameters come from when plants are explicitly represented. In practice this determines whether plant-level data is used directly or whether parameters are inherited from the `Gen Technology` sheet.

#### `transmission_network_generation_option`
Controls how transmission topology is built for the run.

#### `power_flow_mode_flag`
Controls the transmission representation used in optimization. Typical modes include:
- `PTDF`
- `Network_Flow`
- `B-theta`

This choice affects both model formulation and the required network data.

### Solution Method Setting

#### `solver_name`
Selects the optimization solver family used by the run, for example `CPLEX` or `HiGHS`.

#### `run_GTEP_in_parallel_flag`
Controls whether expansion-planning cases are run in parallel when multiple workers are available.

#### `run_operation_in_parallel_flag`
Controls whether operational simulations are run in parallel.

### Other Simulation Setting

#### `logging_level_value`
Controls logging detail. The current implementation uses values such as:
- `simple`
- `detailed`

#### `PTDF_threshold_value`
Controls PTDF filtering threshold when PTDF-based transmission modeling is used.

#### `per_unit_base_value`
Defines the power base for per-unit conversion.

#### `per_unit_econ_base_value`
Defines the economic base used with per-unit economic conversions.

#### `const_name_flag`
Controls whether constraints are saved with explicit names in the optimization model.

#### `export_model_reference_json_expansion_flag`
Controls whether expansion model reference data is exported in JSON form.

#### `export_model_reference_json_operation_flag`
Controls whether operation model reference data is exported in JSON form.

#### `export_model_lp_expansion_flag`
Controls whether the expansion model instance is exported in LP format.

#### `export_model_lp_operation_flag`
Controls whether the operation model instance is exported in LP format.

#### `model_lp_file_name_value`
Defines the LP export filename.

#### `MIP_relaxed_solution_bounds_flag`
Enables a workflow where the model solves a relaxed problem first and then uses that information to redefine integer bounds.

#### `MIP_relaxed_solution_bounds_step_value`
Defines the step size used in the relaxed-solution bound update process.

## 2. `Planning Design`
This sheet defines the long-term planning horizon, financial assumptions, and certain planning-wide features.

### Planning Horizon

#### `dollar_year_value`
Base dollar year used for cost normalization.

#### `current_year_value`
Starting year of the study.

#### `lead_year_value`
Lead year offset applied before planning decisions take effect.

#### `num_stages_value`
Number of planning stages.

#### `num_years_per_stage_value`
Number of years represented in each stage.

#### `targetyear_value`
Target year offset for future-study framing.

#### `final_year_value`
Final modeled year in the planning horizon.

### Multi Round Simulation

#### `multi_round_solution_process_flag`
Controls whether the planning problem is solved in one pass or through a multi-round workflow.

#### `num_stages_per_simulation_round_value`
Defines how many planning stages are solved in each round.

#### `num_overlaps_between_simulation_rounds_value`
Controls the overlap between adjacent multi-round solves.

#### `continue_from_previous_run_flag`
Allows continuation from a prior saved multi-round result.

### Financial

#### `WACC_value`
Weighted average cost of capital.

#### `discount_rate_value`
Social discount rate used in planning analysis.

#### `project_finance_factor_value`
Project-finance adjustment factor, used with capital-cost assumptions.

#### `reserve_cost_type_flag`
Controls how reserve costs are represented, for example:
- percentage of marginal cost
- absolute value in $/MWh

### Transmission Expansion and Line Loss

#### `transmission_expansion_limit_value`
Upper bound on transmission expansion.

#### `transmission_cost_value`
Transmission upgrade cost, typically in $/MW-mile.

#### `transmission_investment_CRP_value`
Capacity recovery period for transmission investments.

#### `transmission_loss_percent_value`
Transmission loss factor by distance.

### Inertia and Material

#### `enforce_rotational_inertia_constraints_flag`
Enables rotational inertia constraints when supported by the model formulation.

#### `enforce_material_constraints_flag`
Enables raw material constraints in the expansion model.

### Other Planning Parameters

#### `RSR_level_investment_flag`
A specialized investment-setting flag used in the planning workflow.

#### `num_scenario_value`
Number of stochastic scenarios used when a stochastic planning formulation is active.

#### `num_hours_per_day_value`
Hours represented per day.

#### `num_sub_period_value`
Sub-hourly resolution multiplier.

#### `num_days_per_OP_run_value`
Number of days in a single operation-model run grouping.

#### `reference_carbon_emission_level_value`
Reference carbon-emission level used in policy or comparison logic.

## 3. `Gen Technology`
This is the master technology table. Each row defines a generation or storage technology option, and the columns span multiple categories such as:
- category
- dispatch and commitment options
- cost inputs
- technical characteristics
- market and policy
- reliability
- investment options
- resource limits

This sheet acts as the central source for technology-level assumptions used across A-LEAF.

Important observations:
- each technology has a `Tech_ID`
- many cost fields can be tied to `ATB` or other external sources
- reliability, commitment, and investment flags live in this sheet

Because this is a wide entity table rather than a simple settings sheet, it is best documented by column groups rather than row-by-row records.

## 4. `RA Setting`
This sheet controls the Reliability Assessment workflow.

### Network Setting

#### `use_predefined_network_data_flag`
If enabled, RA uses saved expansion outputs as its network input instead of rebuilding the network from the current run context.

#### `predefined_network_data_file_name`
Filename of the saved expansion/network data used when predefined network input is enabled.

#### `generate_gen_index_with_investment_options_flag`
Controls whether the RA generator index should include investment-option mapping. This is useful for keeping generator indexing consistent across scenarios or reused inputs.

#### `round_id_to_run_RA`
Selects which saved planning round is used when RA is tied to prior expansion-planning output.

### RA Setting

#### `num_risk_scenario`
Number of outage samples generated per representative day group.

#### `RA_method`
Selects the RA simulation mode, for example:
- `Economic Dispatch`
- `Sequential Economic Dispatch`

#### `system_peak_scale`
Multiplier applied to load during RA runs.

#### `risk_filtering_flag`
Enables or disables RA filtering.

#### `risk_tol_value`
Filtering threshold used in scenario screening.

#### `min_num_risk_in_each_day_value`
Lower bound on retained scenarios per day group, expressed relative to the sampled outage count.

#### `repair_time_average_hours`
Mean repair-time assumption used in outage restoration sampling.

#### `repair_time_bound`
Bound or quantile used to limit sampled repair times.

### Capacity Accreditation Setting

#### `calculate_capacity_credit_Flag`
Enables capacity-accreditation analysis after the base RA run.

#### `capacity_credit_type`
Selects the capacity-accreditation method, for example:
- `ELCC`
- `DLOL`
- `Lookup Table`

#### `capacity_credit_assessment_mode`
Controls whether the accreditation study:
- adds a new resource
- deactivates an existing resource

#### `capacity_credit_RA_simulation_method`
Specifies which RA simulation method is used during capacity-credit analysis.

#### `capacity_credit_reference_RA_metric`
Reference metric used as the accreditation target, such as `EUE` or `NEUE`.

#### `capacity_credit_reference_RA_metric_spatial_resolution`
Defines whether the accreditation target is:
- `Systemwide`
- `Regional`

#### `capacity_credit_max_iteration_value`
Maximum iterations allowed in the ELCC solving loop.

#### `capacity_credit_abs_tol_value`
Absolute convergence tolerance for ELCC.

#### `capacity_credit_rel_tol_value`
Relative convergence tolerance for ELCC.

### Forced Outage Rate Setting

#### `reference_temp`
Reference temperature used in temperature-sensitive outage modeling.

### Storage Setting

#### `Post_Contingency_Discharge_method`
Controls storage discharge behavior in the post-contingency RA simulation.

#### `Post_Contingency_Charge_method`
Controls storage charging behavior in the post-contingency RA simulation.

### Other RA Settings

#### `distributed_run_flag`
Enables distributed RA execution.

#### `num_distributed_scenarios_per_worker_value`
Defines the scenario batch-size cap per worker in distributed execution.

#### `export_dispatch_results_threshold_value`
ENS threshold used to decide whether dispatch results are exported.

#### `export_baseline_dispatch_results_flag`
Controls export of reference ED outputs.

#### `export_ELCC_dispatch_results_flag`
Controls export of ELCC-specific dispatch outputs.

#### `preselected_days_list`
Optional list of representative day groups to include in RA.

#### `output_verbose_level`
Controls how much internal RA detail is retained in outputs.

## 5. `Simulation Configuration`
This sheet controls case-by-case model sequencing and many model-specific options. It is one of the most important sheets in the workbook.

Structure:
- each case is stored in its own column
- the first column gives the category
- the second column gives the setting name
- subsequent columns define values for each case

This sheet determines:
- whether a case runs at all
- whether it is an expansion, operation, or RA case
- which input data variants are used
- representative-day settings by model family
- dispatch mode choices
- policy flags
- reliability and reserve settings
- scenario-specific overrides such as load, wind, PV, fuel, hydro, and ATB selections

Because this sheet is broad and central to the whole framework, it should likely be documented in a dedicated standalone page later.

## 6. `RA Scenarios`
This sheet defines the renewable scenarios used by RA.

Columns:
- `Scenario_ID`
- `Enabled`
- `Weight`
- `Wind_Ons_File_ID`
- `PV_File_ID`

Meaning:
- `Scenario_ID`: renewable scenario name
- `Enabled`: whether the scenario participates in the RA run
- `Weight`: renewable scenario probability weight used in annual RA metrics
- `Wind_Ons_File_ID`: onshore wind timeseries identifier
- `PV_File_ID`: PV timeseries identifier

Important note:
- enabled renewable scenarios are combined with outage samples to form the full joint scenario space used by RA

## 7. `Scenario Reduction Setting`
This sheet controls representative-day selection and scenario-reduction inputs.

Key parameters include:
- temporal resolution
- which data types are used in scenario reduction
- whether duration curves are generated
- whether extreme days are fixed
- whether representative-day overlap is allowed
- optional preselected extreme-day lists

This sheet is central when representative-day groups are generated from full-year timeseries rather than read from a pre-defined file.

## 8. `ATB Setting`
This table maps A-LEAF technology groups to ATB assumptions.

It typically includes:
- `ATB_Setting_ID`
- `UNITGROUP`
- fuel and technology classifications
- ATB case/scenario selections
- technology detail fields

This table is used to connect A-LEAF technology definitions to ATB cost/performance assumptions.

## 9. `Storage Cost and Performance`
This sheet stores storage cost and performance assumptions, typically by setting id and technology duration/type. It includes fields such as:
- MW and hour definitions
- round-trip efficiency
- cycle life
- calendar life
- annual energy throughput
- fixed and variable O&M
- year-by-year cost trajectories

This sheet is used when storage cost and performance are selected through simulation configuration and technology mappings.

## 10. `ITC` and `PTC`
These sheets provide year-by-year tax credit values by technology.

They are used when:
- production tax credits
- investment tax credits

are active in the selected case configuration.

## 11. Solver Sheets

### `CPLEX Setting`
Contains parameter, flag, value, and note columns for the CPLEX solver configuration.

Typical controls include:
- MIP gap
- display/logging
- time limit
- feasibility tolerance
- thread count
- presolve
- scaling
- algorithm choice
- cut and node strategies

### `HiGHS Setting`
Contains solver controls for the HiGHS solver, including:
- time limit
- presolve
- thread count
- output flag

## 12. Raw Material Sheets

### `Raw Materials`
Defines material ids, material types, and annual limits.

### `Gen Technology Raw Materials`
Defines the material intensity of selected technologies by linking technology ids to material quantities.

These sheets are only relevant when material constraints are enabled in planning.

## 13. `External Constraints`
This table allows user-defined external constraints to be applied to selected models.

Typical columns include:
- constraint id
- apply flag
- target model
- resource unit group
- parameter
- region id
- operator
- value

This sheet is used for additional user-defined restrictions such as investment caps, retirement requirements, or regional build limits.

## 14. Other Sheets

### `Scenario ->`, `Financial Params ->`, `Solver Settings ->`, `Matarial Settings ->`, `Other Settings ->`, `Ancillary Tabs->`
These appear to function as workbook section separators or organizational tabs rather than primary input tables.

### `ATB Params`
This sheet is part of the ATB-related data bundle and supports the technology and cost mapping workflow.

## Practical Guidance
- Start with `Simulation Setting`, `Planning Design`, and `Simulation Configuration` to understand overall run behavior.
- Use `Gen Technology`, `ATB Setting`, and `Storage Cost and Performance` to understand technology assumptions.
- Use `RA Setting` and `RA Scenarios` when working on reliability assessment.
- Use `Scenario Reduction Setting` when representative-day construction is part of the workflow.
- Use `External Constraints` only when the study needs custom user-defined limits.

## Related Documentation
- [A-LEAF Documentation](../README.md)
- [RA Overview](../models/RA/RA_Overview.md)
- [RA Settings and Scenarios Reference](../models/RA/RA_Settings_and_Scenarios_Reference.md)
