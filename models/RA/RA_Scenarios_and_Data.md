# RA Scenarios and Data

## Outage Samples
Raw outage uncertainty is stored in `risk_scenario_dict`. Each outage sample is keyed by day group and outage sample id and contains the generator outage matrix used in RA simulation.

Outage samples are generated once and then reused across renewable scenarios. This keeps the outage sampling logic separate from renewable scenario configuration.

## Renewable Scenarios
Renewable uncertainty is configured through the `RA Scenarios` sheet. Enabled scenarios define:
- scenario ids
- scenario weights
- renewable timeseries file identifiers

The current implementation uses renewable scenarios for `wind_ons` and `pv`. Annual renewable profiles are loaded from normalized timeseries files, stored by timeseries tag, and then mapped to representative-day groups before filtering and simulation.

## Joint Scenarios
RA combines outage samples and renewable scenarios into joint scenarios. Each joint scenario contains:
- `outage_risk_id`
- `renewable_scenario_id`

This is the scenario unit used for filtering, simulation, and metric aggregation. The joint design is important because low-renewable conditions can matter even when the outage pattern is unchanged.

## Filtered Joint Scenarios
The filtered simulation workload is stored in `filtered_joint_scenario_map`. This is the authoritative list of RA scenarios that will be simulated after screening.

Filtering is still based on the RA risk-screening logic, but the renewable contribution now depends on the renewable scenario assigned to each joint scenario.

## Fixed-Risk Input Format
When reusing fixed risk samples, the RA input should contain:
- `risk_scenario_dict`
- `filtered_joint_scenario_map`

This preserves both the outage samples and the filtered joint scenario definition. A fixed-risk run therefore reproduces both the sampled outage states and the retained joint scenario workload.

## Important RA Data Objects
- `RA_setting`: RA configuration and scenario settings
- `RA_Info`: main container for RA inputs, results, metrics, and scenario metadata
- `risk_scenario_dict`: raw outage samples
- `joint_scenario_map`: full outage x renewable scenario space
- `filtered_joint_scenario_map`: retained joint scenarios after filtering
- `renewable_scenario_data`: raw renewable scenario timeseries by tag
- `renewable_scenario_daygroup_data`: renewable scenario timeseries mapped to representative-day groups
- `daily_reference_ED_solutions`: reference ED results by day group and renewable scenario

## Key Modeling Assumptions
- Outage samples are generated independently from renewable scenarios.
- Renewable scenarios are discrete weighted scenarios rather than a continuous stochastic process.
- The filtered-out joint scenarios are treated as non-binding scenarios for simulation and contribute zero to expected-value metrics.
- Representative-day weighting is preserved when scenario data is mapped and when metrics are aggregated.
