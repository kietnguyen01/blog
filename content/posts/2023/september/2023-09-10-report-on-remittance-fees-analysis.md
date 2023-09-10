---
title: "Report on Remittance Fees Analysis"
date: 2023-09-10T13:48:11+07:00
series: "Remittance Fees Analysis"
draft: false
tags: ["Report"]
---

[Project Link](https://github.com/kietnguyen01/Remittance-Fees-Analysis)

## Summary

This report summarizes the process and findings of a data analytics project that aims to compare the remittance fees of different cryptocurrencies and stablecoins with traditional methods. The project was documented in 15 blog posts by Kiet Nguyen, a data analyst and blogger. The main steps of the project were:

- Planning the project scope, objectives, and questions
- Selecting the cryptocurrencies and stablecoins to analyze
- Collecting and merging data from various sources, such as CoinGecko, BitInfoCharts, Messari, OwlRacle, and World Bank
- Cleaning and exploring the data using Python and pandas
- Performing the analysis using descriptive statistics and visualization techniques
- Answering the research questions and drawing conclusions

The main findings of the project were:

- Cryptocurrencies have lower remittance fees than traditional methods, especially for large amounts
- Stablecoins have lower volatility than cryptocurrencies, but higher remittance fees
- The remittance fees of cryptocurrencies and stablecoins vary depending on the exchange rate, network congestion, and transaction speed
- The remittance fees of traditional methods vary depending on the sending and receiving countries, the service provider, and the payment method

The report concludes that cryptocurrencies and stablecoins offer a viable alternative to traditional methods for remittance, but they also have some challenges and limitations that need to be addressed.

## Introduction

Remittance is the transfer of money from one country to another, usually by migrant workers who send money to their families or friends in their home countries. According to the World Bank, remittances are an important source of income for many developing countries, accounting for more than 10% of GDP in some cases. However, remitting money across borders can be costly and inefficient, as there are often high fees, long delays, and complex procedures involved.

Cryptocurrencies and stablecoins are digital assets that use blockchain technology to enable peer-to-peer transactions without intermediaries. Cryptocurrencies, such as Bitcoin and Ethereum, have their own native tokens that fluctuate in value according to supply and demand. Stablecoins, such as Tether and DAI, are pegged to a fiat currency or a basket of assets to maintain a stable value. Both cryptocurrencies and stablecoins claim to offer faster, cheaper, and more transparent remittance services than traditional methods.

This report aims to compare the remittance fees of different cryptocurrencies and stablecoins with traditional methods using data from various sources. The report also explores the factors that affect the remittance fees of each method and answers some research questions related to remittance.

## Data Collection

The data for this project was collected from various sources using web scraping and API calls. The sources were:

- [CoinGecko API](https://www.coingecko.com/en/api) - A free and comprehensive cryptocurrency API that provides market data, exchange rates, historical data, and more.
- [Messari](https://messari.io/) - A platform that offers crypto research, data, and tools for professionals, investors, and institutions.
- [BitInfoCharts](https://bitinfocharts.com/) - A service that offers interactive charts and indicators for Bitcoin and other cryptocurrencies.
- [Owlracle](https://owlracle.info/eth) - An open-source gas oracle running predictions for multiple blockchain networks.
- [World Bank](https://data.worldbank.org/) - Open global development data from the World Bank.
- [KNOMAD](https://www.knomad.org/data/remittances) - a brain trust for the global migration community.

The data was collected for 10 cryptocurrencies: BTC, XRP, DOGE, LTC, XMR, XLM, BCH, ALGO, BSV, ZEC, DASH and 7 stablecoins: USDT, USDC, BUSD, DAI, TUSD, USDP, GUSD. The data was collected for the period from January 1st 2019 to December 31st 2022.

The data collection process was documented in detail in the following blog posts:

- [Using CoinGecko API](https://kietnguyen.xyz/posts/2023/june/2023-06-15-using-coingecko-api/)
- [Scrape BitInfoCharts](https://kietnguyen.xyz/posts/2023/june/2023-06-17-scrape-bitinfocharts/)
- [Using Messari To Fill Gaps](https://kietnguyen.xyz/posts/2023/june/2023-06-19-using-messari-to-fill-gaps/)
- [Merging Crypto Dataframes](https://kietnguyen.xyz/posts/2023/june/2023-06-21-merging-crypto-dataframes/)
- [Using OwlRacle API](https://kietnguyen.xyz/posts/2023/june/2023-06-28-using-owlracle-api/)
- [World Bank & KNOMAD Datasets](https://kietnguyen.xyz/posts/2023/july/2023-07-04-world-bank-knomad-datasets/)

## Data Cleaning

The data collected from different sources had different formats, structures, and quality issues. Therefore, the data cleaning process involved:

- Renaming columns
- Dropping duplicates
- Handling missing values
- Converting data types
- Reshaping data frames
- Merging data frames
- Filtering outliers
- Standardizing units

The data cleaning process was documented in detail in the following blog post:

- [Cleaning Crypto & Stablecoin Data](https://kietnguyen.xyz/posts/2023/july/2023-07-10-cleaning-crypto-data/)

## Data Exploration

The data exploration process involved:

- Describing the data using summary statistics, such as mean, median, standard deviation, minimum, maximum, and quartiles
- Visualizing the data using plots, such as histograms, boxplots, scatterplots, line charts, and bar charts
- Analyzing the data using correlation coefficients, hypothesis testing, and regression models

The data exploration process was documented in detail in the following blog posts:

- [Crypto EDA](https://kietnguyen.xyz/posts/2023/july/2023-07-21-crypto-eda/)
- [Stablecoins EDA](https://kietnguyen.xyz/posts/2023/july/2023-07-27-stablecoins-eda/)
- [Remittance EDA](https://kietnguyen.xyz/posts/2023/august/2023-08-01-remittance-eda/)

## Data Analysis

The data analysis process involved:

- Answering the research questions that were defined in the planning stage of the project
- Comparing the remittance fees of different cryptocurrencies and stablecoins with traditional methods
- Identifying the factors that affect the remittance fees of each method
- Drawing conclusions and recommendations based on the findings

The data analysis process was documented in detail in the following blog post:

- [Performing The Analysis](https://kietnguyen.xyz/posts/2023/august/2023-08-21-performing-the-analysis/)

## Conclusion

This report presented the process and findings of a data analytics project that compared the remittance fees of different cryptocurrencies and stablecoins with traditional methods. The main conclusions of the project were:

- Cryptocurrencies have lower remittance fees than traditional methods, especially for large amounts
- Stablecoins have lower volatility than cryptocurrencies, but higher remittance fees
- The remittance fees of cryptocurrencies and stablecoins vary depending on the exchange rate, network congestion, and transaction speed
- The remittance fees of traditional methods vary depending on the sending and receiving countries, the service provider, and the payment method

The main recommendations of the project were:

- For remitters who value speed and low cost over stability, cryptocurrencies are a better option than traditional methods
- For remitters who value stability and transparency over speed and low cost, stablecoins are a better option than traditional methods
- For remitters who want to use cryptocurrencies or stablecoins for remittance, they should be aware of the risks and challenges involved, such as price fluctuations, regulatory uncertainty, technical complexity, and security issues
- For policymakers and regulators who want to facilitate and monitor remittance flows, they should adopt a balanced and flexible approach that recognizes the potential benefits and drawbacks of cryptocurrencies and stablecoins for remittance