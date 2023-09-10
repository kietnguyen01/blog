---
title: "World Bank & KNOMAD Datasets"
date: 2023-07-04T16:28:11+07:00
series: "Remittance Fees Analysis"
draft: false
tags: ["Collecting"]
---

[Project Link](https://github.com/kietnguyen01/Remittance-Fees-Analysis)

Getting the data for traditional remittances was the simple part. Organizations like the World Bank and KNOMAD had collected a lot of data on this topic for years.

- **World Bank**
    - [Personal remittances, paid (current US$)](https://data.worldbank.org/indicator/BM.TRF.PWKR.CD.DT?view=chart)
    - [Personal remittances, received (current US$)](https://data.worldbank.org/indicator/BX.TRF.PWKR.CD.DT?view=chart)
    - [Personal remittances, received (% of GDP)](https://data.worldbank.org/indicator/BX.TRF.PWKR.DT.GD.ZS?view=chart)
    - [Average transaction cost of sending remittances from a specific country (%)](https://data.worldbank.org/indicator/SI.RMT.COST.OB.ZS?view=chart)
    - [Average transaction cost of sending remittances to a specific country (%)](https://data.worldbank.org/indicator/SI.RMT.COST.IB.ZS?view=chart)
    
- **KNOMAD**
    - [Remittance inflows](https://www.knomad.org/sites/default/files/2023-06/remittance_inflows_brief_38_june_2023_3.xlsx)
    - [Outward remittances](https://www.knomad.org/sites/default/files/2023-06/remittance_outflows_brief_38_june_2023_3.xlsx)

However, these datasets were designed to be read by people, not computers. Before they could be fed into Python, some cleaning and formatting had to be done on each dataset. The datasets from World Bank had a lot of metadata and special notes for each country. More time was needed to go through those datasets. While the datasets from KNOMAD were already in a format that could be imported directly to Python with minimal cleaning.

## World Bank Datasets

Each dataset from World Bank followed a similar format with three sheets.

- `Data`: contained countries, country codes, the years, and the particular remittance data.

![Remittance Countries](https://i.imgur.com/9b0akxn.jpg)

- `Metadata - Countries`: contained country codes, regions, income group, and special notes.

![Remittance Metadata Countries](https://i.imgur.com/8cw3Yzy.jpg)

- `Metadata - Indicators`: explained what the indicator meant.

The first two sheets were important because they contained the data that needed to be consolidated. I copied the data from the Data sheet, removed the indicator name, the indicator code, and empty years. Then I used `VLOOKUP` with country codes to get the region and income group from the `Metadata - Countries` sheet.

> `=ARRAYFORMULA(VLOOKUP(B2:B267,'Metadata - Countries'!A:C,2,FALSE))`  # Region 
> `=ARRAYFORMULA(VLOOKUP(B2:B267,'Metadata - Countries'!A:C,3,FALSE))`  # Income

![Remittance VLOOKUP](https://i.imgur.com/KoHNGGl.jpg)

The good thing was that all of the World Bank datasets had a similar format. I repeated the same process for the remaining datasets.

### Lookup Table

The first four categorical columns in each dataset were: `Country Name`, `Country Code`, `Region`, and `Income`. They were the same across all datasets, which meant they could be placed into a separate sheet to be used as a lookup table. Rows that contained data in the wrong columns were fixed.

![Lookup Table](https://i.imgur.com/YZ5NwUD.jpg)

For the remaining columns in each dataset that contained numerical data, they were placed into individual sheets. The `Country Code` column was used as a lookup key to retrieve each country's remittance data.

![Numerical Sheet](https://i.imgur.com/L1ej8Ur.jpg)

Then I went back to the `country-lookup` and move the special groups to a separate sheet. They are specific groups based on various factors, such as economic, geography, income, etc. They do not have a region or income level so they should not be in the `country-lookup`.

Then I revisited the `country-lookup` sheet to move the special groups to a separate sheet. They were specific groups based on various factors, such as economic, geography, income, etc. They did not have a region or income level, so they should not be included in the lookup table.

![Special Groups](https://i.imgur.com/jMbDpqd.jpg)

### Remove Empty Rows

Each World Bank dataset had multiple countries that did not have any remittance data. Those rows needed to be removed since they did not add anything to the dataset. I filtered out completely empty rows and deleted them.

![Filter Empty](https://i.imgur.com/jvwd2AA.jpg)

## KNOMAD Datasets

The KNOMAD datasets were easier to work with than the World Bank datasets. They were already in a format that was easy to import and read in Python. They contained a column of country names but no country codes. This was simple enough to fix. The lookup-table from the World Bank dataset was copied into a sheet here. Then I used `INDEX` and `MATCH` to get the corresponding `Country Code` from the country name.

> `=INDEX('lookup-table'!$A$2:$A$267,MATCH(A2,'lookup-table'!$B$2:$B$267,0))`
> 

![Inflow Sheet](https://i.imgur.com/zq8R84l.jpg)

Of course, there were still some rows that did not match up perfectly due to spelling differences. But they were few enough that they could be fixed manually after using the formula. Multiple rows were also empty, but the same technique above with the World Bank datasets was used to remove those empty rows. Each cell was also multiplied by one million to expand the numbers to their full values.