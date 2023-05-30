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

The ```Revenues``` entry is only for revenues coming fromm the operation of the company. This leaves out extraordinary revenues such as revenues from asset liquidation. That's why some companies in the cleaned file have no revenues, but still have profits.

The ```ProfitLoss``` entry is not the exclusive entry designating profits, and is not used for every companies. Other entries could be used such as the ```NetIncomeLoss``` entry.

Here is the cleaning that has been done using the R programming language: 

```r
library(tidyr)

# Import of the file containing all numeric values in submitted documents to the SEC.
raw_data = read.csv('num.csv',sep = ',')

# Cleaning data
# Only keeping the Revenues and ProfitLoss values to calculate the profit margin.
raw_data <- subset(raw_data, tag %in% c("Revenues", "ProfitLoss"))

# Remove entries where coregistrants are mentioned to only deal with consolidated data
raw_data <- subset(raw_data, coreg == "")

# We only keep entries related to an annual report
raw_data <- subset(raw_data, qtrs == 4)

# We only keep information for the end of year 2022
raw_data <- subset(raw_data, ddate == "20221231")

# Rotate table so that Revenues and ProfitLoss for each entries are now columns 
rotated_data <- pivot_wider(raw_data, names_from = tag, values_from = value)

# We only keep entries for which Revenes and Profitloss are above 0.
# The company has to make money to be worth looking at
rotated_data <- subset(rotated_data, ProfitLoss >= 0)
rotated_data <- subset(rotated_data, Revenues >= 0)

# We remove columns that are not relevant for the analysis
rotated_data <- subset(rotated_data, select = -c(version, coreg, ddate, qtrs, uom, footnote))

# We convert Revenues and ProfitLoss values from strings to numeric values to be able
# to calculate the profit margin for each company.
rotated_data$ProfitLoss <- as.numeric(format(rotated_data$ProfitLoss))
rotated_data$Revenues <- as.numeric(format(rotated_data$Revenues))

# We create a profit_margin column based on the ProfitLoss and Revenues column
# We take the answer in percentage
rotated_data$profit_margin <- round((rotated_data$ProfitLoss / rotated_data$Revenues) * 100, 2)

# we only keep companies that had a profit margin of at least 20%
rotated_data <- subset(rotated_data, profit_margin >= 20)

# We import the csv file mapping the "adsh" value, which is the id value, to the company name
sub_df = read.csv('sub.csv',sep = ";")

# We will only need the entries for the id values that are contained in the "rotated_df"
unique_adsh <- unique(rotated_data$adsh)

# We filter out the entries where the adsh number is not in the unique_adsh list
sub_df <- sub_df[sub_df$adsh %in% unique_adsh, ]

# We only keep the adsh and name value in the dataframe. We are only interested in the name value
sub_df <- subset(sub_df, select = c(adsh, name))

# We merge the dataframe containing the financial data with the dataframe containing company names
# The merge is done on the adsh number
merged_dataframe <- merge(rotated_data, sub_df[, c("adsh", "name")], by = "adsh", all.x = TRUE)

# We order the merged dataframe on the profit margin value, in decreasing order
merged_dataframe <- merged_dataframe[order(merged_dataframe$profit_margin, decreasing = TRUE), ]

# we reorder the columns of the dataframe to have the name next to the adsh value
desired_order <- c("adsh", "name", "ProfitLoss", "Revenues", "profit_margin")

merged_dataframe <- merged_dataframe[, desired_order]

# We write the merged dataframe to a csv file
write.csv(merged_dataframe, file = "profit_margin.csv", row.names = FALSE, sep = ";")
```
