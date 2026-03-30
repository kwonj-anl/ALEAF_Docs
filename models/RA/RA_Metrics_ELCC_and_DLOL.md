# RA Metrics, ELCC, and DLOL

## Reliability Metrics
The main RA metrics are:
- `EUE`: Expected Unserved Energy
- `NEUE`: Normalized Expected Unserved Energy
- `LOLH`: Loss of Load Hours
- `LOLE`: Loss of Load Expectation

These metrics are reported at annual and day-group levels and, where applicable, at regional and systemwide levels.

## How To Interpret The Main Metrics
- `EUE` measures expected unserved energy and is typically the clearest adequacy severity metric.
- `NEUE` normalizes unserved energy by demand and is useful for comparing systems or regions of different sizes.
- `LOLH` measures the expected number of outage hours.
- `LOLE` converts `LOLH` into a days-per-year style adequacy measure.

## Weighting Over Joint Scenarios
RA metrics are aggregated over filtered joint scenarios. The default joint scenario weight is:
- `renewable_scenario_weight / num_risk_scenario`

This reflects equal weighting across outage samples and configured weights across renewable scenarios. Filtered-out scenarios are treated as zero-contribution scenarios.

The current metric design applies joint-scenario weights during accumulation rather than normalizing at the end with an outage-only divisor.

## Max Metrics
The following metrics remain scenario extrema over solved scenarios rather than expected values:
- Max consecutive outage hours
- Max MWh loss
- Max MW loss

These metrics answer a different question from EUE or LOLH. They describe the worst severity observed among the solved scenarios rather than the expected severity.

## ELCC
ELCC reuses the RA workflow to compare reliability metrics before and after adding or deactivating a resource. ELCC runs use the same outage samples and filtered joint scenario structure as the underlying RA run.

This means ELCC inherits the same representative-day structure, renewable scenario treatment, and metric definitions as the base RA calculation.

## DLOL
DLOL is reported from the RA solution outputs and provides an alternative way to summarize reliability impacts. It should be interpreted alongside the standard RA metrics rather than as a replacement for them.

## Practical Validation Questions
When reviewing RA metrics, it is useful to check:
- whether the weighted metrics move in the expected direction when renewable scenarios are added
- whether annual and day-group metrics are internally consistent
- whether regional metrics align with the locations where scarcity is observed in the dispatch results
- whether ELCC and DLOL outputs are consistent with the underlying RA metric behavior
