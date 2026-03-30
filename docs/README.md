# Argonne Low-Carbon Electricity Analysis Framework (A-LEAF)

An integrated national-scale simulation framework for power system operations and planning.

## What A-LEAF Is
A-LEAF is an integrated national-scale power system simulation framework that includes a suite of capacity expansion, unit commitment, and economic dispatch models. It can determine least-cost generation investment plans, generation retirement plans, transmission investment plans, and hourly system scheduling under user-defined assumptions for technology characteristics, electricity demand profiles, system requirements, and electricity market designs.

A-LEAF is designed so these model families can be used either as standalone tools or as linked stages in a broader study workflow. 

The documentation in this folder is organized to support a technical review of the framework, data inputs, and model workflows.

## Model Families
A-LEAF currently includes three major model families: generation and transmission expansion planning, system operation, and reliability assessment.

### Generation and Transmission Expansion Planning (GTEP)
A-LEAF includes a long-term generation and transmission expansion planning model. A-LEAF determines the time, location, and size of new generation and transmission assets while considering the time-varying resource potentials of renewable resources such as wind and solar. It also considers age-based retirement of existing generators, optional retirement of existing units when doing so lowers total system cost, and storage additions and duration choices. This allows the framework to evaluate how portfolios evolve over the planning horizon rather than treating the system as fixed.

The least-cost objective can include new generation investment, generator retirement, transmission investment, fixed and variable operation and maintenance, fuel costs, involuntary load curtailment, and applicable policy-related incentives or requirements. The planning model minimizes total system costs while respecting constraints related to serving electricity demand, meeting resource adequacy targets, maximum and minimum regional generation investments, technical characteristics of generation and storage resources, and policies and regulations.

### System Operation 
The operation model solves a security-constrained unit commitment or economic dispatch problem to determine the optimal dispatch of generation, energy storage, and ancillary services. The dispatch formulation includes constraints for commitment, load balance, power flow and transmission limits, operating reserve requirements, and generator operating limits. Load-balance constraints ensure that enough power is supplied to meet demand in each region and time interval. Power flows can be represented either through a DCOPF or aggregate transfer capability between regions. The operation model also schedules operating reserves while respecting the operating limits of the resources that provide them.

### Reliability Assessment
The reliability assessment model evaluates system resource adequacy under uncertainty, including generator outage uncertainty and renewable availability uncertainty. It is used to estimate outcomes such as expected unserved energy, loss-of-load outcomes, and ELCC-style reliability impacts.

## How The Models Fit Together
The model families are complementary rather than independent:
- expansion planning defines a future system configuration
- operation modeling evaluates how that configuration performs operationally
- reliability assessment evaluates how that configuration performs under stress and uncertainty

Not every study uses all three. A-LEAF supports both standalone runs for a fixed system and sequential workflows where one model feeds another.

Simulation sequencing is controlled through the settings workbook. The workbook determines which models are run, which datasets and assumptions are used, and whether outputs from one model are reused by another. Common sequencing patterns include:
- GTEP only
- operation only
- RA only
- GTEP followed by operation
- GTEP followed by RA

## Technical And Operational Characteristics Of Energy Storage
A-LEAF includes a detailed representation of the physical and operational constraints of energy storage resources. Storage resources can provide capacity, energy, and operating reserves. Modelers can either define storage capacity directly or let A-LEAF endogenously determine optimal new storage capacity.

The framework tracks the state of charge of storage resources while respecting chronological charging and discharging interactions. The total amount of energy a storage resource can deliver over a year can also be restricted through an energy throughput constraint, which acts as a proxy for cycle-life limitations, especially for battery technologies.

## Policy And Regulations
A-LEAF can represent a range of policies and regulations, including technology-specific incentives such as investment tax credits and production tax credits, renewable portfolio standards, emissions limits, and internalized carbon costs. These policy structures can affect both planning outcomes and operational results, depending on the selected study design and model settings.

## Core Inputs
At the framework level, A-LEAF depends on a few major input categories:
- the simulation setting workbook
- network data
- time-series data
- optional saved outputs reused from earlier runs

The most important shared input references in this package are:
- [ALEAF Simulation Setting File Reference](./configuration/ALEAF_Simulation_Setting_File.md)
- [Simulation Configuration Reference](./configuration/Simulation_Configuration_Reference.md)
- [Network Data Reference](./database/Network_Data_Reference.md)
- [Network Configuration Reference](./database/Network_Configuration_Reference.md)
- [Gen Technology Reference](./database/Gen_Technology_Reference.md)
- [Scenario Reduction and Representative-Day Groups](./configuration/Scenario_Reduction_and_Repday_Groups.md)
- [Policy and Financial Settings](./configuration/Policy_and_Financial_Settings.md)

## Core Outputs
The main outputs depend on which model family is used, but the common output categories include:
- JSON result files
- dispatch and system result tables
- saved expansion-planning decisions
- reliability metrics and summaries
- reusable artifacts such as fixed-risk samples

## Cross-Cutting Modeling Topics
Several modeling topics cut across more than one model family:
- [Storage Modeling Reference](./database/Storage_Modeling_Reference.md)
- [Hybrid Resources Reference](./database/Hybrid_Resources_Reference.md)
- [Large Load and Demand Response Reference](./database/Large_Load_and_Demand_Response_Reference.md)

## Model Documentation

### Generation and transmission expansion planning
- [GTEP Overview](./models/GTEP/GTEP_Overview.md)
- [GTEP Planning Horizon and Multi-Round](./models/GTEP/GTEP_Planning_Horizon_and_Multi_Round.md)
- [GTEP Expansion Outputs](./models/GTEP/GTEP_Expansion_Outputs.md)

### Operation
- [Operation Overview](./models/Operation/Operation_Overview.md)
- [Operation Execution and Run Modes](./models/Operation/Operation_Execution_and_Run_Modes.md)
- [Operation Outputs](./models/Operation/Operation_Outputs.md)

### Reliability assessment
- [RA Overview](./models/RA/RA_Overview.md)
- [RA Scenarios and Data](./models/RA/RA_Scenarios_and_Data.md)
- [RA Execution and Results](./models/RA/RA_Execution_and_Results.md)
- [RA Metrics, ELCC, and DLOL](./models/RA/RA_Metrics_ELCC_and_DLOL.md)
- [RA Settings and Scenarios Reference](./models/RA/RA_Settings_and_Scenarios_Reference.md)
