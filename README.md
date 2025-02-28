# healthcare_transparency

This repository contains my submission to the take home exercise. The original instructions are [here](https://github.com/serif-health/takehome)

## Contents
- README.md: this file with an overview of the repo
- healthcare_transparency.ipynb: notebook with code well documented
- tic_extract_20250213.csv: payer sample data (the code will download it if not present)
- hpt_extract_20250213.csv: hospital sample data (the code will download it if not present)
- combined_hpt_tic.csv: combined data set (the code will generate it)

## Context

We are provided two data sets about health transparency: payers and hospitals. Both should contain similar information but it turns out that this is not the case. Aligning and merging them is not trivial and it's the purpose of this exercise.

## Approach

The notebook follows this process:
- Explore data
    - Look at unique values
    - Identify combinations of payer x provider x code in both data sets
- Rename and consolidate schemas and values
- Use NPI lists to remove entries with not enough NPI's
- Aggregate rates at the payer x provider x code level for each table
- Join tables on payer x provider x code.
- Calculate the ratio of the median rates from the payers vs hospitals tables as a measure of how aligned the data is after all the work


### Data exploration and cleaning

First we take a look at the data sets. We confirm that they overlap with 3 payers, 3 providers and 3 billing codes. But it also shows the need of lots of data cleaning.

Cleaning in both data sets:
- Delete columns with a single value.
- For each combination of payer x provider x code, there are multiple entries in both data sets (especially in the payer data set) and the rates still vary significantly (it's not uncommon that by more than 2x).
- Schemas have differing namings that we consolidate.

Cleaning in hospitals data set:
- We correct typos in the `raw_code` column.
- There are multiple `description` for a single `raw_code`. We assume that they are all equivalent and not "variants" of a billing code.
- It contains additional payers compared to the payers data set so we filter those out.
- It lacks EIN information. We google the EIN's present in the payers data set to match with the hospital names.

Cleaning in payers data set:
- It doesn't have insurance plan information.
- Payer names have multiple spellings (eg united healthcare, united, uhc, UHC). We unify the names needed for this exercise.

### Data integration

They key question is how to match combinations of payer x provider x code between the two data sets. It would be trivial if both data set had one entry per combination, but this is not the case, with the payer data set having a large number of entries but with differing rates. 

We use `taxonomy_filtered_npi_list` to remove entries in the payer data set. We notice that the list of individual NPI's can have wildly different lenghts. It seems reasonable to assume that of all the results for a combinations of billing_code x payer x hospital, the one that covers more providers (NPI) from that facility is the code that will actually be used, instead of billing different rates for different NPIs. Therefore, we only keep entries with a long list of NPI's (in particular, more than half of the longest list found for that combination of code x payer x hospital). 

After this filtering, we aggregate all entries at the code x payer x hospital level and aggregate rates, to find the minimum, maximum and the median rates.

We finally combine the two data sets and compare the median rates from the payers vs the hospitals data sets. We define a delta variable as the ratio of both, so delta=1 when both data sets coincide. Of all combinations in the data, delta has a distribution around 1 but it extends from 1/100x to 10x as shown in a plot in the notebook, which means that there is still a significant dispersion and disagreement between both.

## Instructions 

To run the code:
1. Copy the notebook in this repo to your local machine. Either clone the whole repo or simply download the notebook file.
1. Execute the notebook (run all cells, no need to do any edits at all)
- The two input csv files will be downloaded automatically (if they are not already present in the same folder as the notebook).
- The notebook will output the file `combined_hpt_tic.csv`.
- The notebook is well documented step-by-step, this README provides an overview.
- No additional python packages are needed

This notebook runs in a fraction of a second.

