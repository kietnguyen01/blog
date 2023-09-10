---
title: "Merging Crypto Dataframes"
date: 2023-06-21T10:53:41+07:00
series: "Remittance Fees Analysis"
draft: false
tags: ["Collecting"]
---

[Project Link](https://github.com/kietnguyen01/Remittance-Fees-Analysis)

# Merging Crypto Dataframes

After obtaining data from Messari, the dataframes needed to be merged together. However, I encountered a few problems when trying to read in the new Excel files:

- Some of the number columns were `str` instead of `float`.
- Dataframes for `ltc`, `xlm`, and `zec` contained duplicated columns with all null values after import.
- The data type of `date` column was `str` for certain dataframes.

## Fixing Excel Import

To fix the column data types, I went back to Google Sheet to format all of the numeric columns specifically as `Number`. That allowed Polars to assign the data types that fit those columns best during import.

For the duplicated columns, they were not visible in Google Sheets. The simplest solution I used was creating new sheets and copying only the relevant data over. I also added a `2022/12/31` row to `bsv_mes` since the sheet didn’t have that date. This would help `pl.upsample()` on that dataframe because the function filled in dates between the first and last dates.

For the last problem of the `date` column with `str` type, I changed the parameter for `read_csv_options` that enforced the `dtypes` for specific column during import.

{{< highlight py >}}
def import_excel(path: str, names: list):
    """Create dict of dataframes from Excel sheets"""
    data = {}
    for name in names:
        data[name] = pl.read_excel(
            path, 
            sheet_name=name,
            read_csv_options={
                'infer_schema_length': 2000,
                'dtypes': {'date': pl.Date}
            }
        )
    return data
{{< / highlight >}}

## Import and Fill

Then I used `import_excel()` to read the Excel files into Python.

{{< highlight py >}}
# Read filled Excel sheets into dataframes
filled_api = import_excel('data/filled-api-data.xlsx', syms)
filled_scraped = import_excel(
    'data/filled-scraped-data.xlsx', 
    ['btc', 'xrp_bit', 'xrp_mes', 'doge', 'ltc', 'xmr', 'bsv_bit', 'bsv_mes', 'bch', 'xlm', 'zec', 'dash']
)
{{< / highlight >}}

However, some of the dataframes from the `filled_scraped_data.xlsx` had missing dates due to data availability issues from Messari. This would cause problems later when joining the dataframes on the date column. To fix this, I used the function `fill_date()` in `crypto_api.ipynb` that takes a dataframe as an input and returns a dataframe with missing date rows filled with null values.

{{< highlight py >}}
def fill_date(df: pl.DataFrame):
    """Fill in missing date row with null values"""
    df = df.set_sorted('date')
    df = df.upsample(time_column='date', every='1d')
    return df
{{< / highlight >}}

I applied the function to the dataframes that I knew had missing dates.

{{< highlight py >}}
# Fill missing dates in Messari data with null values
for name in ['xrp_mes', 'bsv_mes', 'xlm']:
    filled_scraped[name] = ca.fill_date(filled_scraped[name])
{{< / highlight >}}

## Complete XRP and BSV

XRP and BSV had two dataframes each: one from BitInfoCharts and one from Messari. The BitInfoCharts dataframe had a `transactions_count` column, while the Messari dataframe had `total_fees` and `average_transaction_value`. I wanted to combine these dataframes into one dataframe, then calculate the `average_transaction_fees` and drop `total_fees`.

{{< highlight py >}}
# Merge XRP and BSV dataframes
for name in ['xrp', 'bsv']:
    filled_scraped[name] = filled_scraped[f'{name}_bit'].join(filled_scraped[f'{name}_mes'], on='date', how='inner')
    # Calculate average_transaction_fees column
    filled_scraped[name] = filled_scraped[name].with_columns((pl.col("total_fees") / pl.col("transactions_count")).alias("average_transaction_fees"))
    # Drop total_fees column
    filled_scraped[name] = filled_scraped[name].drop('total_fees')
{{< / highlight >}}

## Merging API and Scraped Data

The last step here was to merge each dataframe from the `filled_api` dictionary with its corresponding dataframe from the `filled_scraped` dictionary. This would give me a complete dataset with all the columns for each cryptocurrency from both sources. Then I could export the combined data to a new Excel file.

{{< highlight py >}}
# Merge API and scraped data
combined_data = {}
for name in syms:
    combined_data[name] = filled_api[name].join(filled_scraped[name], on='date', how='inner')

# Create Excel file with combined data
file_path = os.path.join("data", "combined-data.xlsx")
create_excel(file_path, combined_data)
{{< / highlight >}}