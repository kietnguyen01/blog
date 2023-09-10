---
title: "Remittance EDA"
date: 2023-08-01T17:18:53+07:00
series: "Remittance Fees Analysis"
draft: false
tags: ["Exploring"]
---

[Project Link](https://github.com/kietnguyen01/Remittance-Fees-Analysis)

For the remittance datasets, I used only the following dataframes: `country-lookup`, `cost-to-country`, `cost-from-country`, `inflows`, and `outflows`. I collected and cleaned the other datasets just in case, but after some consideration, I decided not to use them since I didn't think they would contribute to my analysis.

## Summary Stats

First, I merged the datasets with the lookup table containing country codes, regions, and income levels. Then I used the `pivot_table()` function to create a summary table that showed the average remittance by year and grouping column.

```python
def merge_lookup(df, lookup_df):
    """Merge dataframe to lookup dataframe by country code"""
    return pd.merge(df, lookup_df[["Country Code", "Region", "Income"]], on="Country Code")

def pivot_table(df, group):
    """Pivot dataframe by income or region and return transposed table"""
    flow_year_cols = df.columns[1:-2] # exclude Country Code, Region and Income
    pivot_df = pd.pivot_table(data=df, index=group, values=flow_year_cols, aggfunc="mean")
    pivot_df.index.name = "Year"
    pivot_df = pivot_df.transpose()
    return pivot_df
```

The `summary_stats()` function takes the pivot table created earlier and prints the summary statistics using the `tabulate` package.

```python
def summary_stats(df: pd.DataFrame) -> None:
    """Print the summary statistics for dataframe"""
    print(tabulate(df.describe(), headers='keys', tablefmt='fancy_grid', floatfmt='.3f'))
```

![Cost to Country by Income](https://i.imgur.com/E0jezrs.jpg)

![Cost to Country by Region](https://i.imgur.com/PRnv9Kw.jpg)

But looking at tables was not very informative, so I used line plots to tell better stories.

## Visualization

`plot_lineplot()` function plots a line plot of the DataFrame by year and grouping column on the given axis. It creates a visualization that shows the trend of the average remittance by year and grouping column. Since the aggregated data by Region has a higher value, I added a check to change the y label to `Ten Billion $`.

```python
def plot_lineplot(pivot_df, group, ax):
    """Plot lineplot for pivot dataframe by income or region on given axis"""
    sns.lineplot(data=pivot_df, ax=ax)
    ax.set_title(f"Average Remittance {group}")
    ax.set_xlabel("Year")
    if "Income" in group:
        y_label = "Billion $"
    elif "Region" in group:
        y_label = "Ten Billion $"
    ax.set_ylabel(y_label)
    ax.legend(loc="upper left", title=group.split()[-1][:-1], bbox_to_anchor=(1, 1))
    # Set x-axis tick positions and rotate labels
    ax.set_xticks(ax.get_xticks())
    ax.set_xticklabels(ax.get_xticklabels(), rotation=25)
```

### Cost to/from Country

To make the comparison easier to see, I plotted the line plots that were segmented by Income and Region separately.

![Cost to/from Country by Income](https://i.imgur.com/MtaIl6t.png)

By income, the plots revealed that the price of sending and receiving remittance to low-income countries had dropped more than in other income groups over the last ten years. This could mean that remittance services had become more affordable and accessible in low-income countries, which was a good thing for their economic growth and poverty reduction efforts. But remittance to low-income countries still cost more than in other income groups, so there was still work to be done.

![Cost to/from Country by Region](https://i.imgur.com/1TrUePy.png)

By region, the plots told me that the average cost of remittance differed a lot depending on the region, and some regions had seen bigger changes in cost than others over time. For instance, the plot showed that sending and receiving money in Sub-Saharan Africa was the most expensive compared to other regions, and the cost had gone back up in recent years. There were also large changes to remittance cost during the pandemic, but the overall trend seemed to be going down for many regions.

### Remittance Inflow/Outflow

![Inflow/outflow by Income](https://i.imgur.com/4tQoOx3.png)

By income, middle-income countries actually received more money than low-income countries. They appeared to be putting the remittance money back into their own countries, demonstrated by the very low outflows. While high-income countries also received a lot of remittance, they led the world in remittance outflow.

![Inflow/outflow by Region](https://i.imgur.com/eiqjQ2h.png)

By region, two groups stood out in each chart. For inflow, South Asia appeared to be receiving the most remittance by a wide margin. This was expected since India was probably skewing the dataset toward this group. For outflow, North America was the primary group that sent remittance abroad. There was a steady uptrend for this region in recent years but had sharply increased during the pandemic time.