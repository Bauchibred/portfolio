# Introduction

A time-boxed security review of the **Arrakis** protocol which was done by **Bauchibred** with a focus on the security aspects of the application's smart contracts implementation.

# Table of Contents

- [Disclaimer](#disclaimer)
- [Severity classification](#Severity-classification)
- [Scope](#scope)
- [About Bauchibred](#About-Bauchibred)
- [About Arrakis](#About-Arrakis)
- [Summary](#summary)
- [Findings](#findings)

# Disclaimer

A smart contract security review can never verify the complete absence of vulnerabilities. This is a time, resource and expertise bound effort where I try to find as many vulnerabilities as possible. I can not guarantee 100% security after the review or even if the review will find any problems with your smart contracts. Subsequent security reviews, bug bounty programs and on-chain monitoring are strongly recommended.

Additionally, this report was prepared for [this competition](https://audits.sherlock.xyz/contests/86). The competition spanned **15** days, from **_June 14, 2023_** to **_June 29, 2023_**, and was hosted on [Sherlock](https://www.sherlock.xyz/). Given the public nature of the contest, an appointed judge, aided by a panel of other judges and security researchers including myself, had the discretion to determine the final severity of each finding. Due to the inherently subjective nature of such judgments, the final severity assigned might differ from my own classifications. For clarity on the judge's final decisions regarding severity, please refer to [this link](https://github.com/sherlock-audit/2023-06-arrakis-judging/issues).

# Severity classification

| Severity                                                                                                                                                                                            | Description                                                                                                                                                                                                                                    |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ![](https://camo.githubusercontent.com/a0b140cbe7b198d62804d91996e3c09c6803cfbd484c61974af51892481f840e/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d437269746963616c2d643130623062) | A **directly** exploitable security vulnerability that leads to stolen/lost/locked/compromised assets or catastrophic denial of service.                                                                                                       |
| ![](https://camo.githubusercontent.com/77229c2f86cd6aaaaaeb3879eac44712b0e095147962e9058e455d0094f500d3/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d486967682d726564)               | A security vulnerability or bug that can affect the correct functioning of the system, lead to incorrect states or denial of service. It may not be directly exploitable or may require certain, external conditions in order to be exploited. |
| ![](https://camo.githubusercontent.com/d2cf6c2836b2143aeeb65c08b9c5aa1eb34a6fb8ad6fc55ba4345c467b64378a/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d656469756d2d6f72616e6765)     | Assets not at direct risk, but the function of the protocol or its availability could be impacted, or leak value with a hypothetical attack path with stated assumptions, but external requirements.                                           |
| ![](https://camo.githubusercontent.com/d42acfb8cb8228c156f34cb1fab83f431bf1fbebc102d922950f945b45e05587/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d696e6f722d79656c6c6f77)       | A violation of common best practices or incorrect usage of primitives, which may not currently have a major impact on security, but may do so in the future or introduce inefficiencies.                                                       |

# Scope

This review was focused on the following scope:

[v2-periphery @ ee6d7c5f3ffb212887db4ec0e595618ea418070f](https://github.com/ArrakisFinance/v2-periphery/tree/ee6d7c5f3ffb212887db4ec0e595618ea418070f)

- [v2-periphery/contracts/ArrakisV2Router.sol](v2-periphery/contracts/ArrakisV2Router.sol)
- [v2-periphery/contracts/RouterSwapExecutor.sol](v2-periphery/contracts/RouterSwapExecutor.sol)
- [v2-periphery/contracts/abstract/ArrakisV2RouterStorage.sol](v2-periphery/contracts/abstract/ArrakisV2RouterStorage.sol)

[v2-manager-templates @ 9b598356f9fb31e4fbaf07acf060e1f60409a7b0](https://github.com/ArrakisFinance/v2-manager-templates/tree/9b598356f9fb31e4fbaf07acf060e1f60409a7b0)

- [v2-manager-templates/contracts/SimpleManager.sol](v2-manager-templates/contracts/SimpleManager.sol)
- [v2-manager-templates/contracts/oracles/ChainLinkOracle.sol](v2-manager-templates/contracts/oracles/ChainLinkOracle.sol)
- [v2-manager-templates/contracts/oracles/ChainLinkOraclePivot.sol](v2-manager-templates/contracts/oracles/ChainLinkOraclePivot.sol)

[v2-core @ 9133fc412b65c7a902f62f1ad135f062e927b092](https://github.com/ArrakisFinance/v2-core/tree/9133fc412b65c7a902f62f1ad135f062e927b092)

- [v2-core/contracts/ArrakisV2.sol](v2-core/contracts/ArrakisV2.sol)
- [v2-core/contracts/ArrakisV2Beacon.sol](v2-core/contracts/ArrakisV2Beacon.sol)
- [v2-core/contracts/ArrakisV2Factory.sol](v2-core/contracts/ArrakisV2Factory.sol)
- [v2-core/contracts/abstract/ArrakisV2FactoryStorage.sol](v2-core/contracts/abstract/ArrakisV2FactoryStorage.sol)
- [v2-core/contracts/abstract/ArrakisV2Storage.sol](v2-core/contracts/abstract/ArrakisV2Storage.sol)
- [v2-core/contracts/ArrakisV2Resolver.sol](v2-core/contracts/ArrakisV2Resolver.sol)
- [v2-core/contracts/ArrakisV2Helper.sol](v2-core/contracts/ArrakisV2Helper.sol)
- [v2-core/contracts/libraries/Pool.sol](v2-core/contracts/libraries/Pool.sol)
- [v2-core/contracts/libraries/Position.sol](v2-core/contracts/libraries/Position.sol)
- [v2-core/contracts/libraries/Underlying.sol](v2-core/contracts/libraries/Underlying.sol)

~ 2432 NSLOC

NSLOC stands for 'Normalized Source Code', which is one of the custom measurements security researchers use when evaluating the complexity of a codebase.

The steps below are taken to get the NSLOC count of a file

1.  For all functions, reduce any multiline function declarations to a single line.
2.  Remove all comments
3.  Remove all empty lines
4.  Count the remaining lines

# About Bauchibred

**Bauchibred** is an independent smart contract security researcher. Having found numerous security vulnerabilities in various protocols, he does his best to contribute to the blockchain ecosystem and its protocols by putting time and effort into security research & reviews. Check his previous work [here](https://github.com/bauchibred/audits) or reach out on Twitter [@bauchibred](https://twitter.com/bauchibred).

# About Arrakis

Arrakis is a web3 trustless market making infrastructure protocol that enables running sophisticated algorithmic strategies on Uniswap V3, In sense that liquidity providers can utilize Arrakis Vaults to have their liquidity transparently managed in an automated, capital efficient, non-custodial and manner.

- Website: https://www.arrakis.finance/
- Twitter: https://twitter.com/arrakisfinance

# Summary

This report contains **4 medium** severity issues and **3 minor** issues found by **Bauchibred** during the course of the time-boxed audit.

| #      | Title                                                                                      | Severity                                         | Status                                                  |
| ------ | ------------------------------------------------------------------------------------------ | ------------------------------------------------ | ------------------------------------------------------- |
| [M-01] | Uniswap oracles should not be used on L2s                                                  | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-02] | Missing slippage parameter while minting                                                   | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-03] | Using `slot0` to determine deviation is not ideal                                          | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-04] | Data received from chainlink is not successfully checked to be within the valid boundaries | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-01] | Off-by-one error while retrieving the list of vaults created by ArrakisV2Factory           | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-02] | Inconsistent handling of Chainlink integrations                                            | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-03] | Use of Solidity version 0.8.13 which has known issues applicable to Arrakis                | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Resolved-brightgreen) |

# Findings

## [M-01] Uniswap oracles should not be used on L2s

### Summary

Arrakis is planned to be deployed on multiple Layer 2 (L2) networks. However, it is important to note that Uniswap advises against using their oracle on L2 networks, including Optimism and Arbitrum, due to the ease of manipulating price feeds in these environments. Therefore, it is recommended to refrain from utilizing Uniswap's oracle feature on Arbitrum until further updates or improvements are made to enhance oracle security.

### Vulnerability Detail

The information provided by the Uniswap team, as documented in the [Uniswap Oracle Integration on Layer 2 Rollups](https://docs.uniswap.org/concepts/protocol/oracle#oracles-integrations-on-layer-2-rollups) guide, primarily addresses the integration of Uniswap oracle on L2 Optimism. However, it is relevant to note that the same concerns apply to Arbitrum as well. Arbitrum's average block time is approximately 0.25 seconds, making it vulnerable to potential oracle price manipulation.

> #### Oracles Integrations on Layer 2 Rollups
>
> Optimism
> On Optimism, every transaction is confirmed as an individual block. The block.timestamp of these blocks, however, reflect the block.timestamp of the last L1 block ingested by the Sequencer. For this reason, Uniswap pools on Optimism are not suitable for providing oracle prices, as this high-latency block.timestamp update process makes the oracle much less costly to manipulate. In the future, it's possible that the Optimism block.timestamp will have much higher granularity (with a small trust assumption in the Sequencer), or that forced inclusion transactions will improve oracle security. For more information on these potential upcoming changes, please see the [Optimistic Specs repo](https://github.com/ethereum-optimism/optimistic-specs/discussions/23). **For the time being, usage of the oracle feature on Optimism should be avoided.**

### Impact

Easily Manipulated Oracle Data: Due to the specific characteristics of L2 networks, such as high-latency block.timestamp update processes, the Uniswap oracle becomes vulnerable to price manipulation. This manipulation can lead to inaccurate and unreliable price feeds, potentially resulting in significant financial losses for users relying on these price references.

Key to note that this issue affects the whole [UniswapV3PoolOracle.sol]](https://github.com/sherlock-audit/2023-06-arrakis-Bauchibred/blob/23cb020a4c92462a2476744f841de0fe624d974e/v2-manager-templates/contracts/oracles/UniswapV3PoolOracle.sol#L7-L33) contract.

### Recommendation

Until further updates or improvements are made to address the security concerns associated with Uniswap's oracle on Arbitrum, it is strongly recommended to refrain from utilizing the oracle feature in the current implementation.

## [M-02] Missing slippage parameter while minting

### Vulnerability Details

Missing slippage parameter while minting makes it vulnerable to front-run attacks and exposes users to unwanted slippage.

The current implementation of minting functions or it sister functions lack a parameter for controlling slippage, which makes them vulnerable to front-run attacks. Transactions involving large volumes are particularly at risk, as the minting process can be manipulated, resulting in price impact.

Take a look at the ArrakisV2 vault, when minting Arrakis V2 shares, the underlying assets are deposited to UniswapV3 pool to provide liquidity. However, there is no slippage protection.

https://github.com/sherlock-audit/2023-06-arrakis/blob/9594cf930307ebbfe5cae4f8ad9e9b40b26c9fec/v2-core/contracts/ArrakisV2.sol#L149)

```soldity
                pool.mint(me, range.lowerTick, range.upperTick, liquidity, "");
```

### Impact

Users exposed to uncontrolable slippage risks

Do note that the ug is present is present in multiple instances, including but not exclusive to: [1](https://github.com/sherlock-audit/2023-06-arrakis/blob/9594cf930307ebbfe5cae4f8ad9e9b40b26c9fec/v2-core/contracts/ArrakisV2Resolver.sol#L137-L207), [2](https://github.com/sherlock-audit/2023-06-arrakis/blob/9594cf930307ebbfe5cae4f8ad9e9b40b26c9fec/v2-core/contracts/ArrakisV2.sol#L398), etc...

### Recommendation

Consider adding a minAmountOut parameter.

## [M-03] Using `slot0` to determine deviation is not ideal

### Summary

The `rebalance` function in the SimpleManager contract utilizes the most recent price point `slot0` to determine the current pool price and subsequently, the deviation from the oracle price. However, `slot0` is the most recent data point and is therefore extremely easy to manipulate, meaning that the price deviation might be inaccurate, which could potentially lead to incorrect rebalancing and Denial of Service (DoS) due to failure of deviation checks.

```solidity
require(deviation <= maxDeviation_, "maxDeviation");
```

### Vulnerability Detail

In the SimpleManager.sol contract, the `rebalance` function retrieves the current pool price using `slot0` that represents the most recent price point. The function `_checkDeviation` then calculates the deviation of this current price from the oracle price.

Given `slot0`'s susceptibility to manipulation, the deviation calculation might be skewed. This is particularly crucial because the deviation is extensively used in the protocol to maintain balance and perform vital operations.

For example, if the deviation is larger than the `maxDeviation_` parameter in `_checkDeviation` function, the function fails, potentially causing a DoS in the contract. This is due to the line `require(deviation <= maxDeviation_, "maxDeviation");` in the `_checkDeviation` function.

### Impact

The usage of `slot0` to determine deviation could potentially allow malicious actors to manipulate the deviation calculation by altering the most recent price. As a consequence, this might lead to incorrect rebalancing operations, resulting in an inaccurate state of the contract, if the deviation check fails due to the manipulated deviation exceeding the maximum allowed deviation.

Below is the affected code snippet

- [rebalance()](https://github.com/sherlock-audit/2023-06-arrakis/blob/9594cf930307ebbfe5cae4f8ad9e9b40b26c9fec/v2-manager-templates/contracts/SimpleManager.sol#L128-L214) and [`_checkDeviation()`](https://github.com/sherlock-audit/2023-06-arrakis/blob/9594cf930307ebbfe5cae4f8ad9e9b40b26c9fec/v2-manager-templates/contracts/SimpleManager.sol#L366-L385)

```solidity
function rebalance(
    address vault_,
    Rebalance calldata rebalanceParams_
) external {
    ...
    IUniswapV3Pool pool = IUniswapV3Pool(
        _getPool(
            token0,
            token1,
            rebalanceParams_.mints[i].range.feeTier
        )
    );
    uint256 sqrtPriceX96;
    (sqrtPriceX96, , , , , , ) = pool.slot0();
    uint256 poolPrice = FullMath.mulDiv(
        sqrtPriceX96 * sqrtPriceX96,
        10 ** token0Decimals,
        2 ** 192
    );
    _checkDeviation(
        poolPrice,
        oraclePrice,
        vaultInfo.maxDeviation,
        token1Decimals
    );
    ...ommited for brevity
}
function _checkDeviation(
    uint256 currentPrice_,
    uint256 oraclePrice_,
    uint24 maxDeviation_,
    uint8 priceDecimals_
) internal pure {
    ...ommited for brevity
    require(deviation <= maxDeviation_, "maxDeviation");
}
```

### Recommendation

Considering the potential risks associated with using `slot0` to calculate deviation, implementing a Time-Weighted Average Price (TWAP) to determine the price is recommended. By providing a more accurate and harder to manipulate price point, TWAP would yield a more accurate deviation calculation. This would reduce the possibility of incorrect rebalancing and the risk of DoS attacks.

> NB: As the Uniswap team have warned [here](https://docs.uniswap.org/concepts/protocol/oracle#oracles-integrations-on-layer-2-rollups) there are issues if TWAP is going to be implemented on an L2 and these should be taken into account.

## [M-04] Data received from chainlink is not successfully checked to be within the valid boundaries

### Summary

The `ChainLinkOraclePivot` and `ChainlinkOracle` contracts do good jobs in making sure the data gotten from the chainlink oracle are correct or the call reverts, this cocnclusion is reached since the check for outdated price and sequencer are implemented, the `ChainlinkOracle` contract even employs safecast to protect against negative prices when executing the `latestRoundDat`a function. Nevertheless, it does not incorporate a mechanism to verify that the returned prices do not hit the extreme boundaries `(minAnswer and maxAnswer)`. The absence of this mechanism may cause the contract to operate based on incorrect prices, potentially leading to an over- or under-estimation of the asset's value, which could significantly affect the protocol's financial stability.

### Vulnerability Detail

The Chainlink aggregators feature an integrated circuit breaker that is triggered when an asset's price falls outside of a predefined price band. In an event where an asset's value experiences a substantial drop (akin to the LUNA crash), the oracle price will persist in returning the minPrice, instead of the actual market price of the asset. This allows users to continue borrowing against the asset at an incorrect price, analogous to what occurred with [Venus on BSC during the LUNA collapse](https://rekt.news/venus-blizz-rekt/).

In its existing form, the [ `_getLatestRoundData`](https://github.com/sherlock-audit/2023-06-arrakis/blob/9594cf930307ebbfe5cae4f8ad9e9b40b26c9fec/v2-manager-templates/contracts/oracles/ChainLinkOraclePivot.sol#L229-L275) function within the ChainLinkOraclePivot contract obtains the latest round data from the Chainlink Feed. If the asset's market price falls below minAnswer or exceeds maxAnswer, the returned price will still be the minAnswer or maxAnswer, respectively, rather than the actual market price. This can lead to a situation where the protocol transacts with the asset using incorrect pricing data, thereby potentially enabling exploitative behavior.

### Impact

The risk is apparent when the actual price of an asset changes drastically, but the oracle continues to function using the minAnswer or maxAnswer as the asset's price. This situation would obviously allow manipulative actions.

Affected code snippet:

- [ `_getLatestRoundData`](https://github.com/sherlock-audit/2023-06-arrakis/blob/9594cf930307ebbfe5cae4f8ad9e9b40b26c9fec/v2-manager-templates/contracts/oracles/ChainLinkOraclePivot.sol#L229-L275)

```solidity
    function _getLatestRoundData()
        internal
        view
        returns (
            uint256 priceA,
            uint256 priceB,
            uint8 priceFeedADecimals,
            uint8 priceFeedBDecimals
        )
    {
        try priceFeedA.latestRoundData() returns (
            uint80,
            int256 price,
            uint256,
            uint256 updatedAt,
            uint80
        ) {
            require(
                block.timestamp - updatedAt <= outdated, // solhint-disable-line not-rely-on-time
                "ChainLinkOracle: priceFeedA outdated."
            );


            priceA = SafeCast.toUint256(price);
        } catch {
            revert("ChainLinkOracle: price feed A call failed.");
        }


        try priceFeedB.latestRoundData() returns (
            uint80,
            int256 price,
            uint256,
            uint256 updatedAt,
            uint80
        ) {
            require(
                block.timestamp - updatedAt <= outdated, // solhint-disable-line not-rely-on-time
                "ChainLinkOracle: priceFeedB outdated."
            );


            priceB = SafeCast.toUint256(price);
        } catch {
            revert("ChainLinkOracle: price feed B call failed.");
        }


        priceFeedADecimals = priceFeedA.decimals();
        priceFeedBDecimals = priceFeedB.decimals();
    }
```

### Tool used

Manual Review

### Recommendation

The [ `_getLatestRoundData`](https://github.com/sherlock-audit/2023-06-arrakis/blob/9594cf930307ebbfe5cae4f8ad9e9b40b26c9fec/v2-manager-templates/contracts/oracles/ChainLinkOraclePivot.sol#L229-L275) function should be adjusted to include a validation check. If the returned price equals minAnswer or maxAnswer, the function should revert to avoid operating on potentially incorrect prices. This could be implemented in a similar way to this:

```diff
// @audit-fix M implement check for minAnswer/maxAnswer case
function _getLatestRoundData()
    internal
    view
    returns (
        uint256 priceA,
        uint256 priceB,
        uint8 priceFeedADecimals,
        uint8 priceFeedBDecimals
    )
{
    try priceFeedA.latestRoundData() returns (
        uint80,
        int256 price,
        uint256,
        uint256 updatedAt,
        uint80
    ) {
        require(
            block.timestamp - updatedAt <= outdated, // solhint-disable-line not-rely-on-time
            "ChainLinkOracle: priceFeedA outdated."
        );
+        require(price > minAnswer && price < maxAnswer, "price outside valid range");

        priceA = SafeCast.toUint256(price);
    } catch {
        revert("ChainLinkOracle: price feed A call failed.");
    }
    ...
}
```

## [L-01] Off-by-one error while retrieving the list of vaults created by ArrakisV2Factory

### Summary

The `vaults` function of the [ArrakisV2Factory.sol](https://github.com/sherlock-audit/2023-06-arrakis/blob/main/v2-core/contracts/ArrakisV2Factory.sol#L23) contract, when invoked is to generate a list of vaults within a certain range, but it always returns one less vault than expected due to a one-off error in calculating the size of the array.

### Vulnerability Detail

The `vaults` function creates an array `vs` of vault addresses with its length computed as `endIndex_ - startIndex_`. This calculation fails to account for the inclusion of the vault at `endIndex_`, thus leading to an omission of the final vault from the output.
Take a look at the [vaults() function:](https://github.com/sherlock-audit/2023-06-arrakis/blob/9594cf930307ebbfe5cae4f8ad9e9b40b26c9fec/v2-core/contracts/ArrakisV2Factory.sol#L59-L82)

```solidity
    /// @notice get a list of vaults created by this factory
    /// @param startIndex_ start index
    /// @param endIndex_ end index
    /// @return vaults list of all created vaults.
    function vaults(uint256 startIndex_, uint256 endIndex_)
        external
        view
        returns (address[] memory)
    {
        require(
            startIndex_ < endIndex_,
            "start index is equal or greater than end index."
        );
        require(
            endIndex_ <= numVaults(),
            "end index is greater than vaults array length"
        );
        address[] memory vs = new address[](endIndex_ - startIndex_);
        for (uint256 i = startIndex_; i < endIndex_; i++) {
            vs[i - startIndex_] = _vaults.at(i);
        }


        return vs;
    }
```

[L76-79:](https://github.com/sherlock-audit/2023-06-arrakis/blob/9594cf930307ebbfe5cae4f8ad9e9b40b26c9fec/v2-core/contracts/ArrakisV2Factory.sol#L76-L79)

```solidity
address[] memory vs = new address[](endIndex_ - startIndex_);
for (uint256 i = startIndex_; i < endIndex_; i++) {
    vs[i - startIndex_] = _vaults.at(i);
}
```

#### POC

The code snippet below demonstrates a minimalistic proof-of-concept for the contract. Here, vault addresses are simplified as 0x0, 0x1, ..., 0x63. Using the `testVaults()` function to retrieve vaults from index 1 to 10, the function should ideally return an array of 10 addresses but it erroneously returns only 9.

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

contract ArrakisV2Factory {
    address[] private vaults;

    constructor() {
        // Assuming addresses as 0x0, 0x1, ..., 0x63 for simplicity
        for (uint i = 1; i <= 100; i++) {
            vaults.push(address(uint160(i)));
        }
    }

    function numVaults() public view returns (uint256 result) {
        return vaults.length;
    }

    function getVaults(uint256 startIndex_, uint256 endIndex_)
        public
        view
        returns (address[] memory)
    {
        require(
            startIndex_ < endIndex_,
            "start index is equal or greater than end index."
        );
        require(
            endIndex_ <= numVaults(),
            "end index is greater than vaults array length"
        );
        address[] memory vs = new address[](endIndex_ - startIndex_);
        for (uint256 i = startIndex_; i < endIndex_; i++) {
            vs[i - startIndex_] = vaults[i];
        }
        return vs;
    }

    function testVaults() public view returns (address[] memory) {
        // If we try to get the list of vaults from index 1 to 10,
        // it should return 10 vaults, but due to the bug it only returns 9.
        return this.getVaults(1, 10);
    }
}

```

This contract can be copy-pasted onto remix and the `testVaults()` should be called

### Impact

This off-by-one error in the `vaults` function would result in incomplete or inaccurate data retrieval, as the function does not return the full list of vaults within the given index range.

### Recommendation

The recommended solution would be to adjust the `vs` array's length to include the vault at `endIndex_`. This could be done by altering the length of the array to `endIndex_ - startIndex_ + 1`.

```solidity
address[] memory vs = new address[](endIndex_ - startIndex_ + 1);
```

In addition, adjustments are needed in the `require` statements and the `for` loop to ensure the `endIndex_` is included.

```solidity
require(
    endIndex_ < numVaults(),
    "end index is greater than or equal to vaults array length"
);
for (uint256 i = startIndex_; i <= endIndex_; i++) {
    vs[i - startIndex_] = _vaults.at(i);
}
```

This ensures that the `vaults` function correctly returns the complete list of vaults within the specified index range.

## [L-02] Inconsistent handling of Chainlink integrations

### Summary

The use of Chainlink's `latestRoundData()` function is widespread in the `ChainlinkOracle` contract, where it serves to fetch data from Chainlink's oracle. As noted in [OpenZeppelin's secure smart contract guidelines](https://blog.openzeppelin.com/secure-smart-contract-guidelines-the-dangers-of-price-oracles), queries to Chainlink may be reverted for a variety of reasons, including the decision by multisigs to block the query. Indeed, most usages of `latestRoundData()` in the contract are wrapped in a try/catch block, indicating an awareness of this potential issue. Unfortunately, this is not the case for all instances; specifically, the `ChainlinkAdapterOracle::_checkSequencer()` function lacks such a safeguard. This is a concern because if the multisigs decide to block the query, a permanent inability to query about the sequencer could lead to a Denial of Service (DoS), rendering critical functionality of the contract unavailable.

### Vulnerability Detail

Although most instances of `latestRoundData()` in the contract implement a try/catch block to anticipate possible reverts—whether from the oracle going down or from the multisigs blocking the query—this is not the case for `ChainlinkAdapterOracle::_checkSequencer()`. This is a significant oversight, as Chainlink's multisigs could potentially block access to their feeds at any time. This function, in particular, is externally called by `getPrice0()/getPrice1()` for optimistic L2 chains, indicating the importance of implementing safeguards in this context. Here's the code in question: [ChainLinkOracle.sol#L148-L161](https://github.com/sherlock-audit/2023-06-arrakis/blob/9594cf930307ebbfe5cae4f8ad9e9b40b26c9fec/v2-manager-templates/contracts/oracles/ChainLinkOracle.sol#L148-L161).

There are multiple instances where the try/catch block is rightly implemented on calls to `latestRoundData()`, which further underscores the anomaly in the `_checkSequencer()` function. If access to the oracle is blocked or the oracle goes down, the queries revert with the error `revert("ChainLinkOracle: price feed B call failed.");`. This protective measure is not present when calling the sequencer, suggesting an oversight by the developers.

[ChainLinkOracle.sol#L148-L161](https://github.com/sherlock-audit/2023-06-arrakis/blob/9594cf930307ebbfe5cae4f8ad9e9b40b26c9fec/v2-manager-templates/contracts/oracles/ChainLinkOracle.sol#L148-L161)

```solidity
    /// @dev only needed for optimistic L2 chain
    function _checkSequencer() internal view {
        (, int256 answer, uint256 startedAt, , ) = sequencerUptimeFeed
            .latestRoundData();

        require(answer == 0, "ChainLinkOracle: sequencer down");

        // Make sure the grace period has passed after the
        // sequencer is back up.
        require(
            block.timestamp - startedAt > GRACE_PERIOD_TIME, // solhint-disable-line not-rely-on-time, max-line-length
            "ChainLinkOracle: grace period not over"
        );
    }
}
```

Here are other instances where the try/catch is rightly implemented on the calls to latestRoundData():
[ChainLinkOracle.sol#L68-L146](https://github.com/sherlock-audit/2023-06-arrakis/blob/9594cf930307ebbfe5cae4f8ad9e9b40b26c9fec/v2-manager-templates/contracts/oracles/ChainLinkOracle.sol#L68-L146)

```solidity
    function getPrice0() external view override returns (uint256 price0) {


        try priceFeed.latestRoundData() returns (
******
        ) {
            require(
                block.timestamp - updatedAt <= outdated, // solhint-disable-line not-rely-on-time
                "ChainLinkOracle: outdated."
            );

******

        } catch {
            revert("ChainLinkOracle: price feed call failed.");
        }
    }

//NB:multiple code blocks ommited for brevity of report
    function getPrice1() external view override returns (uint256 price1) {

        try priceFeed.latestRoundData() returns (
******
        ) {
            require(
                block.timestamp - updatedAt <= outdated, // solhint-disable-line not-rely-on-time
                "ChainLinkOracle: outdated."
            );


******
        } catch {
            revert("ChainLinkOracle: price feed call failed.");
        }
    }
```

### Impact

The absence of a try/catch block in the `ChainlinkAdapterOracle::_checkSequencer()` function can lead to a worst-case scenario where a Denial of Service (DoS) to the protocol occurs if Chainlink's multisigs decide to deny access to the sequencer feed. Given that the smart contract does not have a fallback mechanism in this instance.

### Recommendation

The development team should consider implementing a try/catch block around the `latestRoundData()` call in the `ChainlinkAdapterOracle::_checkSequencer()` function. This approach is consistent with how the code handles potential reverts in other instances where the `latestRoundData()` function is used. This change would provide a fail-safe if access to the Chainlink data feed is denied, increasing the robustness of the contract and ensuring its continuous operation even in the face of potential oracle or multisig, also very important the team also implements a fallback logic, could be as simple as the cases in the `getPrice0()/getPrice1()` functions.

NB: A sister issue to this has been submiteed in one of the previous audits, source [here](https://gist.github.com/kassandraoftroy/25f7208adb770abee9f46978326cfb3f#2-improper-chainlink-oracle-handling), which was announced to be fixed, but this report proves otherwise since it wasn't protected against in all instances

## [L-03] Use of Solidity version 0.8.13 which has known issues applicable to Arrakis

### Summary

The extensive usage of Solidity version 0.8.13 in the development of ArrakisFinance contracts, including ArrakisV2 and ChainLinkOraclePivot among others, has unveiled notable risks due to known vulnerabilities associated with this specific compiler version. These vulnerabilities pertain mainly to two issues:

1. **Optimizer Bug Regarding Memory Side Effects of Inline Assembly**: OpenZeppelin contracts, from which ArrakisFinance contracts inherit, make use of inline assembly. The Solidity 0.8.13 version, when used with optimization, has been found to contain bugs associated with memory side effects of inline assembly (Reference: [Solidity 0.8.15 Release Announcement](https://blog.soliditylang.org/2022/06/15/solidity-0.8.15-release-announcement/)).

2. **ABI-encoding Vulnerability**: There are also reported bugs related to ABI-encoding in this version of the compiler (Reference: [Solidity 0.8.14 Release Announcement](https://blog.soliditylang.org/2022/05/18/solidity-0.8.14-release-announcement/)). This can be seen from the use of abi.encoding in [Position.sol](https://github.com/sherlock-audit/2023-06-arrakis/blob/9594cf930307ebbfe5cae4f8ad9e9b40b26c9fec/v2-core/contracts/libraries/Position.sol#L26), [ArrakisV2Factory.sol](https://github.com/sherlock-audit/2023-06-arrakis/blob/9594cf930307ebbfe5cae4f8ad9e9b40b26c9fec/v2-core/contracts/ArrakisV2Factory.sol#L111), etc.

NB: This report is more inclined to the first issue and the second is just attached

### Vulnerability Detail

The following details provide further context on the vulnerabilities:

1. **Optimizer Bug Regarding Memory Side Effects of Inline Assembly**: This bug pertains to a malfunction in the compiler when handling inline assembly, leading to potential memory issues that can affect the correctness of the contract execution. Detailed information about this bug can be found in this official Solidity post: [Optimizer Bug Regarding Memory Side Effects of Inline Assembly](https://blog.soliditylang.org/2022/06/15/inline-assembly-memory-side-effects-bug/).

2. **ABI-encoding Vulnerability**: This vulnerability revolves around the mishandling of ABI-encoded data which could lead to incorrect transaction processing or potential security issues. Take a look at the [Solidity 0.8.14 Release Announcement](https://blog.soliditylang.org/2022/05/18/solidity-0.8.14-release-announcement/)

### Impact

The presence of these bugs could lead to incorrect computation and data handling, given the sensitive nature of inline assembly, any malfunctions could lead to unpredictable contract behaviors.

The vulnerabilities can be found in multiple instances, including but not limited to:

- [ChainLinkOraclePivot.sol](https://github.com/sherlock-audit/2023-06-arrakis/blob/9594cf930307ebbfe5cae4f8ad9e9b40b26c9fec/v2-manager-templates/contracts/oracles/ChainLinkOraclePivot.sol#L2-L4)
- [ArrakisV2.sol](https://github.com/sherlock-audit/2023-06-arrakis/blob/9594cf930307ebbfe5cae4f8ad9e9b40b26c9fec/v2-core/contracts/ArrakisV2.sol#L2-L4)
- [Position.sol](https://github.com/sherlock-audit/2023-06-arrakis/blob/9594cf930307ebbfe5cae4f8ad9e9b40b26c9fec/v2-core/contracts/libraries/Position.sol#L26)
- [ArrakisV2Factory.sol](https://github.com/sherlock-audit/2023-06-arrakis/blob/9594cf930307ebbfe5cae4f8ad9e9b40b26c9fec/v2-core/contracts/ArrakisV2Factory.sol#L111)

### Recommendation

The simplest and most effective mitigation strategy is to transition to a more recent, secure version of the Solidity compiler devoid of the identified bugs.
