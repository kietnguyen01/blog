---
title: "Crypto EDA"
date: 2023-07-21T14:29:38+07:00
series: "Remittance Fees Analysis"
draft: false
tags: ["Exploring"]
---

[Project Link](https://github.com/kietnguyen01/Remittance-Fees-Analysis)

Before conducting EDA, I encountered a significant problem. Although Polars dataframes are speedy, they are not ideal for exploring data. This is because many packages that are commonly used with pandas do not work with Polars. I had to use workarounds and hacks to get some packages to work with Polars, which made performing EDA frustrating. After a few days of struggling, I decided to convert the Polars dataframes back to pandas. This way, I don’t waste too much time on hacks that might not work in the future.

## The Stats

The first step was to display the summary statistics. To make the output more visually appealing, I used the `tabulate` package.

{{< highlight py >}}
from tabulate import tabulate

def summary_stats(dfs: dict) -> None:
    """Print the summary statistics for each dataframe"""
    for name, df in dfs.items():
        df = df.drop('date', axis=1)
        print(f'>>> {name.upper()}')
        print(tabulate(df.describe(), headers='keys', tablefmt='fancy_grid', floatfmt='.3f'))
{{< / highlight >}}

![Crypto Summary Statistics](https://i.imgur.com/XDhDpUM.jpg)

The summary stats provided me with an overview of the basic values of the tokens.

The other stats I wanted to check were the standard deviation, skewness, and kurtosis.

{{< highlight py >}}
def fees_stats(dfs: dict) -> None:
    """Calculate the standard deviation, skewness and kurtosis of the transaction fees for each token"""
    stats_list = []
    for name, df in dfs.items():
        # calculate and append each statistic to the list
        std = df['average_transaction_fees'].std()
        sk = df['average_transaction_fees'].skew()
        kt = df['average_transaction_fees'].kurt()
        stats_list.append((name, std, sk, kt))
    # create a new dataframe from the list
    stats_df = pd.DataFrame(stats_list, columns=['token', 'std', 'skew', 'kurt'])
    # Print the dataframe as table
    print(tabulate(stats_df, headers='keys', tablefmt='fancy_grid', floatfmt='.3f'))
{{< / highlight >}}

![Crypto Other Stats](https://i.imgur.com/vRM6VBw.jpg)

Some interesting things to note about the transaction fees stats are:

- BTC and ZEC had the highest standard deviations. Their fees deviated the most from the mean.
- LTC and DASH had the highest skewness. Their fees were heavily skewed to the right, meaning a lot of outliers.
- LTC and DASH also had the highest kurtosis. Their fees were highly peaked compared to other tokens.

## Visualizations

### Box Plots

The first visualization I wanted to see for the transaction fees was the box plot.

{{< highlight py >}}
import pandas as pd
import seaborn as sns

def get_fees_column(df: pd.DataFrame) -> str:
    """Get transaction fees column name from dataframe"""
    cols = df.columns
    if 'median_transaction_fees' in cols:
        return 'median_transaction_fees'
    elif 'average_transaction_fees' in cols:
        return 'average_transaction_fees'
    else:
        return ''

def fees_df(dfs: dict) -> pd.DataFrame:
    """Create a dataframe of transaction fees"""
    fees_data = pd.DataFrame()
    for token, df in dfs.items():
        col_name = get_fees_column(df)
        fees_data[token] = df[col_name]
    return fees_data

def fees_boxplots(dfs: dict) -> None:
    """Plot the box plots of transaction fees for each column"""
    df = fees_df(dfs)
    fig, axes = plt.subplots(nrows=len(df.columns), ncols=1, figsize=(10, 15))
    colors = sns.color_palette()
    for col, ax, color in zip(df.columns, axes, colors):
        # Plot box plot without outliers
        sns.boxplot(x=df[col], ax=ax, showfliers=False, color=color)
        ax.set_title(col.upper())
        ax.set_xlabel('')
    fig.suptitle('Crypto Transaction Fees')
    fig.subplots_adjust(top=0.85)
    plt.tight_layout()
    plt.show()
{{< / highlight >}}

![Crypto Fees Box Plots](https://i.imgur.com/dgPVZQJ.jpg)

For the box plots, I chose not to plot the outliers. This is because the outliers made the data extremely right-skewed. In general, the IQR showed that most of the transaction fees were skewed toward the left of the distribution. Most tokens also had medians that were to the left, which means that the data were skewed to the right. This right-skewed distribution was also shown with the right whiskers extending far into the right.

### Correlation Heatmaps

The next visualization I used was the correlation heatmap to check the correlation between different variables in a dataframe.

{{< highlight py >}}
def corr_heatmaps(dfs: dict) -> None:
    """Plot the correlation heatmap of variables in each dataframe"""
    fig, axes = plt.subplots(nrows=10, ncols=1, figsize=(12, 60))
    axes = axes.flatten()  
    for name, df in dfs.items():
        # Get index position of token name
        i = list(dfs.keys()).index(name)
        ax = axes[i]
        df = df.drop('date', axis=1)
        corr_matrix = df.corr()
        sns.heatmap(corr_matrix, annot=True, ax=ax, cmap='mako_r')
        ax.set_title(name.upper())
        ax.set_xticklabels(ax.get_xticklabels(), rotation=15)
    plt.tight_layout()
    plt.show()
{{< / highlight >}}

![Crypto Correlation Heatmaps](https://i.imgur.com/p4Zh4vV.jpg)

The variables I paid attention to were the transaction fees, transactions count, and the transaction values. The interesting thing was that some tokens did not have strong correlation between all three variables. What that tell me was that for some tokens, transaction fees did not play an important role in how much users were transacting.

### Scatterplots

**Edited 02/08/23**

To further confirm the relationship between transaction fees and 24h volume, I used the scatterplot. This plot told me the direction and strength of the relationship. I also plotted a regression line to show me a prediction of where the relationship was heading. The transaction value variable was used as the hue to see whether more expensive transactions had higher values than cheaper transactions.

{{< highlight py >}}
def get_value_column(df: pd.DataFrame) -> str:
    """Get transaction value column name from dataframe"""
    cols = df.columns
    if 'median_transaction_value' in cols:
        return 'median_transaction_value'
    elif 'average_transaction_value' in cols:
        return 'average_transaction_value'
    else:
        return ''

def fees_scatterplots(dfs: dict) -> None:
    """Plot the scatterplots of transaction fees against 24h volume and set hue to transaction value"""
    # Create two subplots per row
    fig, axes = plt.subplots(nrows=len(dfs) // 2 + len(dfs) % 2, ncols=2, figsize=(15, 20))
    # Flatten the axes array
    axes = axes.flatten()
    colors = sns.color_palette()
    for name, df in dfs.items():
        # Get index position of token name
        i = list(dfs.keys()).index(name)
        ax = axes[i]
        fees_col = get_fees_column(df)
        volume = df['24h_volume']
        value_col = get_value_column(df)
        sns.regplot(x=volume, y=fees_col, data=df, ax=ax)
        if value_col:
            sns.scatterplot(x=volume, y=fees_col, data=df, ax=ax, color=colors[i], hue=value_col)
        else:
            sns.scatterplot(x=volume, y=fees_col, data=df, ax=ax, color=colors[i])
        ax.set_title(name.upper())
        ax.set_xlabel('24h_volume')
        ax.set_ylabel(fees_col)
    fig.suptitle('Transaction Fees vs 24h Volume')
    plt.tight_layout()
    plt.show()
{{< / highlight >}}

![Crypto Scatterplots](https://i.imgur.com/AYdM4Jd.png)

There is a positive correlation between transaction fees and 24h volume for most cryptocurrencies. This means that as the 24h volume increases, the transaction fees tend to increase as well. However, there are a few cryptocurrencies that do not follow this trend. For example, XRP and DOGE have relatively low transaction fees, even when the 24h volume is high.

The hue variable in the scatterplots allows us to see the relationship between transaction fees and 24h volume for different values of the transaction value. For example, the plots for BTC and LTC show that the transaction fees tend to be higher for larger transactions. This is because larger transactions require more computational resources to process, and miners can charge higher fees for processing these transactions.