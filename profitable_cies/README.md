# Analysis report
## Task

Identify American publicly traded companies that had a profit margin of at least 20% for the fiscal year 2022, for further analysis.

## Data used

Financial Statement Data Sets for the first quarter of 2023 on the U.S. Securities Aand Exchange Commission's [website](https://www.sec.gov/dera/data/financial-statement-data-sets)

The dataset contains multiple text files that have been converted to the csv format. 

Two files in the dataset have been used for this analysis:
- the file containing the financial data, called ```num.csv```
- the file containing the information of companies that submitted documents, called ```sub.csv```

## Data cleaning

Two values have been collected from the financial data file:
- Revenues
- ProfitLoss

The ```Revenues``` entry is only for revenues coming fromm the operation of the company. This leaves out extraordinary revenues such as revenues from asset liquidation. That's why some entries in the cleaned file have no revenues, but still have profits.

The ```ProfitLoss``` entry is not the exclusive entry designated profits, and not every company uses it.
