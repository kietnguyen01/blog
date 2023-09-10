---
title: "Performing the Analysis"
date: 2023-08-21T18:09:26+07:00
series: "Remittance Fees Analysis"
draft: false
tags: ["Analysis"]
---

[Project Link](https://github.com/kietnguyen01/Remittance-Fees-Analysis)

In this blog post, I went over the process I used to perform the analysis and make the comparison between different methods of remittance.

## Crypto

let's focus on the crypto aspect of the analysis. The initial step was to calculate the fee percentage for each cryptocurrency by considering average transaction fees and values. I proceeded by creating a new column in each dataframe to store this calculated value. Next, I presented the fee percentage for all cryptocurrencies on a weekly basis, utilizing a line graph. Additionally, to facilitate better comparison between the varying magnitudes of fee percentages, I opted for a logarithmic scale on the y-axis.

```python
import pandas as pd
import seaborn as sns
import statsmodels.api as sm
import matplotlib.pyplot as plt

sns.set(style="whitegrid") 

def crypto_fee_percentage(dfs: dict) -> dict:
    """Calculate and create a new column for fee percentage based on average transaction fees and values"""
    for _, df in dfs.items():
        df['fee_percentage'] = (df['average_transaction_fees'] / df['average_transaction_value']) * 100
    return dfs

def crypto_fee_percentage_lineplot(dfs: dict) -> None:
    """Plot the fee percentage column for all dataframes in the dictionary on a weekly basis"""
    plt.figure(figsize=(10, 6))
    for name, df in dfs.items():
        # Resample to weekly frequency 
        df_weekly = df.resample('W').median()  
        # Calculate rolling mean with 4 week window
        df_roll = df_weekly.rolling(4, min_periods=1).mean()
        # Plot smoothed line
        sns.lineplot(data=df_roll, x=df_roll.index, y='fee_percentage', label=name)
    plt.xlabel('date')
    plt.ylabel('Fee Percentage')
    plt.title('Fee Percentage Plot (Weekly)')
    plt.yscale('log')
    plt.legend(loc='upper left', bbox_to_anchor=(1, 1))
    plt.show()
```

Here is the code that calls the function:

```python
crypto_path = os.path.join('data/crypto/', 'imputed-data.xlsx')
# Import crypto dataframes except XMR
crypto = gf.import_excel(
    crypto_path,
    ['btc', 'xrp', 'doge', 'ltc', 'bch', 'xlm', 'bsv', 'zec', 'dash']
)
# Convert to pandas
for df in crypto:
    crypto[df] = crypto[df].to_pandas()
    crypto[df].index = crypto[df]['date']
    crypto[df] = crypto[df].drop(columns=['date'])

# Add fee percentage column to crypto dict
crypto = ana.crypto_fee_percentage(crypto)

# Create lineplots for fee percentages
ana.crypto_fee_percentage_lineplot(crypto)
```

![Crypto Fee Percentage](https://i.imgur.com/o0tnqyG.png)

There was a significant variation in the fee percentage among different cryptocurrencies and over time. Certain cryptocurrencies, like BTC, BCH, and BSV, exhibited notably high fee percentages, particularly during periods of intense network congestion and heightened demand. On the other hand, cryptocurrencies like XRP, XLM, and DOGE boasted considerably low fee percentages, making them more appealing for remittance transfers.

### Weighted Average

Moving on, the subsequent step involved creating distinct scenarios for each year regarding the crypto fee percentage. To accomplish this, I utilized the mean value of the fee percentage and the 24-hour volume for each cryptocurrency and each year. The 24-hour volume served as a proxy for measuring market activity and liquidity of each cryptocurrency. Utilizing this information, I established a weighted average case for each year, which represented the average fee percentage across all cryptocurrencies, weighted by their respective market volumes.

```python
def crypto_fees_scenarios(dfs: dict) -> pd.DataFrame:
    """Create a dataframe that has different scenarios for each year for crypto fee percentage"""
    scenario_data = []
    for name, df in dfs.items():
        for year in range(2019, 2023):
            df_year = df[df.index.year == year]
            # Get the mean value of fee_percentage directly
            fee_percent = df_year['fee_percentage'].describe().loc['mean']
            # Get the mean value of 24h_volume directly
            volume_weight = df_year['24h_volume'].mean()
            scenario_data.append({
                'crypto': name,
                'year': year,
                'mean_fee_percent': fee_percent,
                'volume_weight': volume_weight
            })
    # Create dataframe and reorder columns
    scenario_df = pd.DataFrame(scenario_data)
    scenario_df = scenario_df.reindex(columns=['crypto', 'year', 'mean_fee_percent', 'volume_weight'])
    return scenario_df

def combine_average_case(scenario_df: pd.DataFrame) -> pd.DataFrame:
    """Create a dataframe that has the weighted_average_case for each year using the volume_weight"""
    combined_data = []
    for year in range(2019, 2023):
        scenario_df_year = scenario_df[scenario_df['year'] == year]
        # Calculate the weighted_average_case using the formula
        weighted_mean_fee = (scenario_df_year['mean_fee_percent'] * scenario_df_year['volume_weight']).sum() / scenario_df_year['volume_weight'].sum()
        combined_data.append({
            'year': year, 
            'Crypto': weighted_mean_fee * 100
        })
    # Create dataframe
    combined_df = pd.DataFrame(combined_data)
    combined_df.index = combined_df['year']
    combined_df = combined_df.drop(columns=['year'])
    return combined_df
```

## Stablecoin

Now, let's shift our focus to the stablecoin segment of the analysis. To begin with, the initial step involved plotting the Ethereum transaction fees and gas prices on a weekly basis using a line graph. Transaction fees represent the total amount of Ether paid by the sender of a transaction to the miner of the block that includes the said transaction. On the other hand, gas prices indicate the amount of Ether per unit of gas that the sender is willing to pay for executing the transaction. It's important to note that gas is a measure of the computational work required to process a transaction on the Ethereum network.

Considering the significance of transaction fees and gas prices, they play a crucial role in determining the cost associated with using stablecoins on Ethereum. As ERC-20 tokens running on smart contracts within the Ethereum network, stablecoins necessitate the payment of gas fees for sending or receiving them. Higher gas fees directly translate to increased expenses when utilizing stablecoins for remittance transfers.

```python
def plot_gas_fees(df: pd.DataFrame) -> None:
    """Plot the Ethereum transaction fees and gas prices on a weekly basis"""
    plt.figure(figsize=(12, 6))
    # Resample to weekly frequency
    df_weekly = df.resample('W').mean()
    # Use the index as the x-axis
    sns.lineplot(data=df_weekly, x=df_weekly.index, y='transaction_fees')
    plt.xlabel('Date')
    plt.ylabel('Gas Fees (Dollar)')
    plt.title('Ethereum Gas Fees Plot (Weekly)')
    plt.show()
```

Calling the function:

```python
stablecoin_path = os.path.join('data/stablecoin/', 'imputed-data.xlsx')
# Import stablecoin dataframes
stablecoin = gf.import_excel(
    stablecoin_path,
    ['usdt', 'usdc', 'busd', 'dai', 'tusd', 'usdp', 'gusd', 'fees']
)
# Convert to pandas
for df in stablecoin:
    stablecoin[df] = stablecoin[df].to_pandas()
    stablecoin[df].index = stablecoin[df]['date']
    stablecoin[df] = stablecoin[df].drop(columns=['date'])

# Plot weekly gas fees
ana.plot_gas_fees(stablecoin['fees'])
```

![Ethereum Gas Fees](https://i.imgur.com/jY98rya.png)

The gas fees exhibited fluctuations over time, characterized by occasional spikes and dips. Notably, the gas fees reached their peak around July 2020, coinciding with a period of high congestion and demand within the Ethereum network due to the widespread popularity of decentralized finance (DeFi) applications. However, the gas fees experienced a decline around January 2021, following the implementation of various improvements and optimizations aimed at reducing congestion and enhancing scalability within the Ethereum network.

Moving forward, the subsequent step involved creating diverse scenarios for each year concerning Ethereum gas fees. To achieve this, I utilized the mean value of the transaction fees for each respective year. Subsequently, I proceeded to calculate the fee percentage associated with sending $200 worth of stablecoins using Ethereum. It's important to note that this calculation relied on a simple approximation, assuming a constant exchange rate between Ether and US dollars, as well as a consistent gas limit for each transaction.

```python
def stablecoin_fees_scenarios(df: pd.DataFrame) -> pd.DataFrame:
    """Create a dataframe that has different scenarios for Ethereum gas fees each year"""
    scenario_data = []
    for year in range(2019, 2023):
        df_year = df[df.index.year == year]
        # Get the mean value of transaction_fees directly
        mean_fee = df_year['transaction_fees'].describe().loc['mean']
        # Calculate fee percentage for $200
        scenario_data.append({
            'year': year,
            'Stablecoin': (mean_fee / 200) * 100
        })
    scenario_df = pd.DataFrame(scenario_data)
    scenario_df.index = scenario_df['year']
    scenario_df = scenario_df.drop(columns=['year'])
    return scenario_df
```

The fee percentage was low in 2019, when the gas fees were relatively stable and cheap. The fee percentage increased in 2020, at the start of the DeFi boom. The fee percentage exploded in 2021 as more projects came online and people were speculating heavily. The fees dropped in 2022 when the market crashed. The fee percentage is projected to decrease further past 2022, when the network is expected to transition to a more scalable and efficient consensus mechanism called proof-of-stake.

## Remittance

Let's delve into the remittance data for this section. To begin, we'll start by predicting the remittance costs for the years 2021 and 2022 using linear regression. These remittance costs represent the average total expenses incurred when sending $200 from one country to another. It's important to note that these costs encompass the fees charged by the service provider as well as the exchange rate margin. To provide a better understanding, these costs are expressed as a percentage of the transaction value.

Undoubtedly, remittance costs play a crucial role in determining the affordability and accessibility of remittance transfers. Lower costs translate to more funds being available for remittance users to both send and receive. In fact, the World Bank has set a global target of reducing the average remittance costs to 3% by 2030, aiming to enhance the overall efficiency of remittance services.

To predict the remittance costs for each income group and region, I employed a simple linear regression model. This model was built upon historical data spanning from 2011 to 2020. In this analysis, the year served as the independent variable, while the remittance cost acted as the dependent variable. It's worth mentioning that for simplicity, I assumed a constant relationship between the two variables, disregarding any potential outliers or nonlinearities.

```python
def predict_remittance_cost(df: pd.DataFrame) -> pd.DataFrame:
    """Predict remittance costs for the year 2021 and 2022 using linear regression"""
    # Empty predictions dataframe
    pred_df = pd.DataFrame(columns=df.columns)
    for col in df.columns:
        # Year and remittance cost variables
        X = df.index.values
        y = df[col].values
        # Year constant term
        X = sm.add_constant(X)
        # Fit model with Ordinary Least Squares
        model = sm.OLS(y, X).fit()
        # Predict with fitted model
        pred_X = sm.add_constant([2021, 2022])
        pred_y = model.predict(pred_X)
        # Add the predictions to the dataframe
        pred_df[col] = pred_y
    # Set the index of dataframe
    pred_df.index = [2021, 2022]
    pred_df.index.name = df.index.name
    return pred_df
```

And to call it:

```python
remittance_path = os.path.join('data/remittance/', 'remittance-tables.xlsx')
# Import remittance dataframes
remittance = gf.import_excel(
    remittance_path,
    ['cost_to_income', 'cost_to_region', 'cost_from_income', 'cost_from_region',
     'inflow_income', 'inflow_region', 'outflow_income', 'outflow_region']
)
# Convert to pandas
for name in remittance:
    # Convert to pandas
    remittance[name] = remittance[name].to_pandas()
    # Impute missing columns in cost_from dataframes
    if name == 'cost_from_income' or name == 'cost_from_region':
        remittance[name] = cc.mf_impute(remittance[name])
    remittance[name].index = remittance[name]['year']
    remittance[name] = remittance[name].drop(columns=['year'])

# Generate predictions for 2021/2022 for income to/from tables
pred_dict = {}
for key in remittance.keys():
    # Check if the dataframe has 2021 and 2022 or not
    if 2021 not in remittance[key].index and 2022 not in remittance[key].index:
        # Call the function on each dataframe
        pred_df = ana.predict_remittance_cost(remittance[key])
        # Store the predictions in the new dict
        pred_dict[key] = pred_df
        # Append the predictions to the original dataframes
        remittance[key] = pd.concat([remittance[key], pred_df], axis=0)
```

## Comparison Analysis

I will use a line plot to visualize the fee percentage for each option and each year. I will also discuss the advantages and disadvantages of each option and provide some recommendations for remittance users.

```python
# Remittance dataframes
cti = remittance['cost_to_income']
ctr = remittance['cost_to_region']

# Create concatenated dataframe for 2019-2022 along columns with keys
all_fees = pd.concat([
    crypto_fees,
    stablecoin_fees,
    cti[(cti.index >= 2019) & (cti.index <= 2022)],
    ctr[(ctr.index >= 2019) & (ctr.index <= 2022)]
], axis=1)

# Reshape the dataframe from wide to long
all_fees = all_fees.reset_index()
long_fees = pd.melt(all_fees, id_vars='year', var_name='group', value_name='fee_percentage')

# Select the columns for Crypto, Stablecoin, and Income groups
df = all_fees.loc[:, ['year', 'Crypto', 'Stablecoin', 'High income', 'Low income', 'Lower middle income', 'Upper middle income']]

# Reshape the dataframe from wide to long
long_fees = pd.melt(df, id_vars='year', var_name='group', value_name='fee_percentage')

# Plot each group over the four years
fig = plt.figure(figsize=(10, 6))
sns.lineplot(data=long_fees, x='year', y='fee_percentage', hue='group', errorbar=None)
plt.xticks([2019, 2020, 2021, 2022])
plt.legend(loc='upper left', bbox_to_anchor=(1, 1))
plt.show()

# Select the columns for Crypto, Stablecoin, and Region groups
df = all_fees.loc[:, ['year', 'Crypto', 'Stablecoin', 'East Asia & Pacific', 'Europe & Central Asia', 'Latin America & Caribbean', 'Middle East & North Africa', 'South Asia', 'Sub-Saharan Africa']]

# Reshape the dataframe from wide to long
long_fees = pd.melt(df, id_vars='year', var_name='group', value_name='fee_percentage')

# Plot each group over the four years
fig = plt.figure(figsize=(10, 6))
sns.lineplot(data=long_fees, x='year', y='fee_percentage', hue='group', errorbar=None)
plt.xticks([2019, 2020, 2021, 2022])
plt.legend(loc='upper left', bbox_to_anchor=(1, 1))
plt.show()
```

![Income Comparison](https://i.imgur.com/lS4KtyE.png)

![Region comparison](https://i.imgur.com/patHQYT.png)

Analyzing the plot, it becomes evident that the fee percentage varies significantly across different years and groups for each remittance option. Let's delve into some key observations:

- Crypto exhibited the lowest fee percentage in 2019 and 2022, but the highest fee percentage in 2020 and 2021. This stark contrast reflects the inherent volatility and unpredictability of crypto prices and fees over time. While cryptocurrencies offer fast and borderless transactions, they also expose users to substantial risks and uncertainties.
- Stablecoin displayed a moderate fee percentage in 2019 and 2020 but experienced a significant increase in 2021 and 2022. This notable surge can be attributed to the impact of Ethereum gas fees on stablecoin transfers. Stablecoins boast low volatility and high liquidity, but their cost-effectiveness heavily relies on the network congestion and scalability of the underlying Ethereum platform.
- Among the income groups, the high-income segment consistently exhibited the lowest fee percentage across all years. This trend highlights the availability and affordability of remittance services in high-income countries. High-income users benefit from a wider range of options and increased competition in the remittance market, which ultimately lowers the associated costs.
- Conversely, the low-income segment consistently displayed the highest fee percentage across all years. This disparity reflects the limited access and inclusion of remittance services in low-income countries. Users within this income bracket face fewer options and encounter higher barriers when engaging in remittance transfers, leading to increased costs.
- Notably, the fee percentages for the lower middle income and upper middle-income groups remained relatively similar across all years. This observation underscores the diversity and heterogeneity of remittance services within these income groups. Users falling within these categories encounter a mix of options and challenges when conducting remittance transfers, which inevitably affects the overall costs involved.

## Recommendations

Based on the observations we've made, here are some recommendations for remittance users to consider:

- For users who are willing to take risks and can benefit from low fees during favorable market conditions, crypto can be a viable option. However, it's crucial for users to be fully aware of the potential losses and difficulties that may arise when market conditions become unfavorable. Careful consideration and understanding of the crypto market dynamics are essential.
- Stablecoin can be a suitable option for users seeking to avoid volatility and enjoy high liquidity. However, it's important to take into account the impact of Ethereum gas fees on stablecoin transfers. Exploring alternative platforms or solutions that can help reduce or mitigate these fees may be beneficial for stablecoin users.
- Remittance services can be a reliable and convenient option for users prioritizing reliability and convenience over speed and cost. However, it's advisable for users to compare different providers and corridors to find the best deals available, while also being cautious of hidden charges or unfavorable exchange rates. Thorough research and due diligence can help users make informed decisions.

In conclusion, it's important to note that there is no one-size-fits-all solution for remittance transfers. Each option comes with its own advantages and disadvantages. Users should thoroughly evaluate their needs, preferences, and the specific characteristics of each option before making their decisions. Additionally, staying updated on changes and trends in the remittance market is crucial, as new technologies and innovations can present new opportunities and challenges for remittance transfers.