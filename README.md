# UChicago-Household_Finance_Research-Project

## Overview

This project examines whether the COVID-era housing boom (2019–2021) restored the housing wealth losses inflicted by the Great Recession (2007–2009), and whether any such recovery was distributed equitably across income groups and local housing markets. The analysis uses IPUMS USA American Community Survey (ACS) microdata for owner-occupied households and proceeds in two complementary parts:

- **Part 1 (State-Level):** Compares housing value changes across low-income (bottom 20%), middle-income (20–80%), and high-income (top 20%) homeowners by state and tests for a systematic "payback" relationship between the two episodes.
- **Part 2 (PUMA-Level):** Identifies local markets most "decimated" by the Great Recession and determines what fraction had recovered to their pre-crisis values by 2021.

---


## Data Source

**IPUMS USA — American Community Survey (ACS) Public-Use Microdata**

| Detail | Description |
|---|---|
| Provider | [IPUMS USA](https://usa.ipums.org/usa/), Minnesota Population Center |
| Survey | American Community Survey (ACS), 1-year samples |
| Years | 2007, 2009, 2019, 2021 *(2020 excluded due to COVID-related collection issues)* |

### Variables Used

| Variable | Description |
|---|---|
| `YEAR` | Survey year |
| `STATEFIP` | State FIPS code |
| `PUMA` | Public Use Microdata Area code |
| `GQ` | Group quarters status (kept codes 1 and 2 — households only) |
| `OWNERSHP` | Tenure status (kept code 1 — owner-occupied) |
| `HHINCOME` | Total household income |
| `VALUEH` | Self-reported housing unit value (primary outcome) |
| `HHWT` | Household survey weight |

### Downloading the Data

1. Create an account at [https://usa.ipums.org/usa/](https://usa.ipums.org/usa/)
2. Select the variables listed above for ACS 1-year samples for years **2007, 2009, 2019, and 2021**
3. Request and download the extract as a **CSV file**
4. Save the file as `usa_00010.csv` (or update the filename in the `.Rmd` file accordingly)

### Sample Restrictions Applied in Code

- `GQ %in% c(1, 2)` — households only, excluding group quarters
- `OWNERSHP == 1` — owner-occupied units only
- `HHINCOME != 9999999` — drops missing/not-in-universe income
- `VALUEH` not in `{0, 9999998, 9999999}` — drops missing/not-in-universe housing values

After cleaning, the working sample contains **8,955,228 household-year observations** across 51 states and 1,185 unique PUMA codes.

---

## Software Requirements

### R Version

R version **4.0 or higher** is recommended. The code was developed and tested in a standard R environment.

### Required R Packages

| Package | Version | Purpose |
|---|---|---|
| `tidyverse` | Data manipulation (`dplyr`, `tidyr`) and visualization (`ggplot2`) |
| `Hmisc` | Weighted quantile computation (`wtd.quantile`) |
| `broom` | Tidying regression model outputs |

### Installing Packages

Run the following in your R console before knitting the `.Rmd` file:

```r
install.packages(c("tidyverse", "Hmisc", "broom"))
```

---

## Running the Analysis

### Step 1 — Set Up the Working Directory

Open `Rajat_Kanti_Paul_Household_Finance_Final_Project_Code_File.Rmd` and update the `setwd()` call in the `load-data` chunk to match the local path where `usa_00010.csv` is saved:

```r
setwd("~path/to/project/folder")
hhdata <- read.csv("file_name.csv")
```

### Step 2 — Knit the Document

In RStudio, open the `.Rmd` file and click **Knit → Knit to PDF**, or run from the R console:

```r
rmarkdown::render("Rajat_Kanti_Paul_Household_Finance_Final_Project_Code_File.Rmd")
```

This will execute all code chunks sequentially and produce a PDF with all results, tables, and figures embedded.

---

## Code Workflow

The `.Rmd` file is structured as a single end-to-end pipeline. The chunks run in order as follows:

```
Raw IPUMS CSV
      │
      ▼
[1] setup          — Load libraries; set global chunk options
[2] load-data      — Read usa_00010.csv into R
[3] clean-data     — Apply sample restrictions (GQ, OWNERSHP, HHINCOME, VALUEH)
[4] data-summary   — Print observation counts and unique state/PUMA counts
      │
      ├─── PART 1: STATE-LEVEL ANALYSIS ──────────────────────────────
      │
[5]  state-income-cutoffs   — Compute weighted 20th/80th income percentiles
                               by state × year using wtd.quantile()
[6]  state-income-groups    — Assign each household to Low / Middle / High
                               income group based on state-specific cutoffs
[7]  state-winsorize        — Winsorize VALUEH at 1st/99th percentile within
                               state × year × income group
[8]  state-aggregate        — Compute weighted mean housing value by
                               state × year × income group
[9]  state-changes          — Pivot wide; compute % change 2007–2009 and
                               2019–2021; flag recovery (val2021 >= val2007)
[10] state-recovery-summary — Tabulate share recovered and average changes
                               by income group (→ Table 1 in paper)
[11] state-recovery-bar     — Bar chart: recovery share by income group
                               (→ Figure 1 in paper)
[12] linear-models          — Estimate OLS: %ΔValue(07–09) ~ %ΔValue(19–21)
                               separately for Low, Middle, High income groups
[13] linear-results         — Print regression summaries (→ Table 2 in paper)
[14] linear-plot            — Scatter plot with per-group regression lines
                               (→ Figure 2 in paper)
      │
      └─── PART 2: PUMA-LEVEL ANALYSIS ───────────────────────────────
      │
[15] puma-setup        — Create unique state_puma_id = STATEFIP + PUMA (zero-padded)
[16] puma-winsorize    — Winsorize VALUEH at 1st/99th percentile within PUMA × year
[17] puma-aggregate    — Compute weighted mean housing value by PUMA × year
[18] puma-changes      — Pivot wide; compute % change 2007–2009 and 2019–2021
[19] decimated-define  — Define "decimated" PUMAs as bottom quartile of 2007–2009
                          % change (cutoff ≈ −9.67%); flag recovery
[20] decimated-summary — Tabulate recovery rate and average changes by
                          decimation status (→ key statistics in Section 5.3)
[21] puma-scatter      — Scatter plot: Great Recession vs. COVID change, PUMAs
                          colored by decimation status (→ Figure 3 in paper)
[22] puma-bar          — Bar chart: recovery share for decimated vs. other PUMAs
                          (→ Figure 4 in paper)
```

### Key Analytical Decisions

| Decision | Implementation |
|---|---|
| Income group thresholds | Computed **within each state × year** to reflect local economic conditions |
| Winsorization | Applied at **1st/99th percentile** within each geographic unit × year × group cell |
| Unique PUMA ID | Constructed as `paste0(STATEFIP, sprintf("%05d", PUMA))` to handle state-dependency |
| "Decimated" definition | Bottom **25th percentile** of 2007–2009 PUMA-level % change |
| Recovery criterion | 2021 weighted mean value **≥** 2007 weighted mean value (nominal, not inflation-adjusted) |
| Survey weights | All means and quantiles use `HHWT` household weights throughout |
