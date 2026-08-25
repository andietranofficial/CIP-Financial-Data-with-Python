# CIP-Financial-Data-with-Python

## Architecture

Full three-student group workflow, as documented in `StudC_Presentation.pptx`:

![Workflow diagram — Student A, Student B, Student C](assets/workflow-diagram.jpeg)

- **Student A** scrapes/collects S&P 500 tickers (Wikipedia), the 500 most-active non-member tickers (Eoddata.com), and qualitative/ESG data (Yahoo Finance), then joins them into two reference files: `Biber_Martin_StudentA_tickers_sp500_stage` (+ `..._SP500_ESG_stage`) and `Biber_Martin_StudentA_non_members_stage`.
- **Student C (this repo)** extracts stock price history via `yfinance`, merges it with Student A's reference data, and ranks/groups it to identify top tickers — handing two of those ticker lists off to Student B.
- **Student B** scrapes SEC/EDGAR insider-transaction data for those tickers and hands the results back.
- **Student C** then merges Student B's insider data (and its own ranking data) into the four `Result_Question_0N` answers, which are finally loaded into MariaDB.

## Business Overview

This is the "Student C" contribution to a multi-student coursework project analyzing S&P 500 stock behavior. The project pulls a year of stock price history for S&P 500 members, S&P 500 non-members, and the SPY benchmark ETF, enriches it with sector and ESG (Environmental/Social/Governance) scores, and cross-references it with insider trading activity (SEC/EDGAR filings) to answer questions about who profits from high-performing, high-ESG stocks, and whether S&P 500 membership correlates with trading activity.

## Aim

Build the middle stage of a three-student ETL pipeline in Python: extract stock data via the Yahoo Finance API, merge it with peer-provided reference data, rank/filter it to hand off to a teammate's insider-trading scrape, fold her results back in to answer four research questions with pandas, and persist every source, staged, and result dataset into a MariaDB database for downstream querying.

## Dataset Description

**Upstream (produced by teammates, consumed by this repo):**
- **Student A** — `Biber_Martin_StudentA_tickers_sp500_stage` (S&P 500 tickers + `Sector`/`Sub-Industry`, joined with qualitative/ESG data scraped from Yahoo Finance into `..._SP500_ESG_stage` with `ESG Score`, `ENVRisk`, `SocialRisk`, `GovRisk`) and `Biber_Martin_StudentA_non_members_stage` (500 most-active non-S&P-500 tickers by volume, from Eoddata.com).
- **Student B** — `Tallner_Kirsten_StudB_top5_esg_stage` and `..._top5_sp500_stage`, insider-transaction data scraped from SEC/EDGAR (via Selenium/BeautifulSoup) for the ticker shortlists this repo sends her; columns `A` (shares acquired) and `D` (shares disposed) per person/company/ticker.

**Extracted directly in this repo:**
- **Yahoo Finance** (via `yfinance`): daily OHLCV stock data (`2020-12-31` to `2021-10-29`) for ~500 S&P 500 members, ~500 non-members, and the `SPY` ETF.

All of the above live outside this repository, in a sibling `../Tran_Dao_Data/` directory (referenced by the notebooks as their relative data path), not committed here.

## Approach

1. **Extract** (`Code/Tran_Dao_StudC_extract_code.ipynb`) — download raw daily price history for S&P 500 members, non-members, and SPY via `yfinance`, using Student A's ticker lists as input.
2. **Merge** (`Code/Tran_Dao_StudC_transform_code.ipynb`) — combine the three raw price datasets into `stock_data_stage`, deliberately corrupt a copy for a data-cleaning exercise, clean it, then merge in Student A's sector/ESG data to produce `combined_stock_data_stage`.
3. **Manipulate 1 / hand off** (`Code/Tran_Dao_StudC_Question_01.ipynb`, `Question_02.ipynb`) — sort, rank, and group to produce `top5_esg_stage`, `top5_sp500_stage`, and `sp500_ranking_stage`; send the top-5 ticker lists to Student B for her insider-transaction scrape.
4. **Manipulate 2 / analyze** (`Question_01.ipynb`–`Question_04.ipynb`) — merge Student B's returned insider-transaction data (and the ranking/combined stage data) to answer the four research questions:
   - Q1: top 3 insider buyers among the 5 best-ESG-ranked, sector-outperforming S&P 500 IT stocks.
   - Q2: top 3 insider buyers and sellers among the 5 best-performing S&P 500 stocks.
   - Q3: whether S&P 500 members or the 500 most-active non-members have higher average trading volume.
   - Q4: how many of the 20 best-ESG-ranked stocks beat the SPY benchmark.
   Each answer is written out as `Result_Question_0N`.
5. **Load** (`Code/Tran_Dao_StudC_createDB_mariaDB.ipynb`, then `Code/Tran_Dao_StudC_loadData_mariaDB.ipynb`) — create the `CIP` MariaDB database, then load every source/stage CSV and each `Result_Question_0N` into its own table for querying via SQL.

## Tech Stack

- **Language:** Python (Jupyter notebooks)
- **Libraries:** pandas, numpy, yfinance, matplotlib, missingno, pymysql, sqlalchemy
- **Database:** MariaDB / MySQL
- **Upstream teammates' tools** (not run from this repo): Selenium, BeautifulSoup — used by Student A and Student B for web-scraped inputs/insider data.

## Setup

```bash
pip install pandas numpy yfinance matplotlib missingno pymysql sqlalchemy jupyter
```

Populate a sibling `../Tran_Dao_Data/` directory with the required input files (ticker lists, ESG data from Student A), then run the notebooks in order:

```bash
jupyter notebook Code/Tran_Dao_StudC_extract_code.ipynb
jupyter notebook Code/Tran_Dao_StudC_transform_code.ipynb
jupyter notebook Code/Tran_Dao_StudC_Question_01.ipynb
jupyter notebook Code/Tran_Dao_StudC_Question_02.ipynb
jupyter notebook Code/Tran_Dao_StudC_Question_03.ipynb
jupyter notebook Code/Tran_Dao_StudC_Question_04.ipynb
jupyter notebook Code/Tran_Dao_StudC_createDB_mariaDB.ipynb
jupyter notebook Code/Tran_Dao_StudC_loadData_mariaDB.ipynb
```

## Key Learning Takeaways

- Practiced a full data-cleaning workflow: missing values, duplicates, outlier detection (box plots), dtype/date-range constraints, and value-inconsistency fixes.
- Coordinated a real cross-team data handoff: sending derived ticker shortlists to a teammate and folding her scraped results back into the analysis.
- Wrote pandas DataFrames into a relational database (MariaDB) via SQLAlchemy and verified the load with `SHOW TABLES` / `read_sql`.
