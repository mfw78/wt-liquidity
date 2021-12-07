---
title: Winding Tree Liquidity Analysis
lang: en-US
author: mfw78 <mfw78@chasingseed.com>
---

# Winding Tree (LIF) Market Analysis

## Overview

Winding Tree (LIF) is an ERC20 token issued by [Winding Tree](https://www.windingtree.com),
currently trading only via decentralised exchanges (DEX). Historically, LIF has been plagued by
data indexing problems and low liquidity increasing - leading to friction for onramping new LIF 
users.

Indexing problems have been due to a combination of indexing services assuming the presence
of *OPTIONAL* [ERC20](https://eips.ethereum.org/EIPS/eip-20#methods) methods in LIF's ERC20
token, as well as low liquidity not meeting the minimum requirements for price tracking by
various services.

In order to implement the *OPTIONAL* ERC20 methods, the LIF token from the ICO had to be
[deprecated](https://etherscan.io/token/0xeb9951021698b42e4399f9cbb6267aa35f82d59d) (hereby
referred to as *LIFv1*), and subsequently replaced by a 
[fully ERC20 compliant token](https://etherscan.io/token/0x9c38688e5acb9ed6049c8502650db5ac8ef96465) 
(hereby referred to as *LIFv2*) that implements the aforementioned *OPTIONAL* methods.

In summary, the following tokens are considered for market analysis:

1. LIFv1: [0xeb9951021698b42e4399f9cbb6267aa35f82d59d](https://etherscan.io/token/0xeb9951021698b42e4399f9cbb6267aa35f82d59d)
2. LIFv2: [0x9c38688e5acb9ed6049c8502650db5ac8ef96465](https://etherscan.io/token/0x9c38688e5acb9ed6049c8502650db5ac8ef96465)

## Objectives

It is the objective of the Winding Tree to foster and develop a market that:

1. Facilitates smooth trading of LIF to reduce friction for onboarding.
2. Meets the requirements for price monitoring as determined by the respective DEX.
3. Minimise team liquidity risk, discouraging pumping / dumping of LIF.

## Analysis

For the purposes of market analysis, LIFv1 and LIFv2 shall both be taken into account as
the `claim()` function is still possible, therefore LIFv1 is for all intents and purposes the same
as LIFv2 as they are exchangeable (one-way) on a 1:1 basis. Failure to analyse both markets as one
may result in policy that would otherwise lead to arbitrage opportunities, and possible negative
outcomes for liquidity providers.

### Market Overview

    LIFv1
    -----
    Number of holders: 6363
    Average holder balance: 2633.7 LIF
    
    LIFv2
    -----
    Number of LIFv2 holders: 489
    Average holder balance: 12,887.2 LIF
    
    Conversion statistics
    ---------------------
    Amount of LIFv2 claimed: 25.23%
    
**NOTE: Number of holders above excludes the LIFv2 smart contract address which is the largest 
holder of both LIFv1 and LIFv2 (unclaimed LIFv2).**

### DEX Liquidity

Liquidity currently available, is spread as follows:

| Token | Exchange   | Number of LPs | WETH Locked | LIF Locked   |
| ----- | ---------- | ------------- | ----------- | ------------ |
| LIFv1 | Uniswap V2 | 4             | 15.3243     | 164,055      |
| LIFv1 | Uniswap V3 | 0             | 39 WEI*     | 8457 x 10-18 |
| LIFv1 | SushiSwap  | 1             | 0.001001    | 0.999        |
| LIFv1 | IDEX       | Unknown       | N/A         | 539,606      |
| LIFv2 | Uniswap V3 | 2             | 0.0466      | 208,578      |

From the above table it can be seen that the only market with anything resembling liquidity is
that of LIFv1 on Uniswap V2. It is the intent of this report to address the LIF market liquidity
issues.

### Price Indexing

Indexing of pricing / volume data from a DEX is generally done via 
[TheGraph](https://thegraph.com) protocol. The respective DEX publishes a subgraph, which is
essentially a map of how indexers are to process and aggregate exchanges events from the DEX. It's
from these subgraphs that pricing, volume and other trading data is determined.

Fortunately, with these being open source, we can see how their subgraphs work:

* Uniswap V2: https://github.com/Uniswap/v2-subgraph
* Uniswap V3: https://github.com/Uniswap/v3-subgraph
* SushiSwap: https://github.com/sushiswap/sushiswap-subgraph

The price and volume tracking parameters are summarised in the following table:

| DEX         | Minimum ETH Locked | Minimum USD Threshold      |
| ----------- | ------------------ | -------------------------- |
| Uniswap V2  | 2 [^1]             | 400,000 [^3] (<5 LPs) [^2] |
| Uniswap V3  | 52 [^4]            | N/A                        |
| SushiSwap   | 3 [^5]             | 3,000 [^6]                 |

It is the intent of this report to put forward a liquidity strategy that addresses the above
parameters to ensure wide price and volume coverage.

[^1]: [Uniswap V2 Subgraph source code](https://github.com/Uniswap/v2-subgraph/blob/7c82235cad7aee4cfce8ea82f0030af3d224833e/src/mappings/pricing.ts#L69)
[^2]: [Uniswap V2 Subgraph source code](https://github.com/Uniswap/v2-subgraph/blob/7c82235cad7aee4cfce8ea82f0030af3d224833e/src/mappings/pricing.ts#L120)
[^3]: [Uniswap V2 Subgraph source code](https://github.com/Uniswap/v2-subgraph/blob/7c82235cad7aee4cfce8ea82f0030af3d224833e/src/mappings/pricing.ts#L66)
[^4]: [Uniswap V3 Subgraph source code](https://github.com/Uniswap/v3-subgraph/blob/c839c0b3afb09aece6e33991beecbafd4eddb0b0/src/utils/pricing.ts#L44)
[^5]: [Sushiswap Subgraph source code](https://github.com/sushiswap/sushiswap-subgraph/blob/dcc76971af9b477203b1712e40aa869d4a9be6f1/config/mainnet.json#L29)
[^6]: [Sushiswap Subgraph source code](https://github.com/sushiswap/sushiswap-subgraph/blob/dcc76971af9b477203b1712e40aa869d4a9be6f1/config/mainnet.json#L28)

### LIFv2 Claims

#### Process overview

The LIFv2 claims process is done via a [dApp](https://lif.windingtree.com/). This dApp essentially
facilitates the end user taking the following actions:

1. `approve()` the LIFv2 contract to spend their LIFv1 tokens.
2. `claim()` on the LIFv2 contract which will then transfer their LIFv1 tokens to the LIFv2 
   contract, and send them the corresponding amount of LIFv2.

As such, the above transactions are required to be executed by the token holder, and the associated
Ethereum network transaction fees are paid by the token holder. This gives rise to two main
barriers for the claim process:

1. Network transaction costs (gas fees)
2. Dead wallets

#### Gas fees

Let's analyse this using a couple of example transactions:

* [approve() transaction](https://etherscan.io/tx/0xe9746c6235dca6a14ff702e24789f43519dc16d22f4f0060949e3056cee9c5b5)
* [claim() transaction](https://etherscan.io/tx/0x208237af790fdb8078beae437d3dd628db9dc1108dbfb0982ea305629e7896b0)

The following table summarises the gas consumption by method:

| Method      | Gas consumed |
| ----------- | ------------ |
| `approve()` | 48,359 [^7]  |
| `claim()`   | 74,627 [^8]  |
| Subtotal:   | 122,986      |

[^7]: [Example `approve()` transaction](https://etherscan.io/tx/0xe9746c6235dca6a14ff702e24789f43519dc16d22f4f0060949e3056cee9c5b5)
[^8]: [Example `claim()` transaction](https://etherscan.io/tx/0x208237af790fdb8078beae437d3dd628db9dc1108dbfb0982ea305629e7896b0)

Therefore, on average, let's assume that the amount of gas consumed by the claim process is **122986 Gas**. 
Note that gas amounts will vary based upon claim amounts used in the process. We can take this and further 
break it down looking at a gas price spread to analyse multiple gas price scenarios:

| Gas price | TX Fees (ETH) |
| --------- | ------------- |
| 50 gwei   | 0.0061493 ETH |
| 75 gwei   | 0.0092239 ETH |
| 100 gwei  | 0.0122986 ETH |
| 125 gwei  | 0.0153732 ETH |
| 150 gwei  | 0.0184479 ETH |
| 175 gwei  | 0.0215225 ETH |
| 200 gwei  | 0.0245972 ETH |

The current price at the time of writing for LIFv1 (the most liquidity in terms of outright USD) was 
0.00009257 ETH / LIF. Based on this, we can determine the *minimum* amount of LIF for a zero-loss, viable, 
economic claim (ie. the amount of LIF required to cover transaction costs):

| Gas Price | Minimum LIF |
| --------- | ----------- |
| 50 gwei   | 66.429      |
| 75 gwei   | 99.642      |
| 100 gwei  | 132.857     |
| 125 gwei  | 166.072     |
| 150 gwei  | 199.286     |
| 175 gwei  | 232.500     |
| 200 gwei  | 265.715     |

Looking closer at the current claims that have been completed, we can roughly determine the market 
sentiment / fee appetite with respect to the claim process. Currently, there are 489 LIFv2 holders. 
Let's *assume* that these are all claimants (ie. assuming no *new* holders from swapping on
secondary markets).

For the purpose of this analysis, it seems most appropriate to use the *median* token balance.
This represents the appetite for at least 50% of the holders as the *minimum* amount that
they would swap, and otherwise accept the fees for the swap (note that the *mean* token balance
is significantly higher).

The LIFv2 token holder's account balance summary statistics are as follows:

    Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
     0.0    855.4   2103.8  12887.2   9125.3 413704.9

Since the claim process began, let's *assume* an average gas price of 100 gwei. In this
scenario, 50% of claimants have been willing to accept a maximum 6.315% network transaction cost to
LIF claimed ratio (132.857 minimum LIF / 2103.8 median LIF held).

Assuming an average gas price of 100 gwei and a market appetite for a 6.315% transaction
fee, the following scenarios demonstrate the amount of LIF supply (and number of affected
wallets) that would be deemed uneconomical for claiming, and therefore effectively out 
of circulation:

| LIF Price (LIF / ETH) | Number of wallets | Amount of LIF OOC |
| --------------------- | ----------------- | ----------------- |
| 0.00009257 (Current)  | 5271 (76.93%)     | 2,918,839.61      |
| 2x current            | 4429 (64.64%)     | 1,650,990.12      |
| 4x current            | 3304 (48.22%)     |   684,084.47      |
| 8x current            | 2258 (32.95%)     |   262,347.34      |

The above table demonstrates that a failure to either introduce a more economical claim method, or
to achieve an increase in price may lead to a future DAO being more likely to be subject to 51%
governance attacks given the corresponding increase in token supply centralisation.

Any liquidity strategy should ensure smooth trading to an upper bound giving the widest
distribution of LIF as possible.

### Dead Wallets

*Analysis pending*

### Decentralisation of LIFv2

The *Nakamoto Co-efficient* can be used to determine how decentralised a system is. The *Nakamoto
Co-efficient* represents the number of actors that would have to conspire in order to execute a 
51% attack on a system's consensus mechanism. 

In the context of LIF, this would be most applicable in a future DAO situation whereby voting power
may be concentrated, and represent a DAO governance attack vector.

Based upon the current portion of the market that have migrated to LIFv2, if the `claim()` method
were to be disabled, LIFv2 would have the follow characteristics:

    Actual supply: 6,301,843.128
    Nakamoto co-efficient: 20

Efforts must be increased to encourage the migration of LIFv2, with potentially a view in the
future to conduct some form of token distribution event that increased the breadth of the LIFv2
supply in order to harden the token against centralisation attacks.

## Liquidity Strategy

Recently on a [Snapshot proposal](https://snapshot.org/#/windingtree.eth/proposal/QmNMtnoR2qrNj4xy4pGe6yuQcAwLRmxyhszFAWVZnPa1fZ), 
the community's temperature was taken on whether to migrate to a sidechain, and as to which DEX is
favoured by the community. Overwhelmingly, the community is in favour of 
[Uniswap](https://app.uniswap.org). 

For developing the liquidity strategy, let's consider the statistics of all DEX trades in the 
current calendar year.:

    LIFv1:
    count   1655
    mean       0.0001730595
    std        0.0013236749
    min        0.0000393600
    25%        0.0000842550
    50%        0.0001056900
    75%        0.0001684600
    max        0.0538768100

    LIFv2:
    count   71
    mean     0.0001111290
    std      0.0000332345
    min      0.0000999300
    25%      0.0001008950
    50%      0.0001028300
    75%      0.0001088400
    max      0.0003305700

For the purpose of liquidity position creation, let's consider creating positions to back a
current price of 0.00009257 ETH / LIF so as to minimise opportunity for arbitrage bots.

In total, it is suggested to make 2 markets: Uniswap V2 and Uniswap V3.

### Uniswap V2

* Funding requirements: ~2.75 ETH, ~27,000 LIF.
* Overview: Meet 2 ETH minimum liquidity requirement, and 5 LPs.

Steps:

1. Generate a random mnemonic.
2. Use generated private keys 1 - 5 as separate LP accounts.
3. Transfer 2.75 ETH and 27000 LIF to account #1
4. Create LIF / WETH Uniswap V2 pool with account #1 - adding 2.5 ETH and 27,000 LIF to pool.
5. Send 20% of LP tokens to accounts #2 - #5.

Transactions required:

* ETH transfers: 1
* ERC20 approvals: 1
* Uniswap V2 pool creation: 1
* ERC20 transfers: 5

By doing the above, 5 liquidity provider positions can be created, saving gas on LP add
transactions, and approval transactions.

Goals met:

* Price tracking with > 2 ETH
* Volume tracking with >= 5 LPs

### Uniswap V3

* Funding requirements: ~60.3 ETH, ~254,013 LIF
* Overview: Meet 52 ETH minimum liquidity requirement, and future proof 60 ETH requirement.

Create multiple positions:

| Lower tick  | Upper tick  | WETH | LIF    |
| ----------- | ----------- | ---- | ------ |
| 0           | Infinite    | 5    | 54013  |
| 0.000035024 | 0.000040206 | 50   | 0      |
| 0.000040206 | 0.000092571 | 5    | 0      |
| 0.000092571 | 0.0037063   | 0    | 200000 |

*NB: Ticks above are in ETH / LIF*

Steps:

1. A swap transaction to "fix" the current pool as it has been "slammed" against a stop.
2. Remove present liquidity position by WT.
3. Add above positions in order (4 separate transactions).

Transactions required:

* ERC20 approvals: 1 (UniswapV3 Router / NFT position manager)
* Uniswap V3 swaps: 1
* Remove liquidity: 1
* Add liquidity: 4

Goals met: 

* Price tracking with > 52 ETH
* Reduces liquidity risk by anchoring most liquidity below minimum price paid by 
  market participants (and therefore protects price tracking).
* Discourages initial dumping by inducing a reasonable slippage penalty for those dumping.
* Provides healthy liquidity up to a 4x range for smooth trading so that most wallets
  may become economical to go through claim process.

# Conflicts of interest

The author declares they:

1. Have an interest of 10142 LIFv2.
2. Are actively engaged in cross liquidity pool arbitrage trading.