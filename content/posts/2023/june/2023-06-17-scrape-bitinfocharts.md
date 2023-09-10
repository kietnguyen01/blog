---
title: "Scrape Bitinfocharts"
date: 2023-06-17T09:53:09+07:00
series: "Remittance Fees Analysis"
draft: false
tags: ["Collecting"]
---

[Project Link](https://github.com/kietnguyen01/Remittance-Fees-Analysis)

The next data I needed were the number of transactions, fees amount, and transaction values. BitInfoCharts had most of that data, but it did not have an easy way for me to download it. The data points were presented in an interactive graph format, which meant I needed to use Python and Selenium to scrape the data.

## Headless Selenium

I used headless Selenium to load the page and regular expressions to find the indices of the start and end dates in the page source. Then I extracted the substring between those indices, which contained the graph data. I added a `try-except` block to check if the last date of `2022/12/31` did not exist, then the code should use whichever last date the graph contained.

{{< highlight py >}}
def scrape_source(url: str, driver: webdriver, start_date: str, end_date: str) -> str:
    """Scrape token data from BitInfoCharts with headless Selenium"""
    # Wait for the chart to load and get page source
    driver.get(url)
    wait = WebDriverWait(driver, 1)
    wait.until(EC.presence_of_element_located((By.ID, 'container')))
    html = driver.page_source

    # Find the intended start and end dates using regex
    start = re.search(rf'\[new Date\("{start_date}"\)', html, re.DOTALL)
    end = re.search(rf'\[new Date\("{end_date}"\)[^\]]*', html, re.DOTALL)

    # Use a try-except block to handle the case when end is None
    try:
        # Extract the graph data in between
        graph_data = html[start.start():end.end() + 1]
    except AttributeError:
        # Use the last date found as the end index
        end_str = re.findall(r'\[new Date\("[\d/]*"\)[^\]]*\]', html, re.DOTALL)[-1]
        end = re.search(end_str, html, re.DOTALL)
        graph_data = html[start.start():end.end() + 1]

    return graph_data
{{< / highlight >}}

## Regular Expression

Since `scrape_source()` returned the raw HTML as a string, I needed to use regex to extract the data points. I didn’t like using regex but it was the most efficient way to get the job done. The regex found all matches of date and value pairs in the graph data and converted them to floats. Then I appended each value to the dictionary of each scraped statistic.

{{< highlight py >}}
def extract_vals(stats: list, driver: webdriver, start: str, end: str) -> dict:
    """Extract stat values from scraped page source into polars dataframe"""
    url_start = 'https://bitinfocharts.com/comparison/'
    url_end = '.html#alltime'

    # Dict with empty list for each stat key
    token_dict = {stat: [] for stat in stats}

    for stat in stats:
        full_url = unquote(url_start + stat + url_end)  # Decode url with special chars
        data = scrape_source(full_url, driver, start, end)

        # Extract stat values from scraped data
        pattern = r'new Date\("(.+?)"\),([\d.]+)'
        matches = re.findall(pattern, data)
        for match in matches:
            token_dict[stat].append(float(match[1]))

    return token_dict
{{< / highlight >}}

## Creating Dataframes

I created a separate Polars dataframe for each stat in the returned dictionary. Then I concatenated them together horizontally. This ensured that even if a stat list was shorter, it was going to be extended to be the same length as the other stats. Then I generated a `date` column and reordered it to be in the front of the dataframe.

{{< highlight py >}}
def create_dataframe(token_dict: dict, start: str, end: str) -> pl.DataFrame:
    """Create polars dataframe with date column and token data"""
    # Create separate dataframe for each state, concat them and extend the shorter columns
    start = datetime.strptime(start, '%Y/%m/%d').date()
    end = datetime.strptime(end, '%Y/%m/%d').date()
    df = pl.concat(
        items=[pl.DataFrame({_name: _values})
            for _name, _values in token_dict.items()],
        how="horizontal",
    )
    # Create polars dataframe with date column
    delta = end - start
    dates = [start + timedelta(days=i) for i in range(delta.days + 1)]
    df = df.with_columns(pl.Series("date", dates))
    return df
{{< / highlight >}}

Then I called `create_dataframe` for each token ID to create a dictionary of scraped dataframes.

{{< highlight py >}}
def create_scrape_dict(token_stats: dict, driver: webdriver, start: str, end: str) -> dict:
    """Create a dictionary of Polars dataframes for each crypto id"""
    # Build a dataframe for each token and its stats list
    token_dfs = {}
    for token, stats in token_stats.items():
        token_dict = extract_vals(stats, driver, start, end)
        token_df = create_dataframe(token_dict, start, end)
        token_dfs[token] = token_df
    return token_dfs
{{< / highlight >}}

## Running the Code

The last part was just running the code.

{{< highlight py >}}
# Dict of token stats list for scraping
token_stats = {
    'btc': ['bitcoin-transactions', 'bitcoin-transactionvalue', 'bitcoin-mediantransactionvalue', 'bitcoin-transactionfees', 'bitcoin-median_transaction_fee'],
    'xrp': ['xrp-transactions', 'transactionfees-xrp'],
    'doge': ['dogecoin-transactions', 'dogecoin-transactionvalue', 'dogecoin-mediantransactionvalue', 'dogecoin-transactionfees', 'dogecoin-median_transaction_fee'],
    'ltc': ['litecoin-transactions', 'litecoin-transactionvalue', 'litecoin-mediantransactionvalue', 'litecoin-transactionfees', 'litecoin-median_transaction_fee'],
    'xmr': ['monero-transactions', 'monero-transactionfees'],
    'bsv': ['transactions-bsv'],
    'bch': ['bitcoin%20cash-transactions', 'bitcoin%20cash-transactionvalue', 'bitcoin%20cash-mediantransactionvalue', 'bitcoin%20cash-transactionfees', 'bitcoin%20cash-median_transaction_fee'],
    'zec': ['zcash-transactions', 'zcash-transactionfees', 'transactionvalue-zec', 'mediantransactionvalue-zec'],
    'dash': ['dash-transactions', 'dash-transactionfees', 'dash-median_transaction_fee', 'transactionvalue-dash', 'mediantransactionvalue-dash']
}

start_date = '2019/01/01'
end_date = '2022/12/31'

# Create dict of scraped dataframes
scrape_dict = cs.create_scrape_dict(token_stats, driver, start_date, end_date)
{{< / highlight >}}