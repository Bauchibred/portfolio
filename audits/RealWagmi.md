# Introduction

A time-boxed security review of the **RealWagmi** protocol which was done by **Bauchibred** with a focus on the security aspects of the application's smart contracts implementation.

# Table of Contents

- [Disclaimer](#disclaimer)
- [Severity classification](#Severity-classification)
- [Scope](#scope)
- [About Bauchibred](#About-Bauchibred)
- [About RealWagmi](#About-RealWagmi)
- [Summary](#summary)
- [Findings](#findings)

# Disclaimer

A smart contract security review can never verify the complete absence of vulnerabilities. This is a time, resource and expertise bound effort where I try to find as many vulnerabilities as possible. I can not guarantee 100% security after the review or even if the review will find any problems with your smart contracts. Subsequent security reviews, bug bounty programs and on-chain monitoring are strongly recommended.

Additionally, this report was prepared for [this competition](https://audits.sherlock.xyz/contests/88). The competition spanned **5** days, from **_June 20, 2023_** to **_June 25, 2023_**, and was hosted on [Sherlock](https://www.sherlock.xyz/). Given the public nature of the contest, an appointed judge, aided by a panel of other judges and security researchers including myself, had the discretion to determine the final severity of each finding. Due to the inherently subjective nature of such judgments, the final severity assigned might differ from my own classifications. For clarity on the judge's final decisions regarding severity, please refer to [this link](https://github.com/sherlock-audit/2023-06-real-wagmi-judging/issues).

# Severity classification

| Severity                                                                                                                                                                                            | Description                                                                                                                                                                                                                                    |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ![](https://camo.githubusercontent.com/a0b140cbe7b198d62804d91996e3c09c6803cfbd484c61974af51892481f840e/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d437269746963616c2d643130623062) | A **directly** exploitable security vulnerability that leads to stolen/lost/locked/compromised assets or catastrophic denial of service.                                                                                                       |
| ![](https://camo.githubusercontent.com/77229c2f86cd6aaaaaeb3879eac44712b0e095147962e9058e455d0094f500d3/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d486967682d726564)               | A security vulnerability or bug that can affect the correct functioning of the system, lead to incorrect states or denial of service. It may not be directly exploitable or may require certain, external conditions in order to be exploited. |
| ![](https://camo.githubusercontent.com/d2cf6c2836b2143aeeb65c08b9c5aa1eb34a6fb8ad6fc55ba4345c467b64378a/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d656469756d2d6f72616e6765)     | Assets not at direct risk, but the function of the protocol or its availability could be impacted, or leak value with a hypothetical attack path with stated assumptions, but external requirements.                                           |
| ![](https://camo.githubusercontent.com/d42acfb8cb8228c156f34cb1fab83f431bf1fbebc102d922950f945b45e05587/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d696e6f722d79656c6c6f77)       | A violation of common best practices or incorrect usage of primitives, which may not currently have a major impact on security, but may do so in the future or introduce inefficiencies.                                                       |

# Scope

The review focused on the commit [hash](https://github.com/RealWagmi/concentrator/tree/dcff15564d079f5ff32686ad738873f274de48fd) and primarily covered the following contracts:

- [concentrator/contracts/Dispatcher.sol](concentrator/contracts/Dispatcher.sol)
- [concentrator/contracts/DispatcherCode.sol](concentrator/contracts/DispatcherCode.sol)
- [concentrator/contracts/Factory.sol](concentrator/contracts/Factory.sol)
- [concentrator/contracts/MultiStrategy.sol](concentrator/contracts/MultiStrategy.sol)
- [concentrator/contracts/Multipool.sol](concentrator/contracts/Multipool.sol)
- [concentrator/contracts/MultipoolCode.sol](concentrator/contracts/MultipoolCode.sol)
- [concentrator/contracts/interfaces/ICode.sol](concentrator/contracts/interfaces/ICode.sol)
- [concentrator/contracts/interfaces/IDispatcher.sol](concentrator/contracts/interfaces/IDispatcher.sol)
- [concentrator/contracts/interfaces/IFactory.sol](concentrator/contracts/interfaces/IFactory.sol)
- [concentrator/contracts/interfaces/IMultiStrategy.sol](concentrator/contracts/interfaces/IMultiStrategy.sol)
- [concentrator/contracts/interfaces/IMultipool.sol](concentrator/contracts/interfaces/IMultipool.sol)

~ 958 NSLOC in 11 files

NSLOC stands for 'Normalized Source Code', which is one of the custom measurements security researchers use when evaluating the complexity of a codebase.

The steps below are taken to get the NSLOC count of a file

1.  For all functions, reduce any multiline function declarations to a single line.
2.  Remove all comments
3.  Remove all empty lines
4.  Count the remaining lines

# About Bauchibred

**Bauchibred** is an independent smart contract security researcher. Having found numerous security vulnerabilities in various protocols, he does his best to contribute to the blockchain ecosystem and its protocols by putting time and effort into security research & reviews. Check his previous work [here](https://github.com/bauchibred/audits) or reach out on Twitter [@bauchibred](https://twitter.com/bauchibred).

# About RealWagmi

**RealWagmi** is an all-in-one platform for trading, liquidity provision, swapping, and yield strategy generation.

It's been built for multichain, so by providing liquidity on Wagmi, one can swap tokens and earn yield on their assets effortlessly, all while enjoying the benefits of the highly efficient, permissionless protocol on multiple networks.

- Website: https://wagmi.com/
- Twitter: https://twitter.com/PopsicleFinance

# Summary

This report contains **3 medium** severity issues found by **Bauchibred** during the course of the time-boxed audit.

| #      | Title                                                                               | Severity                                         | Status                                                  |
| ------ | ----------------------------------------------------------------------------------- | ------------------------------------------------ | ------------------------------------------------------- |
| [M-01] | Uniswap oracle should not be used on L2s                                            | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-02] | Wrong Assumption has been made that only one UniV3 pool exists for a pair of tokens | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-03] | TwapDuration is too low making it very easy to manipulate                           | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Acknowledged-black)   |

# Findings

## [M-01] Uniswap oracle should not be used on L2s

### Summary

RealWagmi is planned to be deployed on multiple Layer 2 (L2) networks. However, it is important to note that Uniswap advises against using their oracle on L2 networks, including Optimism and Arbitrum, due to the ease of manipulating price feeds in these environments. Therefore, it is recommended to refrain from utilizing Uniswap's oracle feature on Arbitrum until further updates or improvements are made to enhance oracle security.

### Vulnerability Detail

The information provided by the Uniswap team, as documented in the [Uniswap Oracle Integration on Layer 2 Rollups](https://docs.uniswap.org/concepts/protocol/oracle#oracles-integrations-on-layer-2-rollups) guide, primarily addresses the integration of Uniswap oracle on L2 Optimism. However, it is relevant to note that the same concerns apply to Arbitrum as well. Arbitrum's average block time is approximately 0.25 seconds, making it vulnerable to potential oracle price manipulation.

> #### Oracles Integrations on Layer 2 Rollups
>
> Optimism
> On Optimism, every transaction is confirmed as an individual block. The block.timestamp of these blocks, however, reflect the block.timestamp of the last L1 block ingested by the Sequencer. For this reason, Uniswap pools on Optimism are not suitable for providing oracle prices, as this high-latency block.timestamp update process makes the oracle much less costly to manipulate. In the future, it's possible that the Optimism block.timestamp will have much higher granularity (with a small trust assumption in the Sequencer), or that forced inclusion transactions will improve oracle security. For more information on these potential upcoming changes, please see the [Optimistic Specs repo](https://github.com/ethereum-optimism/optimistic-specs/discussions/23). **For the time being, usage of the oracle feature on Optimism should be avoided.**

### Impact

Integration of easily manipulated oracle data

### Recommendation

Until further updates or improvements are made to address the security concerns associated with Uniswap's oracle on Arbitrum, it is strongly recommended to refrain from utilizing the oracle feature in the current implementation.

## [M-02] Wrong Assumption has been made that only one UniV3 pool exists for a pair of tokens

### Vulnerability Detail

Uniswap allows for numerous pools to exist for the same token pair as long as they have different fee levels. For certain pairs (e.g. pools with 1% fee for stablecoins) some pools may exist, but have very little liquidity. This can make price manipulation very cheap , even if TWAP is used
Now the `_addUnderlyingPool` function allows for adding and selecting different pools based on the fee for a given pair of tokens. However, this could be an issue since Uniswap V3 pools with different fees can exist for the same pair of tokens, note that whenever an attempt is made to add a new pool this line reverts the execution

```solidity
        revert InvalidFee(_fee);
```

But the fact that different pools for same tokens with different fees is not taken into account, and if 2 pools exist for a pair of tokens and one has more liquidity than the other then only the one with more liquidity should be integrated with protocol to ensure stability.

### Impact

Integrating a more error prone pool to protocol

### Recommendation

The concept of different pools for the same token should be accounted for and a measure should be taken to ensure that only the right pool is registered for a pair of tokens

## [M-03] TwapDuration is too low making it very easy to manipulate

### Vulnerability Detail

RealWagmi's [Multipool.sol](https://github.com/sherlock-audit/2023-06-real-wagmi/blob/82a234a5c2c1fc1921c63265a9349b71d84675c4/concentrator/contracts/Multipool.sol) contract currently has a 150 seconds value being used as a TWAP intterval is too small

[Multipool.sol#L88](https://github.com/sherlock-audit/2023-06-real-wagmi/blob/82a234a5c2c1fc1921c63265a9349b71d84675c4/concentrator/contracts/Multipool.sol#L88)

```solidity
    uint32 public twapDuration = 150;
```

The multipool contract includes multiple instances of using twapDuration that introduce potential vulnerabilities and deviate from protocol standards.

### Impact

- A low twapDuration allows attackers to manipulate the TWAP easily, potentially leading to undesirable contract states.

- Insufficient twapDuration for Accurate TWAP Calculation: Using a twapDuration of 150 seconds obviously does not provide reliable average price representation

### Tool used

Manual Review

### Recommendation

Use a more standard duration for TWAP instead, advisably anything within the range of at least 15 minutes to 30 minutes or above
