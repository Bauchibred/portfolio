# Introduction

A time-boxed security review of the **Iron Bank** protocol which was done by **Bauchibred** with a focus on the security aspects of the application's smart contracts implementation.

# Table of Contents

- [Disclaimer](#disclaimer)
- [Severity classification](#Severity-classification)
- [Scope](#scope)
- [About Bauchibred](#About-Bauchibred)
- [About Iron Bank](#About-Iron-Bank)
- [Summary](#summary)
- [Findings](#findings)

# Disclaimer

A smart contract security review can never verify the complete absence of vulnerabilities. This is a time, resource and expertise bound effort where I try to find as many vulnerabilities as possible. I can not guarantee 100% security after the review or even if the review will find any problems with your smart contracts. Subsequent security reviews, bug bounty programs and on-chain monitoring are strongly recommended.

Additionally, this report was prepared for [this competition](https://audits.sherlock.xyz/contests/84). The competition spanned **13** days, from **_May 29, 2023_** to **_June 11, 2023_**, and was hosted on [Sherlock](https://www.sherlock.xyz/). Given the public nature of the contest, an appointed judge, aided by a panel of other judges and security researchers including myself, had the discretion to determine the final severity of each finding. Due to the inherently subjective nature of such judgments, the final severity assigned might differ from my own classifications. For clarity on the judge's final decisions regarding severity, please refer to [this link](https://github.com/sherlock-audit/2023-05-ironbank-judging).

# Severity classification

| Severity                                                                                                                                                                                            | Description                                                                                                                                                                                                                                    |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ![](https://camo.githubusercontent.com/a0b140cbe7b198d62804d91996e3c09c6803cfbd484c61974af51892481f840e/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d437269746963616c2d643130623062) | A **directly** exploitable security vulnerability that leads to stolen/lost/locked/compromised assets or catastrophic denial of service.                                                                                                       |
| ![](https://camo.githubusercontent.com/77229c2f86cd6aaaaaeb3879eac44712b0e095147962e9058e455d0094f500d3/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d486967682d726564)               | A security vulnerability or bug that can affect the correct functioning of the system, lead to incorrect states or denial of service. It may not be directly exploitable or may require certain, external conditions in order to be exploited. |
| ![](https://camo.githubusercontent.com/d2cf6c2836b2143aeeb65c08b9c5aa1eb34a6fb8ad6fc55ba4345c467b64378a/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d656469756d2d6f72616e6765)     | Assets not at direct risk, but the function of the protocol or its availability could be impacted, or leak value with a hypothetical attack path with stated assumptions, but external requirements.                                           |
| ![](https://camo.githubusercontent.com/d42acfb8cb8228c156f34cb1fab83f431bf1fbebc102d922950f945b45e05587/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d696e6f722d79656c6c6f77)       | A violation of common best practices or incorrect usage of primitives, which may not currently have a major impact on security, but may do so in the future or introduce inefficiencies.                                                       |

# Scope

The review focused on the commit [hash](https://github.com/ibdotxyz/ib-v2/tree/66c70f3f58a1dc1e07908b4dae2f55c30e3b7edd) and primarily covered the following contracts:

- [ib-v2/src/extensions/TxBuilderExtension.sol](ib-v2/src/extensions/TxBuilderExtension.sol)
- [ib-v2/src/extensions/UniswapExtension.sol](ib-v2/src/extensions/UniswapExtension.sol)
- [ib-v2/src/extensions/libraries/UniswapV2Utils.sol](ib-v2/src/extensions/libraries/UniswapV2Utils.sol)
- [ib-v2/src/extensions/libraries/UniswapV3Utils.sol](ib-v2/src/extensions/libraries/UniswapV3Utils.sol)
- [ib-v2/src/flashLoan/FlashLoan.sol](ib-v2/src/flashLoan/FlashLoan.sol)
- [ib-v2/src/libraries/Arrays.sol](ib-v2/src/libraries/Arrays.sol)
- [ib-v2/src/libraries/DataTypes.sol](ib-v2/src/libraries/DataTypes.sol)
- [ib-v2/src/libraries/PauseFlags.sol](ib-v2/src/libraries/PauseFlags.sol)
- [ib-v2/src/protocol/oracle/PriceOracle.sol](ib-v2/src/protocol/oracle/PriceOracle.sol)
- [ib-v2/src/protocol/pool/Constants.sol](ib-v2/src/protocol/pool/Constants.sol)
- [ib-v2/src/protocol/pool/CreditLimitManager.sol](ib-v2/src/protocol/pool/CreditLimitManager.sol)
- [ib-v2/src/protocol/pool/Events.sol](ib-v2/src/protocol/pool/Events.sol)
- [ib-v2/src/protocol/pool/IronBank.sol](ib-v2/src/protocol/pool/IronBank.sol)
- [ib-v2/src/protocol/pool/IronBankProxy.sol](ib-v2/src/protocol/pool/IronBankProxy.sol)
- [ib-v2/src/protocol/pool/IronBankStorage.sol](ib-v2/src/protocol/pool/IronBankStorage.sol)
- [ib-v2/src/protocol/pool/MarketConfigurator.sol](ib-v2/src/protocol/pool/MarketConfigurator.sol)
- [ib-v2/src/protocol/pool/interest-rate-model/TripleSlopeRateModel.sol](ib-v2/src/protocol/pool/interest-rate-model/TripleSlopeRateModel.sol)
- [ib-v2/src/protocol/token/DebtToken.sol](ib-v2/src/protocol/token/DebtToken.sol)
- [ib-v2/src/protocol/token/DebtTokenStorage.sol](ib-v2/src/protocol/token/DebtTokenStorage.sol)
- [ib-v2/src/protocol/token/IBToken.sol](ib-v2/src/protocol/token/IBToken.sol)
- [ib-v2/src/protocol/token/IBTokenStorage.sol](ib-v2/src/protocol/token/IBTokenStorage.sol)
- [ib-v2/src/protocol/token/PToken.sol](ib-v2/src/protocol/token/PToken.sol)
- [ib-v2/src/interfaces/DebtTokenInterface.sol](ib-v2/src/interfaces/DebtTokenInterface.sol)
- [ib-v2/src/interfaces/DeferLiquidityCheckInterface.sol](ib-v2/src/interfaces/DeferLiquidityCheckInterface.sol)
- [ib-v2/src/interfaces/IBTokenInterface.sol](ib-v2/src/interfaces/IBTokenInterface.sol)
- [ib-v2/src/interfaces/InterestRateModelInterface.sol](ib-v2/src/interfaces/InterestRateModelInterface.sol)
- [ib-v2/src/interfaces/IronBankInterface.sol](ib-v2/src/interfaces/IronBankInterface.sol)
- [ib-v2/src/interfaces/PTokenInterface.sol](ib-v2/src/interfaces/PTokenInterface.sol)
- [ib-v2/src/interfaces/PriceOracleInterface.sol](ib-v2/src/interfaces/PriceOracleInterface.sol)

~ 2250 NSLOC

NSLOC stands for 'Normalized Source Code', which is one of the custom measurements security researchers use when evaluating the complexity of a codebase.

The steps below are taken to get the NSLOC count of a file

1.  For all functions, reduce any multiline function declarations to a single line.
2.  Remove all comments
3.  Remove all empty lines
4.  Count the remaining lines

# About Bauchibred

**Bauchibred** is an independent smart contract security researcher. Having found numerous security vulnerabilities in various protocols, he does his best to contribute to the blockchain ecosystem and its protocols by putting time and effort into security research & reviews. Check his previous work [here](https://github.com/bauchibred/audits) or reach out on Twitter [@bauchibred](https://twitter.com/bauchibred).

# About Iron Bank

**Iron Bank** is a decentralized lending platform focused on capital efficiency allowing protocols and individuals to supply and borrow cryptoassets.

- Website: https://linktr.ee/ibdotxyz
- Twitter: https://twitter.com/ibdotxyz

# Summary

This report contains **3 medium** severity issues and **2 minor** issues found by **Bauchibred** during the course of the time-boxed audit.

| #      | Title                                                                                                                                 | Severity                                         | Status                                                  |
| ------ | ------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ | ------------------------------------------------------- |
| [M-01] | Lack of `minAnswer/maxAnswer` Circuit Breaker Checks while Querying Prices in PriceOracle.sol                                         | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-02] | Possibility of Stale Data Usage in the PriceOracle Contract                                                                           | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-03] | Lack of Implementation of Fallback Oracles which Could Cause Protocol-wide Failure if the Chainlink Oracle Goes Down or Reverts Calls | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-01] | Users Might Face Immediate Liquidation Post Loan Acquisition                                                                          | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-02] | Unnecessary Precision Loss in `TripleSlopeRateModel`                                                                                  | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Resolved-brightgreen) |

# Findings

## [M-01] Lack of `minAnswer/maxAnswer` Circuit Breaker Checks while Querying Prices in PriceOracle.sol

### Summary

The PriceOracle contract, while currently applying a safety check to ensure returned prices are greater than zero, which is commendable, as it effectively mitigates the risk of using negative prices, there should be an implementation to ensure the returned prices are not at the extreme boundaries (`minAnswer` and `maxAnswer`).
Without such a mechanism, the contract could operate based on incorrect prices, which could lead to an over- or under-representation of the asset's value, potentially causing significant harm to the protocol.

### Vulnerability Details

Chainlink aggregators have a built in circuit breaker if the price of an asset goes outside of a predetermined price band. The result is that if an asset experiences a huge drop in value (i.e. LUNA crash) the price of the oracle will continue to return the minPrice instead of the actual price of the asset. This would allow user to continue borrowing with the asset but at the wrong price. This is exactly what happened to [Venus on BSC when LUNA imploded](https://rekt.news/venus-blizz-rekt/).
In its current form, the `getPriceFromChainlink` function within the PriceOracle contract retrieves the latest round data from the Chainlink Feed Registry. If the asset's market price plummets below `minAnswer` or skyrockets above `maxAnswer`, the returned price will still be `minAnswer` or `maxAnswer`, respectively, rather than the actual market price. This could potentially lead to an exploitation scenario where the protocol interacts with the asset using incorrect price information.

Take a look at [PriceOracle.sol#L60-L72](https://github.com/sherlock-audit/2023-05-ironbank/blob/9ebf1702b2163b55479624794ab7999392367d2a/ib-v2/src/protocol/oracle/PriceOracle.sol#L60-L72):

```solidity
function getPriceFromChainlink(address base, address quote) internal view returns (uint256) {
    (, int256 price,,,) = registry.latestRoundData(base, quote);
    require(price > 0, "invalid price");

    // Extend the decimals to 1e18.
    return uint256(price) * 10 ** (18 - uint256(registry.decimals(base, quote)));
}
```

#### Illustration:

Suppose TokenA has a minimum price set at $1. If the actual price of TokenA dips to $0.10, the aggregator continues to report $1 as the price. Consequently, users can interact with protocol using TokenA as though it were still valued at $1, which is a tenfold overestimate of its real market value.

### Impact

The potential for misuse arises when the actual price of an asset drastically changes but the oracle continues to operate using the `minAnswer` or `maxAnswer` as the asset's price. This situation might enable manipulative behaviours, such as borrowing excessive funds against an overvalued asset or selling undervalued assets, posing a risk to the protocol's financial stability.

### Recommendation

The `getPriceFromChainlink` function should be modified to include a validation check. If the returned price is equal to `minAnswer` or `maxAnswer`, the function should revert to avoid operating on potentially incorrect prices. This can be implemented as a case similarly to this:

```solidity
function getPriceFromChainlink(address base, address quote) internal view returns (uint256) {
    (, int256 price,,,) = registry.latestRoundData(base, quote);
    require(price > 0, "invalid price");
    require(price > minAnswer && price < maxAnswer, "price outside valid range");

    // Extend the decimals to 1e18.
    return uint256(price) * 10 ** (18 - uint256(registry.decimals(base, quote)));
}
```

Please note, you will need to define `minAnswer` and `maxAnswer` according to your protocol's specifications.

Implementing this change will add an extra layer of security to your protocol, ensuring that price data used within the protocol falls within expected and reasonable boundaries, further reducing the risk of price manipulation and potential exploitation.

Note that only Chainlink oracles are implemented in protocol's oracle system, which exarcebates this issue, cause it's often assumed that by pooling data from several oracles, issues like the one outlined above can be circumvented. However, this may not always hold true, as other oracles may have their own vulnerabilities that can be exploited. For instance, if the Chainlink oracle is used in tandem with a UniswapV3Oracle that employs a long-term time-weighted average price (TWAP), the system could be susceptible to manipulation when the TWAP hovers near the minimum price during a downturn. In such a situation, the value of the third oracle in the system is essentially negated due to the agreement between the prices provided by the Chainlink and UniswapV3 oracles. Furthermore, if other secondary oracles like Band are utilized, malicious actors could potentially launch distributed denial-of-service (DDoS) attacks on relayers to disrupt price updates, which is why a revert in any of the case from my pov should be the go to

## [M-02] Possibility of Stale Data Usage in the PriceOracle Contract

### Summary

The PriceOracle contract retrieves prices for a variety of assets through the Chainlink oracle system. It uses the Chainlink feed registry for asset price data. However, the contract does not perform a check for the freshness of the data, which means it could potentially use stale data if Chainlink fails to provide an update in a timely manner.

### Vulnerability Detail

The contract's `getPrice()` and `_setAggregators()` methods both retrieve price data from Chainlink using the `registry.latestRoundData()` function. This function returns a tuple that includes the price and timestamp among other things, but the contract only uses the price and ignores the timestamp. As a result, the contract has no mechanism for verifying that the price data is recent.

This is particularly noticeable in the [getPriceFromChainlink()](https://github.com/sherlock-audit/2023-05-ironbank/blob/9ebf1702b2163b55479624794ab7999392367d2a/ib-v2/src/protocol/oracle/PriceOracle.sol#L36-L72) function:

```solidity
    function getPriceFromChainlink(address base, address quote) internal view returns (uint256) {
        (, int256 price,,,) = registry.latestRoundData(base, quote);
        require(price > 0, "invalid price");

        // Extend the decimals to 1e18.
        return uint256(price) * 10 ** (18 - uint256(registry.decimals(base, quote)));
    }
```

The above snippet shows the usage of `registry.latestRoundData()`, but the timestamp field from the function call is ignored, resulting in a lack of staleness check on the received price data.

### Impact

The absence of a staleness check means that potentially outdated prices could be used for financial calculations and transactions. This can lead to incorrect valuation of user positions, unfair trades, and in severe cases, improper liquidations. The effects can be detrimental to both individual users and the overall integrity of the platform.

### Recommendation

To mitigate this issue, we recommend adding a staleness check for the received price data. This could be implemented as a configurable threshold parameter that defines the maximum allowed age of the data. If the timestamp of the received data is older than the current time minus this threshold, the contract should reject the data as stale and potentially fetch new data or revert the transaction. This will ensure that only fresh and accurate price data is used for calculations and transactions.

## [M-03] Lack of Implementation of Fallback Oracles which Could Cause Protocol-wide Failure if the Chainlink Oracle Goes Down or Reverts Calls

### Summary

The dependency on a single oracle, particularly Chainlink, without fallback options presents a significant risk to the PriceOracle protocol. In some extreme cases, oracles can be taken offline or token prices can fall to zero. In these cases, liquidations, deposits and withdrawals or any execution that deals with querying of prices will be frozen (all calls will revert), If the Chainlink oracle for a particular asset is suspended, or the Chainlink multisigs decide to revert the call, all price queries will be inaccessible, potentially causing protocol-wide failure.

### Vulnerability Details

The PriceOracle contract relies heavily on Chainlink's oracle via the FeedRegistryInterface for pricing data. The contract fetches the asset prices in USD from Chainlink and normalizes the price by the asset's decimals. The contract also employs the Chainlink registry to fetch the latest round data for the specified base and quote assets.

In a case where the Chainlink oracle for a particular asset goes down, or the Chainlink multisigs decide to revert calls, all price queries for that asset will fail. This is because the getPrice and getPriceFromChainlink functions require the price returned from Chainlink's oracle to be greater than zero, or they revert.

```solidity
function getPriceFromChainlink(address base, address quote) internal view returns (uint256) {
    (, int256 price,,,) = registry.latestRoundData(base, quote);
    require(price > 0, "invalid price");

    // Extend the decimals to 1e18.
    return uint256(price) * 10 ** (18 - uint256(registry.decimals(base, quote)));
}
```

The absence of any fallback oracles in the protocol exacerbates this issue. If the Chainlink oracle fails or reverts calls, there are no secondary oracles in place that the protocol can default to. Consequently, this could cause all calls to query the price to be inaccessible, leading to a protocol-wide failure.

### Impact

The protocol's heavy reliance on Chainlink's oracle without a fallback oracle in place presents a significant risk. Should the Chainlink oracle go down or its calls be reverted, the protocol's core functionality could be severely disrupted, potentially leading to a complete system failure (in term of price queries). This could have serious implications for users and the protocol's overall stability and trustworthiness.

Affected code eference

- [PriceOracle.sol#L60-L72](https://github.com/sherlock-audit/2023-05-ironbank/blob/9ebf1702b2163b55479624794ab7999392367d2a/ib-v2/src/protocol/oracle/PriceOracle.sol#L60-L72)

### Recommendations

Though this introduces a little more complexity, consider implementing fallback oracles to supplement the primary Chainlink oracle. Then also implement all chainlink price queries in try/catch block, if the priary call reverts for any reason, the protocol can fall back on these secondary oracles, thereby ensuring continuous and uninterrupted price data.

## [L-01] Users Might Face Immediate Liquidation Post Loan Acquisition

### Summary

The `MarketConfigurator` contract in the protocol which seems to follow the concept of Compound V2's CF/LT, includes a function `configureMarketAsCollateral()`, which sets parameters related to collateral configuration. But the case is that the `collateralFactor` and `liquidationThreshold` parameters can be set to equal values. And when this happens, incentive for users to be immediately liquidated upon borrowing now exists, which seems impractical and potentially detrimental for a lending protocol.

### Vulnerability Detail

In the `configureMarketAsCollateral()` function, the `liquidationThreshold` is validated such that it should be greater than or equal to the `collateralFactor` (see the marked "@audit" comment). If both are set to equal values, it allows for immediate liquidation upon borrowing. This is because the collateral factor represents the proportion of borrowing power to the total collateral value, and the liquidation threshold denotes when a user's account can be liquidated. Thus, when they are equal, the user's account is liquidation eligible right after they borrow any amount.

This behavior seems questionable in the context of a lending platform. This means that as soon as a user borrows, they can be liquidated, which is not a healthy or practical operation for a lending protocol.

Take a look at [MarketConfigurator.sol#L114-L158](https://github.com/sherlock-audit/2023-05-ironbank/blob/9ebf1702b2163b55479624794ab7999392367d2a/ib-v2/src/protocol/pool/MarketConfigurator.sol#L114-L158), see the @audit tag for info.

```
    /**
     * @notice Configure a market as collateral.
     * @dev This function is used for the first time to configure a market as collateral.
     * @param market The market to be configured
     * @param collateralFactor The collateral factor of the market
     * @param liquidationThreshold The liquidation threshold of the market
     * @param liquidationBonus The liquidation bonus of the market
     */
    function configureMarketAsCollateral(
        address market,
        uint16 collateralFactor,
        uint16 liquidationThreshold,
        uint16 liquidationBonus
    ) external onlyOwner {
        DataTypes.MarketConfig memory config = getMarketConfiguration(market);
        require(config.isListed, "not listed");
        require(
            config.collateralFactor == 0 && config.liquidationThreshold == 0 && config.liquidationBonus == 0,
            "already configured"
        );
        require(collateralFactor > 0 && collateralFactor <= MAX_COLLATERAL_FACTOR, "invalid collateral factor");
        require(
            liquidationThreshold > 0 && liquidationThreshold <= MAX_LIQUIDATION_THRESHOLD
                && liquidationThreshold >= collateralFactor,
            "invalid liquidation threshold"
        );
       //  @audit the above suggests that `liquidationThreshold = collateralFactor` is allowed which doesn't make sense as users can immediately get liquidated once they take out a loan which is not practical behaiviour
        require(
            liquidationBonus > MIN_LIQUIDATION_BONUS && liquidationBonus <= MAX_LIQUIDATION_BONUS,
            "invalid liquidation bonus"
        );
        require(
            uint256(liquidationThreshold) * uint256(liquidationBonus) / FACTOR_SCALE
                <= MAX_LIQUIDATION_THRESHOLD_X_BONUS,
            "liquidation threshold * liquidation bonus larger than 100%"
        );

        config.collateralFactor = collateralFactor;
        config.liquidationThreshold = liquidationThreshold;
        config.liquidationBonus = liquidationBonus;
        ironBank.setMarketConfiguration(market, config);

        emit MarketCollateralFactorSet(market, collateralFactor);
        emit MarketLiquidationThresholdSet(market, liquidationThreshold);
        emit MarketLiquidationBonusSet(market, liquidationBonus);
    }
```

### Impact

This can result in discouraging users from participating in the protocol as it leaves no room for price fluctuation and could potentially liquidate user positions immediately after a borrow operation. This might affect the overall health and adoption of the lending platform.

### Recommendation

It would be advisable to consider modifying the function to disallow setting the `liquidationThreshold` to be equal to the `collateralFactor`. The function could be revised to ensure that the `liquidationThreshold` is strictly greater than the `collateralFactor`, which would prevent the potential issue of immediate liquidation upon borrowing. The adjustment would be a minor change in the `require` statement that checks these parameters:

```solidity
require(
    liquidationThreshold > 0 && liquidationThreshold <= MAX_LIQUIDATION_THRESHOLD
        && liquidationThreshold > collateralFactor,
    "invalid liquidation threshold"
);
```

In this way, a user's account becomes eligible for liquidation only when the borrowed amount exceeds the borrowing power determined by the `collateralFactor`. It would be a healthier and more practical approach for a lending protocol, and it would give users some leeway to manage their positions in response to price fluctuations.

## [L-02] Unnecessary Precision Loss in `TripleSlopeRateModel`

### Summary

This report identifies potential precision loss issues in the `TripleSlopeRateModel` contract. The division operations occurring before multiplication in certain mathematical expressions in the `getBorrowRate()` function could lead to unnecessary precision loss.

### Vulnerability Detail

The `getBorrowRate()` function calculates the borrowing rate based on the market's cash and borrow amount. It employs a triple slope model to implement three different rates based on the level of utilization.

Take a look at the [getBorrowRate() function](https://github.com/sherlock-audit/2023-05-ironbank/blob/9ebf1702b2163b55479624794ab7999392367d2a/ib-v2/src/protocol/pool/interest-rate-model/TripleSlopeRateModel.sol#L44-L64C6)

```solidity
    /**
     * @notice Get the borrow rate per second.
     * @param cash The cash in the market
     * @param borrow The borrow in the market
     * @return The borrow rate per second
     */
    function getBorrowRate(uint256 cash, uint256 borrow) public view returns (uint256) {
        uint256 utilization = getUtilization(cash, borrow);
        if (utilization <= kink1) {
            // base + utilization * slope1
            return baseBorrowPerSecond + (utilization * borrowPerSecond1) / 1e18;
        } else if (utilization <= kink2) {
            // base + kink1 * slope1 + (utilization - kink2) * slope2
            return baseBorrowPerSecond + (kink1 * borrowPerSecond1) / 1e18
                + ((utilization - kink1) * borrowPerSecond2) / 1e18;
        } else {
            // base + kink1 * slope1 + (kink2 - kink2) * slope2 + (utilization - kink2) * slope3
            return baseBorrowPerSecond + (kink1 * borrowPerSecond1) / 1e18 + ((kink2 - kink1) * borrowPerSecond2) / 1e18
                + (utilization - kink2) * borrowPerSecond3 / 1e18;
        }
    }
```

As seen the function involves divisions before multiplications in several expressions which could potentially lead to precision loss. Here are the affected code snippets:

```solidity
// First Instance
return baseBorrowPerSecond + (kink1 * borrowPerSecond1) / 1e18 + ((utilization - kink1) * borrowPerSecond2) / 1e18;

// Second Instance
return baseBorrowPerSecond + (kink1 * borrowPerSecond1) / 1e18 + ((kink2 - kink1) * borrowPerSecond2) / 1e18 + (utilization - kink2) * borrowPerSecond3 / 1e18;
```

The division operations that occur before multiplication could truncate the values, leading to unnecessary precision loss. For instance, in the first instance, `(kink1 * borrowPerSecond1) / 1e18` will truncate the product of `kink1` and `borrowPerSecond1` before adding it to `baseBorrowPerSecond`, potentially causing a significant precision loss. A similar issue happens in the second instance.

### Impact

Precision loss in the calculation of borrowing rates could result in inaccurate rates, which may impact the overall functionality of the contract. Since this is used in a financial setting, any little inaccuracy could lead to financial imbalances, affecting users' transactions and the contract's overall integrity.

### Recommendation

To mitigate these precision loss issues, the arithmetic operations in the expressions should be rearranged such that multiplication operations are performed before divisions. For instance, the operations for `getBorrowRate()` can be improved to:

```diff
 function getBorrowRate(uint256 cash, uint256 borrow) public view returns (uint256) {
        uint256 utilization = getUtilization(cash, borrow);
        if (utilization <= kink1) {
            // base + utilization * slope1
            return baseBorrowPerSecond + (utilization * borrowPerSecond1) / 1e18;
        } else if (utilization <= kink2) {
            // base + kink1 * slope1 + (utilization - kink2) * slope2
-            return baseBorrowPerSecond + (kink1 * borrowPerSecond1) / 1e18
-                + ((utilization - kink1) * borrowPerSecond2) / 1e18;
+ return baseBorrowPerSecond + ((kink1 * borrowPerSecond1) + ((utilization - kink1) * borrowPerSecond2)) / 1e18;
        } else {
            // base + kink1 * slope1 + (kink2 - kink2) * slope2 + (utilization - kink2) * slope3
-            return baseBorrowPerSecond + (kink1 * borrowPerSecond1) / 1e18 + ((kink2 - kink1) * borrowPerSecond2) / 1e18
-                + (utilization - kink2) * borrowPerSecond3 / 1e18;
+ return baseBorrowPerSecond + ((kink1 * borrowPerSecond1) + ((kink2 - kink1) * borrowPerSecond2) + (utilization - kink2) * borrowPerSecond3 ) / 1e18;
        }
    }
```

Such changes will help to maintain the accuracy of the calculations in the contract and minimize potential financial risks
