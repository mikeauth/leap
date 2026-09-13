# Power-network disruption data construction

This is a **synthetic teaching dataset**, not operational power-system data. The files are `power_substations.csv` (144 substations) and `power_links.csv` (344 undirected links). `generate_datasets.py` is a separate, staff-facing reproducibility script; neither student notebook contains data-generation code. NumPy's `default_rng(2042)` fixes the random draw sequence.

## Timeline and column definitions

The link topology, load ratio and equipment age are known when the initial disruption is recorded. One substation is already interrupted at that point. `interrupted_later` records a simulated interruption over the following 24 hours only for the other substations. A blank for the initially interrupted substation means *ineligible*, not *no later interruption*. There is no simulated dynamic cascade, protective control or electrical load flow.

| File and column | Meaning |
| --- | --- |
| Substations: `substation_id` | Unique identifier S001 to S144; for joining, never a predictor. |
| Substations: `load_ratio` | Simulated pre-disruption load relative to a nominal limit, uniformly drawn from 0.42 to 0.94, rounded to three decimals. |
| Substations: `equipment_age_years` | Simulated equipment age, uniform integer 2 to 40 inclusive. |
| Substations: `initially_interrupted` | 1 for central substation S066, 0 for all others. |
| Substations: `interrupted_later` | Later binary outcome drawn only for the 143 initially functioning substations; blank for S066. There are 39 later positives in this fixed release. |
| Links: `substation_a`, `substation_b` | The two endpoints of a pre-disruption physical link; direction and weight are not represented. |

## Graph and target rule

Create a 12 by 12 grid graph with four-neighbour links, then choose 80 of its 121 square cells without replacement. Add one of the two possible diagonals to each chosen cell, at random. This creates local cross-links and varied clustering values while keeping the graph small and connected. Save each undirected link once. Degree counts links touching a substation. Local clustering coefficient is the proportion of possible links **among its direct neighbours** that are present.

For generating the later target only, standardise each of the four arrays below over **all 144 substations**, using its population mean and standard deviation, after rounding `load_ratio`. The data generator uses these simulated log-odds for substation *i*:

`-1.04 + 0.70*z(load_ratio_i) + 0.52*z(equipment_age_years_i) + 0.46*z(degree_i) + 0.75*z(local_clustering_i)`

Convert log-odds to probability with the logistic function and independently draw a Bernoulli outcome for every substation except S066. Assign a blank later outcome to S066. This construction rule deliberately creates possible associations with network position; it does **not** imply that the first failure travels over links. Student models must fit their own scaling on training observations only.

## Interpretation limits

The graph is stylised, its links have no impedance or capacity, and interruptions are independent draws conditional on a constructed probability. Added features may yield mixed held-out results in this one sample. No model score proves failure propagation or usefulness for real-world grid operation. The initial interruption sets the time at which the prediction is posed but has no causal role in the outcome-generation rule.

