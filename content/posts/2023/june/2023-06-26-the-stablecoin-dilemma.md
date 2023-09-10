---
title: "The Stablecoin Dilemma"
date: 2023-06-26T15:29:51+07:00
series: "Remittance Fees Analysis"
draft: false
tags: ["Thoughts"]
---

[Project Link](https://github.com/kietnguyen01/Remittance-Fees-Analysis)

Finding reliable transaction fees for stablecoins was a significant challenge. Unlike crypto and remittance services, stablecoins had less transparent and accessible data on their fees.

Stablecoins relied on different blockchains to issue and transfer their tokens, and each blockchain had a different fee structure. The most popular stablecoin, USDT, was issued on multiple blockchains, such as Ethereum, Tron, and BSC, while other stablecoins like USDC were mainly issued on Ethereum. The transaction fees for stablecoins depended on the blockchains they used and the market conditions. The fees also varied depending on the network congestion and the complexity of the transaction.

Due to the difficulty of finding data on stablecoin transaction fees, I considered dropping stablecoins from my analysis. Since my analysis focused on the remittance fees aspect, the lack of reliable data would have made the analysis not as robust. However, stablecoins played a significant role in the remittance market, especially in developing countries where access to traditional financial services was limited or costly. Removing them from the analysis would have taken away an important part of the crypto remittance market. In the end, I decided to include them.

## Protocol Share

Stablecoin protocol share was the percentage of the total supply of a stablecoin that was issued on a specific blockchain. For example, the protocol share of USDT on Ethereum was the percentage of the total USDT supply that was issued as ERC-20 tokens on Ethereum protocol. The same went for the USDT supply as TRC-20 on Tron protocol.

Finding data on the historical protocol share of each stablecoin was not easy. There were not many sources that provided this information in a consistent and reliable way. Some sources like [Blockchair](https://blockchair.com/) showed the current circulation of USDT on different blockchains, but they did not provide historical data. Tether’s official website sometimes published transparency reports that showed the breakdown of USDT by blockchain, but it did not do so regularly or consistently.

That’s why I decided to use Ethereum gas fees as a simpler method to estimate fees. Many stablecoins had the majority of their supplies on Ethereum, so it served as a good proxy for stablecoin transaction fees.

## Estimating Transaction Fees

One good source of historical gas fees was Owlracle, which provided historical gas prices for Ethereum in Gwei from 2015 to 2022. Its API was a free and easy-to-use service that allowed you to query gas prices for different time intervals. This method gave me a rough approximation of the transaction fees for stablecoins using Ethereum gas. However, it also had some limitations and assumptions that may have affected its accuracy and precision.

- The gas price and gas amount could have varied depending on the type and complexity of the stablecoin transaction.
- The gas price and gas amount could have also varied depending on the time and date of the stablecoin transaction.
- The exchange rate between ETH and USD could have fluctuated due to market forces and speculation.

Therefore, this estimate of the stablecoin transaction fees should only be used as a rough approximation.