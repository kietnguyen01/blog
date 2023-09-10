---
title: "Using Owlracle API"
date: 2023-06-28T17:25:28+07:00
series: "Remittance Fees Analysis"
draft: false
tags: ["Collecting"]
---

[Project Link](https://github.com/kietnguyen01/Remittance-Fees-Analysis)

[Owlracle](https://owlracle.info/eth) was a great platform that provided a free API to access historical transaction and gas fees for different blockchains. The [documentation](https://owlracle.info/docs) was straightforward to understand. 

![Owlracle Documentation](https://i.imgur.com/TBtisj7.jpg)

## Getting the Dates

There was a small limitation with Owlracle's API, as it only allowed a maximum of 1000 candles. This meant that I could only get up to 1000 days of data with each call. However, 1000 days was an arbitrary number to use in my code, so I decided to use 365 days instead. This made organizing my data easier with individual years. To create a list of dates that were 365 days apart, I just needed to provide a `start` and `end` date.

{{< highlight py >}}
def dates_between(start: str, end: str) -> list:
    """Return a list of first dates of each year, given a start and end"""
    start_date = datetime.strptime(start, '%Y/%m/%d')
    end_date = datetime.strptime(end, '%Y/%m/%d')
    date_list = []

    # Check if the start date is the first day of the year
    if start_date.month == 1 and start_date.day == 1:
        current_date = start_date
    else:
        # If not, use the first day of current year
        current_date = start_date.replace(month=1, day=1)

    while end_date > current_date:
        # Add current date to the list of dates and replace the month and day to 1
        date_list.append(current_date)
        current_date = current_date.replace(month=1, day=1)
        # Add one year from the current date and replace the month and day to 1
        current_date = current_date.replace(year=current_date.year + 1, month=1, day=1)
        
    # Add first day of next year for end date
    date_list.append(end_date.replace(year=end_date.year + 1, month=1, day=1))

    # Return list of dates as strings
    return [date.strftime('%Y/%m/%d') + ' 00:00:00' for date in date_list]
{{< / highlight >}}

I was careful when calling this API and added the `00:00:00` part to the date. This ensured that the dates were correct, and my time zone did not affect the call.

## Making the Call

Calling the API was straightforward. The documentation provided the necessary arguments for the request. A couple of things to note here were:

- I put the API key in the `.env` file so that it did not get exposed when committing to GitHub. I just loaded the variable with the `load_dotenv()` function.
- This API also used Unix time, similar to the CoinGecko API.

{{< highlight py >}}
import requests
from dotenv import load_dotenv
from datetime import datetime, timezone

# Load environment variables from .env file
load_dotenv(".env")

def unix_time(date: str) -> float:
    """Take in date in string format and return UNIX timestamp float"""
    stamp = datetime.strptime(date, '%Y/%m/%d %H:%M:%S')
    return int(stamp.replace(tzinfo=timezone.utc).timestamp())

def api_call(dates_list: list) -> dict:
    """Make a call to the Owlracle API and return historical gas JSON"""
    key = os.getenv('OAPI')
    results = {}    # Dict of JSON

    # Loop over the list of dates in pairs using zip
    for start_date, end_date in zip(dates_list, dates_list[1:]):
        unix_start = unix_time(start_date)
        unix_end = unix_time(end_date)
    
        # Make the API call with the start and end dates
        res = requests.get(f'https://api.owlracle.info/v4/eth/history?apikey={key}&from={unix_start}&to={unix_end}&candles=365&timeframe=1d&txfee=true')
        # Add the response JSON to the results dictionary with the year as the key
        year = datetime.strptime(start_date, '%Y/%m/%d %H:%M:%S').year
        results[year] = res.json()
        
    return results
{{< / highlight >}}

## Extract Data and Create Dataframes

For example, if I called the API from `2019/01/01` to `2020/01/01`, the returned JSON would be like below.

![Owlracle Candle](https://i.imgur.com/JnLuD8B.jpg)

**Edited 05/07/23**. Previously, I was trying to get the average fees and prices. What I did was calculating the mean of highs and lows. But this turned out to be a mistake because all of the values were extremely high. What worked better was simply extracting the closing values.

{{< highlight py >}}
def extract_json(json_data: dict) -> dict:
    """Extract the txFee and the gasPrice from the JSON data, calculate the average
    Return a dict containing three lists: dates, transaction_fees, and gas_prices"""
    result = {}
    dates = []
    transaction_fees = []
    gas_prices = []
    for year, data in json_data.items():
        for candle in data['candles']:
            # Extract txFee and gasPrice from candle
            fee = candle['txFee']['close']
            price = candle['gasPrice']['close']
            
            # Only add in data from the current year
            date = candle['timestamp'].split('T')[0]
            if date.startswith(str(year)):
                dates.append(date)
                transaction_fees.append(fee)
                gas_prices.append(price)
        
    # Add the sorted dates, fees, and prices lists to the result dict
    result["date"] = sorted([datetime.strptime(date, '%Y-%m-%d').date() for date in dates])
    result["transaction_fees"] = [x for _, x in sorted(zip(dates, transaction_fees))]
    result["gas_prices"] = [x for _, x in sorted(zip(dates, gas_prices))]
    
    return result
{{< / highlight >}}

The functions above extracted `gasPrice`, `txFee`, `timestamp` from the JSON. Then I appended the data into their separate lists so that they could be turned into Polars dataframes easily later. There were also several missing dates in the data, so I used the same `fill_date()` function from `coingecko_api.ipynb`.

{{< highlight py >}}
def fill_date(df: pl.DataFrame):
    """Fill in missing date row with null values"""
    df = df.set_sorted('date')
    df = df.upsample(time_column='date', every='1d')
    return df

def create_df(start: str, end: str) -> pl.DataFrame:
    """Create a Polars DataFrame from the Owlracle API data between the given start and end dates"""
    between = dates_between(start, end)
    json = api_call(between)
    extracted = extract_json(json)
    df = pl.DataFrame(extracted)
    return fill_date(df)
{{< / highlight >}}