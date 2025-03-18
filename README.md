# SafeGraph-PublicCompanies

This repository contains code for academic researchers to link SafeGraph Brand data to commonly used public company unique identifiers. This notebook is designed to be a starting point, and is subject to several limitations ([Limitations](#limitations)).

## Data and Requirements

### Data
* SafeGraph Brand data — Available via [Dewey Data](https://app.deweydata.io/products/5a88b56e-1155-4784-bba6-c0f4afd9b6ef/package)
* Compustat Fundamentals Annual — Available via WRDS
    * *This notebook requires WRDS API access, but can be easily modified to read in local files*
    * [*Using Python on WRDS Platform*](https://wrds-www.wharton.upenn.edu/documents/1443/wrds_connection.html)
* SEC EDGAR Index Files — Scraped from SEC Website

### Requirements & User Modifications
* This notebook uses typical Python libraries, including pandas, numpy, and fuzzywuzzy.
* This notebook requires WRDS API access, but can be easily modified to read in local files
* Required code modifications from the user:
    * path to the user's local directory
    * the user's name and email address (for SEC EDGAR scraping)
* Optional code modifications from the user:
    * `min_score` — The minimum fuzzy match score to accept
        * *The default score in the function `find_best_match` is 85*

## Matching Methodology
1. Separate the SafeGraph Brand data into two datasets:
    1. `sgTickerData` — Contains all SafeGraph Brand data that has a `STOCK_SYMBOL` (ticker)
    2. `sgNoTickerData` — Contains all SafeGraph Brand data that does not have a `STOCK_SYMBOL` (ticker)
2. Match SafeGraph Brand data to Compustat data using the `STOCK_SYMBOL` (ticker)
3. If there is no ticker match, or if there is no CIK (SEC unique identifier), we use SEC EDGAR data to fuzzy match the SafeGraph Brand to the SEC Company Name.
4. For SafeGraph Brands that do not have a `STOCK_SYMBOL` (ticker), we use SEC EDGAR data to fuzzy match the SafeGraph Brand to the SEC Company Name.
5. Check if there are any SafeGraph Brands without a match that have a `PARENT_SAFEGRAPH_BRAND_ID` *with* a public company match.

## Notebook Outline
1. Notebook Setup (import packages, set directory, define functions) 
2. Identify unique `gvkey, tic, conm, cik` observations in Compustat
3. Identify `SAFEGRAPH_BRAND_ID` observations that have a `STOCK_SYMBOL` (ticker)
4. Match SafeGraph Data with Compustat Data on Ticker (`STOCK_SYMBOL` and `tic`)
5. Fuzzy Match with SEC data *(Because some Compustat companies do not have CIK (SEC unique identifier), and because some SafeGraph brands may not have a Compustat ticker, we next use SEC EDGAR data to match on company name.)*
    1. Pull EDGAR data from SEC for 2019 - 2024, identify reporting companies
    2. Fuzzy match `BRAND_NAME` to `Company Name` — Default minimum score is 85 (out of 100)
6. Fuzzy Match SafeGraph Brands without a `STOCK_SYMBOL` to SEC Company Name
7. Merge matches with all data on `PARENT_SAFEGRAPH_BRAND_ID`



## Limitations

- This linking process relies on SafeGraph and Compustat identification of stock tickers. 
    - Tickers change over time as a result of events such as mergers, acquisitions, and company re-brands. 
    - Neither SafeGraph nor Compustat provide historical ticker information (i.e. the ticker identified by each data source is the ticker as of a particular point in time.)
        - Compustat ticker is the most current ticker for the company.
        - The exact date of SafeGraph's ticker identification is not known.
    - If a researcher is interested in a more complete and accuarate tracking of ticker changes overtime, there may be event data available from other sources (e.g. [NYSE](https://www.nyse.com/list/corporate-actions))

- How SafeGraph identifies parent companies is not known. Given the vast number of brands and sometimes complex holding company structures, it's possible that there are brands that are *not* identified with a public company that are in fact owned by a public company. The researcher should be aware of this possibility when planning their research design. 
    - For example, if a researcher is interested in looking at the foot traffic of public companies, and there were unidentified brand relationships, the foot traffic of the public company may be understated.

- SafeGraph does not take into account the time-series of parent company changes. 
    - For example, Tiffany & Co. (a publicly traded company) was acquired by LVMH in January 2021 — SafeGraph reflects the current structure, listing LVMH as the parent company of the Tiffany & Co. brand.

- This notebook utilizes fuzzy matching on company names. The threshold in the `find_best_match` function is set to 85 by default. The exact matching success rate has not been audited in detail for `match_type == 'company name'`. The researcher may modify this threshold considering their tolerance for matching errors.
