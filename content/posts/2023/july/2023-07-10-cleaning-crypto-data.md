---
title: "Cleaning Crypto & Stablecoin Data"
date: 2023-07-10T14:30:02+07:00
series: "Remittance Fees Analysis"
draft: false
tags: ["Cleaning"]
---

[Project Link](https://github.com/kietnguyen01/Remittance-Fees-Analysis)

I started the cleaning process working on the crypto data. The first step was to create a new file called `clean_crypto.ipynb`. Then I moved the code from `crypto.ipynb` that performed the cleaning to this file.

{{< highlight py >}}
def combine_xrp_bsv(scraped: dict) -> dict:
    """Calculate average_transaction_fees for XRP/BSV dataframes. Merge API and Messari dataframes for those projects"""
    # Merge XRP and BSV dataframes
    for name in ['xrp', 'bsv']:
        scraped[name] = scraped[f'{name}_bit'].join(scraped[f'{name}_mes'], on='date', how='inner')
        # Calculate average_transaction_fees column
        scraped[name] = scraped[name].with_columns((pl.col("total_fees") / pl.col("transactions_count")).alias("average_transaction_fees"))
        # Drop total_fees column
        scraped[name] = scraped[name].drop('total_fees')
    return scraped

def merge_api_scraped(api_data: dict, scraped_data: dict, names: list) -> dict:
    """Merge API and scraped data into a single dictionary of dataframes"""
    # Clean scraped dataframes
    for name in ['xrp_mes', 'bsv_mes', 'xlm']:
				scraped_data[name] = winsorize_df(scraped_data[name])
        scraped_data[name] = ca.fill_date(scraped_data[name])
    combine_xrp_bsv(scraped_data)

    # Merge API and scraped data
    combined = {}
    for name in names:
        combined[name] = api_data[name].join(scraped_data[name], on='date', how='inner')
    return combined
{{< / highlight >}}

## Winsorize Outliers

One thing I noticed when checking the crypto data was that the Messari data contained some extreme outliers. Some of the values were more than 20 times the median. While that data could’ve been correct given the volatility of crypto, it wasn’t helpful for my analysis. To remove their effect, I decided to winsorize the data by replacing the lowest/highest 5% of data with the data in those percentiles.

{{< highlight py >}}
from scipy.stats.mstats import winsorize

def winsorize_df(df: pl.DataFrame, limits: tuple=(0.05, 0.05)) -> pl.DataFrame:
    """Winsorize Messari data to remove extreme outliers"""
    # Loop through the columns except the first one
    for col in df.columns[1:]:
        # Apply winsorization with 5% of the lowest/highest replaced
        col_values = df[col].to_numpy()
        win_values = winsorize(col_values, limits=limits)
        new_col = pl.Series(name=col, values=win_values)

        # Replace the original column with the winsorized column
        df = df.drop(col)
        df = df.with_columns(**{col: new_col})
    return df
{{< / highlight >}}

## Impute Missing Values

The next step after winsorizing the data was to impute the missing values. I did not want to use the mean because there were some outliers that influenced the data. After some research, I decided to use MICE. This imputation method ran multiple iterations of ML models to predict the missing values. This method is good at handling multiple types of missing data and can provide a robust result.

{{< highlight py >}}
def mf_impute(df: pd.DataFrame) -> pd.DataFrame:
    """Return imputed pandas dataframe using MICE"""
    kernel = mf.ImputationKernel(
        df,
        datasets=3,
        save_all_iterations=True,
        random_state=123
    )
    # Run the MICE algorithm for 3 iterations on each of the datasets
    kernel.mice(3)
    return kernel.complete_data()

def impute_dfs(dfs: dict) -> dict:
    """Impute missing values in dataframes of a dict"""
    # Convert Polars to pandas
    pandas_dfs = {k: df.to_pandas() for k, df in dfs.items()}

    # Impute each dataframe
    imputed = {}
    for k, df in pandas_dfs.items():
        date_series = df['date']  # Get date column as a series
        df_no_date = df.drop('date', axis=1)
        df_imputed = mf_impute(df_no_date)

        # Concatenate the date back to the imputed dataframe
        imputed[k] = pd.concat([date_series, df_imputed], axis=1)

    # Convert pandas to Polars
    return {k: pl.from_pandas(df) for k, df in imputed.items()}
{{< / highlight >}}

## Clean Stablecoin

The great thing about the imputing and winsorizing functions was that they could also be used for stablecoin data. Since both datasets shared a lot of similarities, I only need to change the cutoff parameter so that only the lowest/highest 1% are winsorized.

{{< highlight py >}}
# Import combined data
syms.append('fees')
combined = gf.import_excel(
    combined_path,
    syms
)

# Impute column missing values with miceforest
imputed = cc.impute_dfs(combined)

# Winsorize fees column
imputed['fees'] = cc.winsorize_df(imputed['fees'], (0.01, 0.01))
{{< / highlight >}}