---
title: "Stablecoins EDA"
date: 2023-07-27T17:33:23+07:00
series: "Remittance Fees Analysis"
draft: false
tags: ["Exploring"]
---

[Project Link](https://github.com/kietnguyen01/Remittance-Fees-Analysis)

**Edited 29/08/23** Changed stablecoin data from CoinGecko to Messari for volumes.

The process for stablecoins EDA was similar to crypto EDA, so I converted the Polars dataframes to pandas and followed the same steps.

## The Stats

I used the `summary_stats()` function from `crypto_eda.py` to display the same statistics. It did exactly what I wanted, so I didn't need to write it again.

![Stablecoins Summary Stats](https://i.imgur.com/l21OvFZ.jpg)

The transaction fees for stablecoins were in a separate sheet.

![Stablecoin Fees Stats](https://i.imgur.com/RP4IdNS.jpg)

I used the same `fees_stats()` function, but instead of looping through the dictionary, I only needed to apply it once.

{{< highlight py >}}
def fees_stats(df: pd.DataFrame) -> None:
    """Calculate the standard deviation, skewness and kurtosis of the transaction fees"""
    # calculate and each statistic
    std = df['transaction_fees'].std()
    sk = df['transaction_fees'].skew()
    kt = df['transaction_fees'].kurt()
    # create a new dataframe
    stats_df = pd.DataFrame([[std, sk, kt]], columns=["std", "skew", "kurt"])
    stats_df = stats_df.rename(index={0: "transaction_fees"})
    # Print the dataframe as table
    print(tabulate(stats_df, headers='keys', tablefmt='fancy_grid', floatfmt='.3f'))
{{< / highlight >}}

![Stablecoins Fees Other Stats](https://i.imgur.com/L7YFOLY.jpg)

## Visualizations

### Fees Plots

With transaction fees, I used a line plot to see how the fees changed over time. I also created a box plot to check where most of the fees were.

{{< highlight py >}}
def fees_plots(df: pd.DataFrame) -> None:
    """Plot the transaction fees over time with a line plot and as a box plot"""
    # Create a figure with two subplots
    fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 5))
    # Group the data by week and calculate the mean of transaction fees
    df_weekly = df.groupby(pd.Grouper(key='date', freq='W'))['transaction_fees'].mean().reset_index()
    # Plot the monthly data with lineplot
    sns.lineplot(data=df_weekly, x='date', y='transaction_fees', label='Transaction Fees', ax=ax1)
    ax1.set_xlabel('Date')
    ax1.set_ylabel('Transaction Fees')
    ax1.set_title('Transaction Fees over Time')
    ax1.tick_params(axis='x', rotation=15)
    ax1.legend()
    # Plot the box plot on the second subplot
    sns.boxplot(data=df, y='transaction_fees', showfliers=False, ax=ax2)
    ax2.set_ylabel('Transaction Fees')
    ax2.set_title('Box Plot of Transaction Fees')
    # Adjust the spacing between the subplots
    plt.tight_layout()
    # Show the figure
    plt.show()
{{< / highlight >}}

![Stablecoin Fees Plots](https://i.imgur.com/Fr14DKv.jpg)

### Line Plots

To see how the volumes of different stablecoins changed over time, I wanted to create a line plot. This would let me see whether people were using some stablecoins more than others. I resampled the data to be monthly because the daily charts were too choppy to read clearly.

{{< highlight py >}}
def volumes_lineplot(dfs: dict) -> None:
    """Plot the 24 volumes of all stablecoins in a line plot"""
    # Create an empty dataframe to store the total volumes
    df_volumes = pd.DataFrame()
    # Loop and append each token's total volume
    for token in dfs.keys():
        if token != 'fees':
            df = dfs[token]
            df_volumes[token] = df['24h_volume']
    # Set date index and resample to weekly
    df_volumes.index = pd.date_range(start='2019-01-01', end='2022-12-31', freq='D', name='date')
    df_volumes = df_volumes.resample('W').mean()
    # Set plot parameters
    plt.figure(figsize=(14, 8))
    sns.lineplot(data=df_volumes)
    plt.xlabel('Date')
    plt.ylabel('24h Volume')
    plt.title('24h Volumes of Stablecoins (Weekly)')
    plt.legend(loc='upper left', bbox_to_anchor=(1, 1))
    plt.yscale('log')
    plt.show()
{{< / highlight >}}

![Old Stablecoins Volumes Plot](https://i.imgur.com/OIEw9Ew.jpg)

### Issues with Dates and Zeros

However, I noticed that both BUSD and DAI had missing dates. I went back to check the dataframes for those stablecoins and found out that they came out in the middle of 2019. This meant that the start dates for them were not `2019-01-01`. To fix this, I added some additional functions to `coingecko_api.py`. These functions checked the first date of the dataframe, and if it wasn't what I wanted, added a new row with that date. Then `fill_date()` filled in those missing dates.

{{< highlight py >}}
def create_new_row(date: datetime.date) -> pl.DataFrame:
    """Create a new row with the given date and null values for the other columns"""
    new_row = pl.DataFrame({
        'date': [date],
        'prices': [None],
        'market_caps': [None],
        'total_volumes': [None]
    })
    # Cast all columns except date to f64
    new_row = new_row.select([pl.col('date'), *[pl.col(c).cast(pl.Float64) for c in new_row.columns[1:]]])
    return new_row

def prepend_new_row(df: pl.DataFrame, start: str) -> pl.DataFrame:
    """Prepend a new row with the desired start date if the first date is not equal to the desired start date"""
    # Get the date series from the dataframe
    date_series = df['date']
    # Check if first date is equal to desired start date
    desired_date = datetime.strptime(start, '%Y/%m/%d').date()
    if date_series[0] != desired_date:
        # Create a new row with desired date
        new_row = create_new_row(desired_date)
        # Prepend new row to dataframe
        df = pl.concat([new_row, df])
    return df
{{< / highlight >}}

Another issue I also found was that BUSD had zeros for its market caps for many dates in the beginning. I wrote another function to address this.

{{< highlight py >}}
def replace_zeros_with_nulls(df: pl.DataFrame, column: str) -> pl.DataFrame:
    """Replace any zeros in the given column with null values"""
    # Get the series from the dataframe
    series = df[column]
    # Apply a lambda function to replace zeros with nulls
    series = series.apply(lambda x: None if x == 0 else x)
    # Replace the column in the dataframe with the modified series
    df = df.with_columns(series.alias(column))
    return df
{{< / highlight >}}

Then I changed `create_df()` accordingly.

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
    # Prepend a new row if needed
    df = prepend_new_row(df, start)
    # Fill in any missing rows
    df = fill_date(df)
    # Replace any zeros with nulls in every column except date
    for column in df.columns[1:]:
        df = replace_zeros_with_nulls(df, column)
    return df
{{< / highlight >}}

This provided me with a new plot that I wanted.

![New Stablecoins Volumes Plot](https://i.imgur.com/X4IPHa4.png)

### Scatterplots

The second visualization I wanted to see was scatterplots of market caps against volumes. Since the data for stablecoins was all about these two variables, I wanted to see their relationships.

{{< highlight py >}}
def mcaps_volumes_scatterplots(dfs: dict) -> None:
    """Plot a scatterplot of the market caps and total volumes of all stablecoins"""
    # Create a 2x4 grid of subplots
    fig, axes = plt.subplots(2, 4, figsize=(16, 8))
    # Loop and plot each token
    colors = sns.color_palette()
    for token, ax, color in zip(dfs.keys(), axes.flatten(), colors):
        if token != 'fees':
            df = dfs[token]
            sns.scatterplot(x='market_caps', y='24h_volume', data=df, ax=ax, color=color)
            ax.set_title(token)
    axes[-1, -1].remove()  # Remove the empty subplot
    fig.tight_layout()
    plt.show()
{{< / highlight >}}

![Stablecoins Scatterplots](https://i.imgur.com/phS4lff.png)

Stablecoins with higher market caps had more consistent volumes as their market caps increased. This was expected since USDT, USDC, and BUSD were backed by large players in the crypto industry, providing them with ample liquidity and usage in various places.

On the other hand, stablecoins with lower market caps experienced drop-offs in volumes as their market caps grew. These stablecoins may have experienced a surge in demand and supply during periods of high volatility or uncertainty in the crypto market, which could explain their higher volumes when their market caps were smaller. Alternatively, these stablecoins may have had insufficient reserves or inadequate disclosure practices, which could have impacted their credibility and trustworthiness as a stable store of value as they grew larger.