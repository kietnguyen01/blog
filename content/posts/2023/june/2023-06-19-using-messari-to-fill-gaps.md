---
title: "Using Messari to Fill Gaps"
date: 2023-06-19T16:48:43+07:00
series: "Remittance Fees Analysis"
draft: false
tags: ["Collecting"]
---

[Project Link](https://github.com/kietnguyen01/Remittance-Fees-Analysis)

After obtaining the initial data from CoinGecko API and BitInfoCharts, I discovered that some columns were missing a significant amount of data. To fill these gaps, I turned to Messari, as they had a lot of historical data for various crypto. However, the site did not provide an easy way for me to programmatically pull or scrape the data as a free user, so I had to manually copy and paste the data into Google Sheets. Since I was going to be using Google Sheets frequently, I used this tool for the tasks here.

## Filling XLM

BitInfoCharts did not have the data for XLM available for me to scrape, so I needed to obtain the number of transactions, fees, and transaction values from Messari. However, I encountered a problem, as the average transaction fees were all \\$0. This was because the total fees for XLM were so low that they might as well be zero when rounding up to the cent. For example, the total fees for 01/01/2019 were \\$1 for 159,128 transactions, which worked out to \\$0.0000063 per transaction, not even a rounding error. Since the zeros added nothing to the data, I left out that column.

![XLM Data](https://i.imgur.com/IZf72iy.jpg)

There were also only 1450 rows, with 11 dates missing. I couldn’t fill those dates with actual values because there wasn't another data source available. I would have to fill in those dates with null values later.

## Getting Real Volume

**Edited 06/08/23** Change crypto data from CoinGecko to Messari for volumes.

I was working with the volume given by CoinGecko and I realized that the numbers were just too inflated. I went searching and learned that crypto has a massive wash trading problem. Fortunately, Messari had a metric called `Real Volume` with this description:

> It is well known that many exchanges conduct wash trading practices in order to inflate trading volume. They are incentivized to report inflated volumes in order to attract traders. "Real Volume" refers to the total volume on the exchanges that we believe with high level of confidence are free of wash trading activities. However, that does not necessarily mean that the volume reported by other exchanges is 100% wash trades. As such, the Messari "Real Volume" applies a penalty to these exchanges to discount the volume believed to come from wash trading activity.

While the way Messari calculated this wasn't transparent, I still thought that their numbers were better than the values from CoinGecko. An overinflated trading volume would cause issues with my analysis later. That's why I decided to change all of the `24h_volume` to the data from Messari instead.

### Data Formatting

The Messari data was formatted differently from my other data. I changed all of the columns' data types to Number and removed all \\$ signs. The next step was to expand all the abbreviated numbers with the correct number of zeros using a formula.

> `=ArrayFormula(IF(ISBLANK(B2:B), "", LEFT(B2:B, LEN(B2:B)-1)*IF(RIGHT(B2:B)="K", 1000, IF(RIGHT(B2:B)="M", 1000000, IF(RIGHT(B2:B)="B", 1000000000, 1)))))`

I applied the same formula to other columns where abbreviated numbers were present. I scanned the result and found something interesting; there were some extreme outliers for total fees on several dates. The average of total fees was around \\$200, but the highest fee was up to \\$15,000, or 75 times the average. It was likely that these fees were inaccurate, but there was no way for me to confirm for sure.

There was also another problem; there were only 1450 rows, which meant that 11 dates were missing from this. I would have to use `pl.upsample()` in Python later to fill those dates with null values.

![XLM Excel](https://i.imgur.com/ATvQzva.jpg)

Then I created a new column for XLM average fees. I calculated it by dividing the total fees by the transaction count. The numbers were extremely small, which was expected. With that, I could delete the total fees column. I also didn’t need the transaction volume column because I already had the 24H volume in the `api_data` file, so I deleted that column.

## Missing Date for XMR

When I called the CoinGecko API for XMR in a previous step, I noticed that the data only contained 1460 rows. I filled in that date with null values. To find the exact missing date, I used a regular expression. It was on `2019/05/29`.

![Regex Find](https://i.imgur.com/y2AQ9Hy.jpg)

While Messari had prices and market caps, it did not have the total volumes for XMR. This meant I needed to use another site for the data. Fortunately, CoinMarketCap was another price aggregator similar to CoinGecko. I obtained the data and pasted it in the blank cells.

| date | prices | market_caps | total_volumes |
| --- | --- | --- | --- |
| 2019-05-29 | $93.99 | 1,598,837,764 | 109,958,292 |

## Finding Blank Cells

Some of the data I scraped from BitInfoCharts were missing some rows. I didn’t know which sheets or specific columns to check, so I used the function `=COUNTBLANK()` to count the number of blank cells for each column in each sheet.

![https://i.imgur.com/7xS3JmA.jpg](https://i.imgur.com/7xS3JmA.jpg)

I repeated this step for each sheet and found where the blank cells were. Then I used a couple more formulas:

> `=MIN(FILTER(ROW(E2:E1462),ISBLANK(E2:E1462)))` # First blank cell
> `=MAX(FILTER(ROW(A2:A1462),ISBLANK(A2:A1462)))` # Last blank cell

This allowed me to know where the gaps were. The two sheets that had blank cells were LTC and ZEC.

![https://i.imgur.com/AunEe8p.jpg](https://i.imgur.com/AunEe8p.jpg)

![https://i.imgur.com/bQVBxWl.jpg](https://i.imgur.com/bQVBxWl.jpg)

However, when I went to Messari to get the missing data, I encountered multiple problems. The ZEC fees had negative values, incorrect abbreviated value, and extreme outliers. The ZEC transaction values were also very different from BitInfoCharts. After some consideration, I chose not to use Messari here. It would be better for me to impute the missing rows with the current data.

## Finishing Up

With the difficult part out of the way, I quickly ran through the last few steps to finish up:

- Renaming columns.
- Deleting columns with unnecessary data.
- Checking if Messari had data for missing project columns.
- Creating new sheets for the new data.
- Formatting the new data.