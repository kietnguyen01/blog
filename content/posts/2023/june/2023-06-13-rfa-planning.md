---
title: "RFA Planning"
date: 2023-06-13T16:25:52+07:00
series: "Remittance Fees Analysis"
draft: false
tags: ["Planning"]
---

[Project Link](https://github.com/kietnguyen01/Remittance-Fees-Analysis)

Let’s dive into the planning process for this project.

## 1. Question

What is the cost of remittance for crypto, stablecoins, and remittance services, and how do they compare against each other?

## 2. Preparation

### Definitions

From my research, there are two types of digital assets that can be used as mediums of exchange: 

- **Cryptocurrencies (crypto)**: designed to operate outside the traditional financial system and relies purely on supply and demand to maintain its value.
- **Stablecoins**: designed to provide fast and cheap transactions using the blockchain while maintaining their value by pegging themselves to a stable real asset like the US dollar.

As such, it's necessary to separate these two assets in my analysis.

**Remittance services** are financial institutions, such as banks and money transfer operators (MTO), that help transfers money between people in different countries.

### Identifying Targets

Before collecting data, I need to determine which crypto and stablecoins to research. I'll use [CoinMarketCap](https://coinmarketcap.com/) to focus on the largest crypto and stablecoins, limiting my selection to projects in the top 100 that are specifically categorized as a medium of exchange.

![Medium of Exchange Tokens](https://i.imgur.com/gfOr5JX.jpg)

## 3. Collect Data

### Cryptocurrencies

[CoinGecko API](https://www.coingecko.com/en/api/documentation) is a simple way to access historical market data as it provides a free API that's sufficient for my needs. I'll obtain price, market cap, and 24H volume data from CoinGecko.

![CoinGecko API](https://i.imgur.com/YvqmCuY.jpg)

To obtain other data, such as the number of transactions, transaction value, and transaction fees, I'll use [BitInfoCharts](ttps://bitinfocharts.com/). This site provides an interactive graph for each statistic, but I'll need to scrape those data points as it doesn't have a way to download the data.

![BitInfoCharts](https://i.imgur.com/wF8gBDu.jpg)

[Messari](https://messari.io/) provides a lot of historical data, but as a free user, I don't have access to much of that data. Additionally, there's no easy way to programmatically extract data from the site. I'll have to manually copy the data into an Excel sheet and import it into Python later.

![Messari](https://i.imgur.com/7GGZxl3.jpg)

### Stablecoins

I can also use CoinGecko API to obtain price, market cap, and 24H volume data for stablecoins. However, the costs of stablecoin transfers depend on the protocol that the stablecoin is on. For example, to approximate the cost of a stablecoin transfer on Ethereum, I would use [Ethereum gas price](https://etherscan.io/chart/gasprice) on that day. 

![Ethereum gas price](https://i.imgur.com/Zv9r9O2.jpg)

Unfortunately, some block explorers like [BscScan](https://bscscan.com/) require a paid API to access historical gas prices. I'll need to spend additional time finding a way to solve this problem, such as getting the data directly from the blockchain of each protocol.

### Remittance Services

Fortunately, data on remittance services has been around for a long time. The World Bank's [Remittance Prices Worldwide](https://remittanceprices.worldbank.org/) site contains the data I'm looking for, and they even publish regular reports with key findings on remittances.

![Remittance Prices Worldwide](https://i.imgur.com/VDtIK2J.jpg)

## 4. Clean Data

### Crypto & Stablecoins

I'll spend a significant amount of time cleaning up the various data types obtained from crypto and stablecoins. The data types include raw HTML from scraping, JSON from API calls, and Excel sheets from manual collection.

### Remittance Services

This data is likely going to be in Excel format. There might be some formatting peculiarities depend on how the Excel is created. But I think cleaning this type of data will be much easier than the ones above.

## 5. Explore Data

Once the data is clean, I'll explore it to identify patterns or outliers that may need to be addressed before proceeding with the analysis. I can use various methods to explore the data, such as plotting histograms and density plots, computing summary statistics like the mean and standard deviation, and plotting scatter plots and heat maps to look for correlations between variables.

## 6. Analysis

I plan to perform descriptive and predictive analytics for this project.

### Descriptive

I can analyze the historical data for each remittance type to understand the trends, patterns, and insights from past data. I can identify the factors that influence remittance fees, such as market cap, trading volume, or transaction fees. I can also compare remittance fees across different types and time periods to identify the ones that offer the lowest fees.

### Predictive

To predict future remittance fees, I can build a regression model or use time-series analysis based on the historical data and other relevant variables. One interesting approach could be using machine learning for this type of analysis.

## 7. Interpret Results

By the end of the analysis, I hope to identify a recommended remittance type. However, this result should be taken with a grain of salt.