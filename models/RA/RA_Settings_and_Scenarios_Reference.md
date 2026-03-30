# RA Settings and Scenarios Reference

This page explains the `RA Setting` and `RA Scenarios` sheets used by the Reliability Assessment model. It focuses on what each parameter controls in the implemented RA workflow and when the parameter matters.

## `RA Setting`

### Core Simulation Settings

#### `round_id_to_run_RA`
Used when RA is run from expansion-planning outputs. It selects which saved planning round should be used to define the generation mix and current year for the RA run.

#### `num_risk_scenario`
Controls the number of outage samples generated per representative day group. In the current joint-scenario design, the full scenario space is:
- `num_risk_scenario x num_renewable_scenarios`

#### `RA_method`
Selects the main RA simulation mode. In practice this determines whether RA uses the perfect-foresight or sequential post-contingency ED path.

#### `reference_temp`
Controls the temperature or reference condition used in outage sample generation and related reliability preparation.

#### `system_peak_scale`
Scales load in the RA run. This directly affects dispatch, scarcity outcomes, and all demand-normalized reliability metrics.

### Risk Filtering and Sampling

#### `risk_tol_value`
Controls the tolerance used in risk filtering. It affects how aggressively the model screens the joint scenario set before dispatch simulation.

#### `min_num_risk_in_each_day_value`
Sets a lower bound on the number of retained risk scenarios per day group when filtering is active.

#### `risk_filtering_flag`
Turns the RA filtering step on or off.
- `true`: use the risk-screening logic to retain only selected joint scenarios
- `false`: retain all joint scenarios, subject to any preselected-day restriction

#### `preselected_days_list`
Optional list of representative day groups to simulate. If left empty, all day groups are eligible. If populated, RA only retains scenarios for the listed day groups.

#### `repair_time_bound`
Controls the outage repair-time bound used in RA outage treatment.

### Execution and Performance

#### `distributed_run_flag`
Enables distributed execution of RA scenarios when that execution mode is configured.

#### `num_distributed_scenarios_per_worker_value`
Controls the scenario batch size assigned to each worker in distributed runs.

#### `output_verbose_level`
Controls how much detail is retained in the RA output payload.

### Output Controls

#### `export_dispatch_results_threshold_value`
Sets the threshold used to decide which scenario dispatch results should be exported.

#### `export_baseline_dispatch_results_flag`
Controls whether reference ED dispatch outputs are exported.

#### `export_ELCC_dispatch_results_flag`
Controls whether ELCC-specific dispatch outputs are exported when ELCC is enabled.

### Capacity Credit / ELCC Settings

#### `calculate_capacity_credit_Flag`
Turns capacity credit post-processing on or off.

#### `capacity_credit_type`
Selects the capacity credit method.

#### `capacity_credit_RA_simulation_method`
Controls which RA simulation mode is used during capacity credit analysis.

#### `capacity_credit_assessment_mode`
Controls whether the ELCC workflow assesses:
- adding a new resource
- deactivating an existing resource

#### `capacity_credit_reference_RA_metric`
Specifies which RA metric is used as the reference target in ELCC calculations.

#### `capacity_credit_reference_RA_metric_spatial_resolution`
Specifies whether the ELCC target metric is evaluated at the:
- systemwide level
- regional level

#### `capacity_credit_rel_tol_value`
Relative convergence tolerance for ELCC iteration.

#### `capacity_credit_abs_tol_value`
Absolute convergence tolerance for ELCC iteration.

#### `capacity_credit_max_iteration_value`
Maximum number of ELCC iterations.

### Post-Contingency Storage Settings

#### `Post_Contingency_Discharge_method`
Controls how storage discharge is handled in the post-contingency RA simulation.

#### `Post_Contingency_Charge_method`
Controls how storage charging is handled in the post-contingency RA simulation.

### Other RA-Related Behavior

#### `generate_gen_index_with_investment_options_flag`
Used during RA generator-index preparation when the model needs to include investment-option representations in the RA generator index.

## `RA Scenarios`

### Purpose
The `RA Scenarios` sheet defines the renewable scenarios used in RA. These scenarios are discrete weighted cases that are combined with outage samples to form the full joint scenario space.

### Expected Columns

#### `Scenario_ID`
Unique renewable scenario name. This is the id carried through:
- renewable scenario parsing
- joint scenario creation
- reference ED by renewable scenario
- final metric weighting

The base renewable scenario is should be `BASE`.

#### `Enabled`
Determines whether the renewable scenario participates in the RA run.

Only enabled scenarios are loaded, weighted, and included in the joint scenario map.

#### `Weight`
Defines the renewable scenario probability weight used in annual RA metrics.

In the current implementation, the default joint scenario weight is:
- `renewable_scenario_weight / num_risk_scenario`

#### `Wind_Ons_File_ID`
Identifies the onshore wind timeseries file used for the scenario.

For the base renewable scenario, RA uses the default wind timeseries path from the main file-path settings. For non-base scenarios, RA loads from the additional-scenario renewable data location.

#### `PV_File_ID`
Identifies the utility-scale PV timeseries file used for the scenario.

For the base renewable scenario, RA uses the default PV timeseries path from the main file-path settings. For non-base scenarios, RA loads from the additional-scenario renewable data location.

### How `RA Scenarios` Is Used
The `RA Scenarios` sheet drives four parts of the RA workflow:
- renewable scenario parsing and weight assignment
- renewable timeseries loading
- joint outage x renewable scenario construction
- reference ED and metric weighting by renewable scenario

