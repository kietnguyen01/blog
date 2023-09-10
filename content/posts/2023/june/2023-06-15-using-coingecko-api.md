---
title: "Using Coingecko API"
date: 2023-06-15T15:25:37+07:00
series: "Remittance Fees Analysis"
draft: false
tags: ["Collecting"]
---

[Project Link](https://github.com/kietnguyen01/Remittance-Fees-Analysis)

I used the CoinGecko API first to get historical market data for each project. Since I wasn’t a paid user, I didn’t have access to an API key. Good thing someone already wrote a [Python wrapper](https://github.com/man-c/pycoingecko) for the API. I could use the package in my code to call the public API directly.

## API Call

A few things to note about the API: 

- The token IDs were from this [Google Sheet](https://docs.google.com/spreadsheets/d/1wTTuxXt8n9q7C4NDXqQpI3wpKu1_5bGVmP9Xz0XGSyU/edit#gid=0).
- It took UNIX timestamps as arguments.
- It returned a nested dictionary of JSON data with timestamps.

This meant I needed three functions to get the data from CoinGecko:

1. Convert a date string into UNIX timestamp.
2. Perform the API call.
3. Extract data points from returned JSON, without the timestamps.

{{< highlight py >}}
import polars as pl
from time import sleep
from pycoingecko import CoinGeckoAPI
from datetime import datetime, timezone

cg = CoinGeckoAPI()

def unix_time(date: str) -> float:
    """Take in date in MM/DD/YYYY format and return UNIX timestamp float"""
    stamp = datetime.strptime(date, '%m/%d/%Y')
    return stamp.replace(tzinfo=timezone.utc).timestamp()

def api_call(name: str, currency: str, start: str, end: str) -> dict:
    """Make a call to the CoinGecko API and return the JSON response"""
    unix_start = unix_time(start)
    unix_end = unix_time(end)
    data = cg.get_coin_market_chart_range_by_id(
        id=name, 
        vs_currency=currency, 
        from_timestamp=unix_start, 
        to_timestamp=unix_end
    )
    sleep(5)
    return data

def extract_json(data: dict) -> dict:
    """"Extracts data from JSON and returns a dictionary without the timestamps"""
    data_points = {}
    timestamps_added = False  # Flag for adding timestamps to dict
    for key, value in data.items():
        if not timestamps_added:
            data_points['timestamps'] = [x[0] for x in value]
            timestamps_added = True
        # Use list comprehension to get the second element of each sublist in value
        data_points[key] = [x[1] for x in value]
    return data_points
{{< / highlight >}}

### Polars Dataframe

Now I could create a dataframe from the data dictionary. I used Polars instead of the standard Pandas. While this dataset was unlikely to be large enough to cause a bottleneck in Pandas, I wanted to use something new for this project. There were some inconveniences to using a new package, but the speed increase compensated for it.

{{< highlight py >}}
def create_df(name: str, currency: str, start: str, end: str) -> pl.DataFrame:
    """Create Polars dataframe from API data"""
    json = api_call(name, currency, start, end)
    data = extract_json(json)
    df = pl.DataFrame(data)
    # Apply lambda function to the timestamps column and create a date column
    date_series = df['timestamps'].apply(lambda x: datetime.fromtimestamp(x/1000).date())
    df = df.with_columns([date_series.alias('date')])
    df = df.drop('timestamps')
    # Reorder date column to the first position
    df = df.select(['date', *df.columns[:-1]])
    return df

def create_crypto_dict(symbols: list, ids: list, currency: str, start: str, end: str) -> dict:
    """Create a dictionary of Polars dataframes for each crypto id"""
    crypto_dict = {}
    for symbol, id in zip(symbols, ids):
        df = create_df(id, currency, start, end)
        crypto_dict[symbol] = df
    return crypto_dict
{{< / highlight >}}

All of the functions above were placed inside the `crypto_fns.ipynb` file. I used a different file to define the IDs, variables, and run the code.

{{< highlight py >}}
%run crypto_fns.ipynb
import ipynb.fs.full.crypto_fns as cf

# List of crypto symbols and IDs
syms = ['btc', 'xrp', 'doge', 'ltc', 'xmr', 'bch', 'xlm', 'algo', 'bsv', 'zec', 'dash']
cids = ['bitcoin', 'ripple', 'dogecoin', 'litecoin', 'monero', 'bitcoin-cash', 'stellar', 'bitcoin-cash-sv', 'zcash', 'dash']

# Variables for API call
currency = 'usd'
start_date = '01/01/2019'
end_date = '12/31/2022'

# Create dictionary of crypto dataframes
crypto = cf.create_crypto_dict(syms, cids, currency, start_date, end_date)
{{< / highlight >}}

## Missing Date

Originally, I wanted to create a dataframe with a `date` column from `01/01/2019` to `12/31/2022`. Then I was going to add the extracted data to that dataframe. But that didn’t work because my code crashed when it tried to add the data for Monero.

{{< highlight sh >}}
ShapeError: unable to add a column of length 1460 to a dataframe of height 1461
{{< / highlight >}}

It turned out the data from CoinGecko for Monero was missing one date. Since I didn’t know which date, I changed my `extract_json()` to include the timestamps. Then I converted the timestamps to datetime after with a `lambda` function. This meant the dataframe for XMR had 1460 rows, but one missing date was not a big deal. 

I wrote a function that fill in the missing row with null values.

{{< highlight py >}}
def fill_date(df: pl.DataFrame, rows: int):
    """Fill in missing date row with null values"""
    if len(df) < rows:
        df = df.set_sorted('date')
        df = df.upsample(time_column='date', every='1d')
    return df
{{< / highlight >}}

And added this line to `create_df` function: `df = fill_date(df, 1461)`.

## Writing to Excel

The API calls and the functions took about a minute to run. I didn’t want to call the API every time I ran my code, so I wrote the extracted data to an Excel file.

{{< highlight py >}}
import os
import xlsxwriter

# File path to data folder
file_path = os.path.join("data", "crypto.xlsx")
# Write crypto data to Excel file
with xlsxwriter.Workbook(file_path) as workbook:
    for key, df in crypto.items():
        worksheet = workbook.add_worksheet(key)
        df.write_excel(workbook=workbook, worksheet=key)
{{< / highlight >}}