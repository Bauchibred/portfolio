# Introduction

A time-boxed security review of the **Venus** protocol which was was done by **bauchibred**, with a focus on the security aspects of the application's smart contracts implementation.

# Table of Contents

- [Disclaimer](#disclaimer)
- [Severity classification](#Severity-classification)
- [Scope](#scope)
- [About Bauchibred](#About-Bauchibred)
- [About Venus](#About-Venus)
- [Summary](#summary)
- [Findings](#findings)

# Disclaimer

A smart contract security review can never verify the complete absence of vulnerabilities. This is a time, resource and expertise bound effort where I try to find as many vulnerabilities as possible. I can not guarantee 100% security after the review or even if the review will find any problems with your smart contracts. Subsequent security reviews, bug bounty programs and on-chain monitoring are strongly recommended.

Additionally, this report was prepared for [this competition](https://code4rena.com/contests/2023-05-venus-protocol-isolated-pools#top). The competition spanned **7** days, from **_May 8, 2023_** to **_May 15, 2023_**, and was hosted on [Code4rena](https://code4rena.com/). Given the public nature of the contest, an appointed judge, aided by a panel of other judges and security researchers including myself, had the discretion to determine the final severity of each finding. Due to the inherently subjective nature of such judgments, the final severity assigned might differ from my own classifications.In this context, `[H-01]` was originally submitted as a medium severity finding which was upgraded by the judge and both `[M-01]` & `[M-02]` were downgraded to grade-A QA reports, please refer to [this link]() if you need more clarification.

# Severity classification

| Severity                                                                                                                                                                                            | Description                                                                                                                                                                                                                                    |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ![](https://camo.githubusercontent.com/a0b140cbe7b198d62804d91996e3c09c6803cfbd484c61974af51892481f840e/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d437269746963616c2d643130623062) | A **directly** exploitable security vulnerability that leads to stolen/lost/locked/compromised assets or catastrophic denial of service.                                                                                                       |
| ![](https://camo.githubusercontent.com/77229c2f86cd6aaaaaeb3879eac44712b0e095147962e9058e455d0094f500d3/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d486967682d726564)               | A security vulnerability or bug that can affect the correct functioning of the system, lead to incorrect states or denial of service. It may not be directly exploitable or may require certain, external conditions in order to be exploited. |
| ![](https://camo.githubusercontent.com/d2cf6c2836b2143aeeb65c08b9c5aa1eb34a6fb8ad6fc55ba4345c467b64378a/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d656469756d2d6f72616e6765)     | Assets not at direct risk, but the function of the protocol or its availability could be impacted, or leak value with a hypothetical attack path with stated assumptions, but external requirements.                                           |
| ![](https://camo.githubusercontent.com/d42acfb8cb8228c156f34cb1fab83f431bf1fbebc102d922950f945b45e05587/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d696e6f722d79656c6c6f77)       | A violation of common best practices or incorrect usage of primitives, which may not currently have a major impact on security, but may do so in the future or introduce inefficiencies.                                                       |

# Scope

The review focused on the commit [hash](https://github.com/code-423n4/2023-05-venus/commit/723001cf7bc0f37aba26fb385ec1a60135f24fe3) and primarily covered the following contracts:

| File                                                                                                                                                                 | SLOC |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---- |
| [Comptroller.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Comptroller.sol)                                                                   | 728  |
| [VToken.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol)                                                                             | 655  |
| [Lens/PoolLens.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Lens/PoolLens.sol)                                                               | 378  |
| [Shortfall/Shortfall.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/Shortfall.sol)                                                   | 289  |
| [Rewards/RewardsDistributor.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Rewards/RewardsDistributor.sol)                                     | 266  |
| [Pool/PoolRegistry.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistry.sol)                                                       | 251  |
| [RiskFund/RiskFund.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/RiskFund.sol)                                                       | 166  |
| [VTokenInterfaces.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VTokenInterfaces.sol)                                                         | 141  |
| [ExponentialNoError.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ExponentialNoError.sol)                                                     | 102  |
| [BaseJumpRateModelV2.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/BaseJumpRateModelV2.sol)                                                   | 93   |
| [ComptrollerInterface.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerInterface.sol)                                                 | 62   |
| [ComptrollerStorage.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ComptrollerStorage.sol)                                                     | 58   |
| [RiskFund/ProtocolShareReserve.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ProtocolShareReserve.sol)                               | 57   |
| [Factories/VTokenProxyFactory.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Factories/VTokenProxyFactory.sol)                                 | 43   |
| [WhitePaperInterestRateModel.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/WhitePaperInterestRateModel.sol)                                   | 43   |
| [ErrorReporter.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/ErrorReporter.sol)                                                               | 42   |
| [RiskFund/ReserveHelpers.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/ReserveHelpers.sol)                                           | 37   |
| [JumpRateModelV2.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/JumpRateModelV2.sol)                                                           | 21   |
| [Factories/JumpRateModelFactory.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Factories/JumpRateModelFactory.sol)                             | 20   |
| [Pool/PoolRegistryInterface.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Pool/PoolRegistryInterface.sol)                                     | 20   |
| [MaxLoopsLimitHelper.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/MaxLoopsLimitHelper.sol)                                                   | 18   |
| [InterestRateModel.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/InterestRateModel.sol)                                                       | 15   |
| [RiskFund/IRiskFund.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/RiskFund/IRiskFund.sol)                                                     | 11   |
| [IPancakeswapV2Router.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/IPancakeswapV2Router.sol)                                                 | 10   |
| [Factories/WhitePaperInterestRateModelFactory.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Factories/WhitePaperInterestRateModelFactory.sol) | 8    |
| [Proxy/UpgradeableBeacon.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Proxy/UpgradeableBeacon.sol)                                           | 7    |
| [RiskFund/IProtocolShareReserve.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Proxy/UpgradeableBeacon.sol)                                    | 4    |
| [Shortfall/IShortfall.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/Shortfall/IShortfall.sol)                                                 | 4    |

~3069 NSLOC in 16 contracts

NSLOC stands for 'Normalized Source Code', which is one of the custom measurements security researchers use when evaluating the complexity of a codebase.

The steps below are taken to get the NSLOC count of a file

1.  For all functions, reduce any multiline function declarations to a single line.
2.  Remove all comments
3.  Remove all empty lines
4.  Count the remaining lines

# About Bauchibred

**Bauchibred** is an independent smart contract security researcher. Having found numerous security vulnerabilities in various protocols, he does his best to contribute to the blockchain ecosystem and its protocols by putting time and effort into security research & reviews. Check his previous work [here](https://github.com/bauchibred/audits) or reach out on Twitter [@bauchibred](https://twitter.com/bauchibred).

# About Venus

[Venus](https://app.venus.io) is a DeFi money market protocol on the BNB Chain that enables P2P lending similar to traditional bank services. Users can earn interest by lending assets and borrow assets without liquidating their holdings. However, the Venus Core Pool presents challenges such as exposure to high risks from asset volatility and limited visibility into market stability. **Venus Isolated Pools** address these issues by offering independent asset collections with tailored risk management settings, allowing users to better manage risks and maximize their returns. This design also lessens the protocol's vulnerability to a single asset's performance.

- Website: https://app.venus.io/

# Summary

This report contains **1 high** severity issue, **2 medium** severity issues and **16 minor** issues found by **Bauchibred** during the course of the time-boxed audit

| #       | Title                                                                                                                                                      | Severity                                         | Status                                                  |
| ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ | ------------------------------------------------------- |
| [H-01]  | WhitePaperInterestRateModel.sol: Inaccurate assumption of the interest rate model since blocksPerYear is wrongly set to 2102400                            | ![](https://img.shields.io/badge/-High-red)      | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-01]  | Comptroller.sol: `setCollateralFactor()` should be integrated with a timelock in regards to `LiquidationThresholdMantissa` and `CollateralFactorMantissa`  | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-02]  | maxLoopsLimit could end up being disadvantageous to protocol and lead to a DOS                                                                             | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-01]] | Comptroller.sol: collateral factor and liquidation threshold shouldn't be allowed to be set equal in `setCollateralFactor()`                               | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-02]  | Comptroller.sol: Incomplete input validation could cause all liquidation attempts to fail                                                                  | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-03]  | RewardsDistributor.sol: Inconsistency in sister function executions                                                                                        | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-04]  | utilizationRate() could DOS in both WhitePaperInterestRateModel.sol or BaseJumpRateModelV2.sol                                                             | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-05]  | VToken.sol: Wrong steps in the execution of the `_redeemFresh()` functions                                                                                 | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [L-06]  | VToken.sol: The error being `NO_ERROR ` in most cases is completely a downside of protocol as users can't know the reasons to why their transaction failed | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-07]  | RiskFund.sol: Use of block.timestamp for `_swapAsset()` should be reconsidered                                                                             | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-08]  | Confusing names of scales in ExponentialNoError.sol                                                                                                        | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [L-09]  | Missing Zero address checks                                                                                                                                | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-10]  | Consider isContract checks in constructors                                                                                                                 | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [L-11]  | Renouncing ownership could break some functionalities                                                                                                      | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/--Acknowledged-black)  |
| [NC-01] | Comptroller.sol: redundant step in preMintHook()                                                                                                           | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [NC-02] | Inconsistent storage gaps                                                                                                                                  | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge-Acknowledged-black)    |
| [NC-03] | Some typos are in the docs and some were missed by bot                                                                                                     | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [NC-04] | Some events are missing complete indexed fields                                                                                                            | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [NC-05] | Comptroller.sol: suggestion to rename minLiquidatableCollateral to maxLiquidatableCollateral                                                               | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |

# Findings

## [H-01] WhitePaperInterestRateModel.sol: Inaccurate assumption of the interest rate model since blocksPerYear is wrongly set to 2102400

### Impact

The base interest rate when utilization rate is 0 is going to be 5 times higher than normal, also the slope of the interest rate is going to get highly flawed as the multiplier of the utilization rate that gives the slope is also going to be 5 times higher than is supposed to be.

This is all due to the fact that `blocksPerYear` is wrongly set to 2102400 (1/5 years) in [WhitePaperInterestRateModel.sol:](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/WhitePaperInterestRateModel.sol) instead of 10512000 as is been set in [BaseJumpRateModelV2.sol](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/BaseJumpRateModelV2.sol#L23) which means that both `baseRatePerBlock` and `multiplierPerBlock` are wrongly calculated since they are directly influenced by the `blocksPerYear`, see WhitePaperInterestRateModel L37-38:

```
        baseRatePerBlock = baseRatePerYear / blocksPerYear;
        multiplierPerBlock = multiplierPerYear / blocksPerYear
```

### Proof of Concept

Take a look at [WhitePaperInterestRateModel.sol#L14-17:](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/WhitePaperInterestRateModel.sol#L14-L17)

```
    uint256 public constant blocksPerYear = 2102400;
```

As seen this is an obvious mistake as `blocksPerYear` should instead be `10512000`, making [3 seconds per block as known for BSC if network is completely stable](https://nodereal.io/blog/en/bnb-smart-chain-performance-anatomy-series-chapter-i-the-critical-3-seconds/)
NB: the `blocksPerYear` value is correctly set in [BaseJumpRateModelV2.so](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/BaseJumpRateModelV2.sol#L23)

```
    uint256 public constant blocksPerYear = 10512000;
```

This obviously means that there is a costly mistake in the WhitePaperInterestRateModel.sol contract as a wrong assumption is made here that each block lasts 15 seconds instead of 3 seconds

Below is the code snippet of when the values of baseRatePerBlock and multiplierPerBlock are being set with the help of blocksPerYear
[WhitePaperInterestRateModel.sol#constructor:](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/WhitePaperInterestRateModel.sol#L36-L41)

```

    constructor(uint256 baseRatePerYear, uint256 multiplierPerYear) {
        baseRatePerBlock = baseRatePerYear / blocksPerYear;
        multiplierPerBlock = multiplierPerYear / blocksPerYear;

        emit NewInterestParams(baseRatePerBlock, multiplierPerBlock);
    }

```

Whereas look at [BaseJumpRateModelV2.sol L23:](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/BaseJumpRateModelV2.sol#L23)

```
uint256 public constant blocksPerYear = 10512000;
```

I believe the latter should be the right value

I also had a discussion with the Venus team (chechu#0799 in this case) to clarify some details about this issue, and the finding got confirmed, see discussion below:

```
Q: There is an inconsistency in setting the blocksPerYear variable within contracts in scope, i.e WhitePaperInterestRateModel (2102400) whereas for BaseJumpRateModelV2.sol it's 10512000, which I believe should be the correct value

A: You are right, congratulations and thanks for the finding 🥳
```

### Recommendation

This error is probably due to a copy and paste of the [Compound protocol's WhitePaperInterestRateModel Contract](https://github.com/compound-finance/compound-protocol/blob/master/contracts/WhitePaperInterestRateModel.sol) and I believe the fix is as simple as setting the `blocksPerYear` value to `10512000` in both contracts.

## [M-01] Comptroller.sol: `setCollateralFactor()` should be integrated with a timelock in regards to `LiquidationThresholdMantissa` and `CollateralFactorMantissa`

### Impact

`LiquidationThresholdMantissa(LT for short)` and `CollateralFactorMantissa(CF for short)` could be set in a disadvantageous manner to borrower.
Look at this example: LT was fomerly 0.5, when CF was say 0.9, and then LT gets set to 0.8 where CF remains at 0.9, this increases borrower's chance of getting liquidated by a whopping 60%, and if borrower is unlucky and took a loan of an asset that's dropping fast in price, borrower could get liquidated in no time.
I think this report is a two-in-one.

1. No timelock applied to this, this is important so that users can get notified of a change in either of LT or CF, and prepare not to get liquidated, before said change checks in since they should have enough time to mitigate.
2. Arguably the fact that LT or CF could be set to any value, rather than higher than their maxes or "LT < CF" could be very costly not just for borrowers but even protocol.

NB: This is just one attack window I wrote about, other ways to affect normal functionality does exist, for example ( leaving LT's value as it is and instead reducing CF... etc)

### Proof of Concept

I asked the Venus team (chechu
#0799 in this case) a question about the setCollateralFactor() function, see discussion below:

```
Q: Can the comptroller.sol#setCollateralFactor() function be set for an already set market?
I mean is it possible for this function to be called more than once for a particular market and used to update the threshold & CF of the said market?

A: yes, it can be invoked several times, with different values, or the same values
```

Also I believe **Impact** though been short has provided enough POC, but below is the codeblock of the [setCloseFactor() function](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#LL712C1-L770C6) just to use to cross reference:

```

    /**
     * @notice Sets the collateralFactor for a market
     * @dev This function is restricted by the AccessControlManager
     * @param vToken The market to set the factor on
     * @param newCollateralFactorMantissa The new collateral factor, scaled by 1e18
     * @param newLiquidationThresholdMantissa The new liquidation threshold, scaled by 1e18
     * @custom:event Emits NewCollateralFactor when collateral factor is updated
     *    and NewLiquidationThreshold when liquidation threshold is updated
     * @custom:error MarketNotListed error is thrown when the market is not listed
     * @custom:error InvalidCollateralFactor error is thrown when collateral factor is too high
     * @custom:error InvalidLiquidationThreshold error is thrown when liquidation threshold is lower than collateral factor
     * @custom:error PriceError is thrown when the oracle returns an invalid price for the asset
     * @custom:access Controlled by AccessControlManager
     */
    function setCollateralFactor(
        VToken vToken,
        uint256 newCollateralFactorMantissa,
        uint256 newLiquidationThresholdMantissa
    ) external {
        _checkAccessAllowed("setCollateralFactor(address,uint256,uint256)");

        // Verify market is listed
        Market storage market = markets[address(vToken)];
        if (!market.isListed) {
            revert MarketNotListed(address(vToken));
        }

        // Check collateral factor <= 0.9
        if (newCollateralFactorMantissa > collateralFactorMaxMantissa) {
            revert InvalidCollateralFactor();
        }

        // Ensure that liquidation threshold <= 1
        if (newLiquidationThresholdMantissa > mantissaOne) {
            revert InvalidLiquidationThreshold();
        }

        // Ensure that liquidation threshold >= CF
        if (newLiquidationThresholdMantissa < newCollateralFactorMantissa) {
            revert InvalidLiquidationThreshold();
        }

        // If collateral factor != 0, fail if price == 0
        if (newCollateralFactorMantissa != 0 && oracle.getUnderlyingPrice(address(vToken)) == 0) {
            revert PriceError(address(vToken));
        }

        uint256 oldCollateralFactorMantissa = market.collateralFactorMantissa;
        if (newCollateralFactorMantissa != oldCollateralFactorMantissa) {
            market.collateralFactorMantissa = newCollateralFactorMantissa;
            emit NewCollateralFactor(vToken, oldCollateralFactorMantissa, newCollateralFactorMantissa);
        }

        uint256 oldLiquidationThresholdMantissa = market.liquidationThresholdMantissa;
        if (newLiquidationThresholdMantissa != oldLiquidationThresholdMantissa) {
            market.liquidationThresholdMantissa = newLiquidationThresholdMantissa;
            emit NewLiquidationThreshold(vToken, oldLiquidationThresholdMantissa, newLiquidationThresholdMantissa);
        }
    }

```

As seen values can be changed whenever and the effects come in place immediately which has it's downside on the receiving end of changes

### Recommendation

At the very least a timelock should be implemented even if it's going to be a very short time frame.

## [M-02] maxLoopsLimit could end up being disadvantageous to protocol and lead to a DOS

### Impact

The first check in `_setMaxLoopsLimit()` requires that the new limit must be higher than the older limit `maxLoopsLimit`
L26: `require(limit > maxLoopsLimit, "Comptroller: Invalid maxLoopsLimit");`

There are two scenarios where I think this could break it's expected functionality:

1. If the cost of executing a function increases with updates or for whatever reason and the current `maxLoopsLimit` is now too large and could cause a DOS if a loop of this iteration is provided, there is no possibility to reduce the maxLoopsLimit since the check only required higher limits to be set
2. Though this case is less likely to happen it's still a possibility, the value of maxLoopsLimit gets mistakenly set to an outrageously high value, there is no provision of resetting this to a normal value, as a normal value in this case would be a lesser value that maxLoopsLimit, and an attempt of setting this would cause a revert of "Comptroller: Invalid maxLoopsLimit"

Note that this affects a lot of functionalities of contracts in scope as `_ensureMaxLoops()` is heavily implemented within contracts, for example in both of Riskfunc.sol and Comptroler.sol this function is used and all what this functiion does is to ensure that length of the loops provided is not more than `maxLoopsLimit` causing a revert if the reverse is the case.

```if (len > maxLoopsLimit) {
            revert MaxLoopsLimitExceeded(maxLoopsLimit, len);
```

Now this means that if any of the two cases mentioned above where to occur the `maxLoopsLimit` is already faulty, and the `_ensureMaxLoops()` function should revert for a lesser value than `maxLoopsLimit` but it won't due to the condition, essentially breaking expected functionalities of protocol.

### Proof of Concept

`
Here is an external onlyOwner's call to set the max loops limit in Riskfund.sol
[RiskFund.setMaxLoopsLimit()](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#L201-L207)

```
    /**
     * @notice Set the limit for the loops can iterate to avoid the DOS
     * @param limit Limit for the max loops can execute at a time
     */
    function setMaxLoopsLimit(uint256 limit) external onlyOwner {
        _setMaxLoopsLimit(limit);
    }
```

Now check out the[ \_setMaxLoopsLimit() function:](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/MaxLoopsLimitHelper.sol#L25-L32)

```
    function _setMaxLoopsLimit(uint256 limit) internal {
        require(limit > maxLoopsLimit, "Comptroller: Invalid maxLoopsLimit");

        uint256 oldMaxLoopsLimit = maxLoopsLimit;
        maxLoopsLimit = limit;

        emit MaxLoopsLimitUpdated(oldMaxLoopsLimit, maxLoopsLimit);
    }
```

Also check out the[ \_ensureMaxLoopst() function:](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/MaxLoopsLimitHelper.sol#L39-L43)

```
    function _ensureMaxLoops(uint256 len) internal view {
        if (len > maxLoopsLimit) {
            revert MaxLoopsLimitExceeded(maxLoopsLimit, len);
        }
    }
}
```

Also you can `grep _ensureMaxLoops() ` within contracts in scoope to see how widely its been implemented don't want to include all cases in report as it would make it unnecesarily bogus

### Recommendation

Simply put i think the `_setMaxLoopsLimit() `function should allow even lesser values of limit to be set. Ofcourse a check that `limit != 0` should be implemented as it's absurd if 0 loops are allowed))

## [L-01] Comptroller.sol: collateral factor and liquidation threshold shouldn't be allowed to be set equal in `setCollateralFactor()`

### Impact

If/When collateral factor and liquidation threshold are set equal in the `setCollateralFactor() ` function the borrow can immediately be liquidated as soon as it's been given out which just doesn't seem as a right way to operate a lending proposal, this is allowed cause the check of `newLiquidationThresholdMantissa` against `newCollateralFactorMantissa` is strict and my argument is that this check should instead be inclusive, see POC

Comptroller.sol#L750-752:

```
        if (newLiquidationThresholdMantissa < newCollateralFactorMantissa) {
                        revert InvalidLiquidationThreshold();
        }
```

#### Proof of Concept

I asked the Venus team (chechu
#0799 in this case) a question about the setCollateralFactor() function, see discussion below:

```
Q: Can the comptroller.sol#setCollateralFactor() function be set for an already set market?
I mean is it possible for this function to be called more than once for a particular market and used to update the threshold & CF of the said market?

A: yes, it can be invoked several times, with different values, or the same values
```

The above clarifies that both `newCollateralFactorMantissa` and `newLiquidationThresholdMantissafunction` could be changed for any market whenever and any value could be passed, though a few checks within the codeblocks ensure that the values are not set outrageously set for example the function's execution ensures that `newLiquidationThresholdMantissa` is never more than `mantissaOne` at L744-747:

```
        // Ensure that liquidation threshold <= 1
        if (newLiquidationThresholdMantissa > mantissaOne) {
            revert InvalidLiquidationThreshold();
        }
```

Other checks too exist one of which is that
`newLiquidationThresholdMantissa` is never less than `newCollateralFactorMantissa`
see L750-752:

```

        if (newLiquidationThresholdMantissa < newCollateralFactorMantissa) {
                        revert InvalidLiquidationThreshold();
        }

```

Now at L749, the commented description of this conditional check states that:

```

        // Ensure that liquidation threshold >= CF

```

This essentially means that the setting of `liquidation threshold = CF` is intended behavior, but I argue that this shouldn't be allowed as there is no point giving out a loan if it can be immediately liquidated.

Take a look at the [setCloseFactor() function fully:](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#LL712C1-L770C6)

```

    /**
     * @notice Sets the collateralFactor for a market
     * @dev This function is restricted by the AccessControlManager
     * @param vToken The market to set the factor on
     * @param newCollateralFactorMantissa The new collateral factor, scaled by 1e18
     * @param newLiquidationThresholdMantissa The new liquidation threshold, scaled by 1e18
     * @custom:event Emits NewCollateralFactor when collateral factor is updated
     *    and NewLiquidationThreshold when liquidation threshold is updated
     * @custom:error MarketNotListed error is thrown when the market is not listed
     * @custom:error InvalidCollateralFactor error is thrown when collateral factor is too high
     * @custom:error InvalidLiquidationThreshold error is thrown when liquidation threshold is lower than collateral factor
     * @custom:error PriceError is thrown when the oracle returns an invalid price for the asset
     * @custom:access Controlled by AccessControlManager
     */
    function setCollateralFactor(
        VToken vToken,
        uint256 newCollateralFactorMantissa,
        uint256 newLiquidationThresholdMantissa
    ) external {
        _checkAccessAllowed("setCollateralFactor(address,uint256,uint256)");

        // Verify market is listed
        Market storage market = markets[address(vToken)];
        if (!market.isListed) {
            revert MarketNotListed(address(vToken));
        }

        // Check collateral factor <= 0.9
        if (newCollateralFactorMantissa > collateralFactorMaxMantissa) {
            revert InvalidCollateralFactor();
        }

        // Ensure that liquidation threshold <= 1
        if (newLiquidationThresholdMantissa > mantissaOne) {
            revert InvalidLiquidationThreshold();
        }

        // Ensure that liquidation threshold >= CF
        if (newLiquidationThresholdMantissa < newCollateralFactorMantissa) {
            revert InvalidLiquidationThreshold();
        }

        // If collateral factor != 0, fail if price == 0
        if (newCollateralFactorMantissa != 0 && oracle.getUnderlyingPrice(address(vToken)) == 0) {
            revert PriceError(address(vToken));
        }

        uint256 oldCollateralFactorMantissa = market.collateralFactorMantissa;
        if (newCollateralFactorMantissa != oldCollateralFactorMantissa) {
            market.collateralFactorMantissa = newCollateralFactorMantissa;
            emit NewCollateralFactor(vToken, oldCollateralFactorMantissa, newCollateralFactorMantissa);
        }

        uint256 oldLiquidationThresholdMantissa = market.liquidationThresholdMantissa;
        if (newLiquidationThresholdMantissa != oldLiquidationThresholdMantissa) {
            market.liquidationThresholdMantissa = newLiquidationThresholdMantissa;
            emit NewLiquidationThreshold(vToken, oldLiquidationThresholdMantissa, newLiquidationThresholdMantissa);
        }
    }

```

### Recommendation

Sponsors should reconsider this, but in short I think the check should be inclusive, i.e the conditional execution should instead be changed to the below
L740-752:

```
// Ensure that liquidation threshold > CF
        if (newLiquidationThresholdMantissa <= newCollateralFactorMantissa) {
                        revert InvalidLiquidationThreshold();
        }

```

## [L-02] Comptroller.sol: Incomplete input validation could cause all liquidation attempts to fail

### Impact

All liquidation attempts would fail if `minLiquidatableCollateral` gets set to an outrageous value (say `type(uint256).max`, just for explanation's sake), also important to note that setting `minLiquidatableCollateral` to 0 means that all regular liquidations would pass which might not be ideal for protocol.

#### Proof of Concept

Take a look at the [setMinLiquidatableCollateral() function:](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L904-L918)

```
    function setMinLiquidatableCollateral(uint256 newMinLiquidatableCollateral) external {
        _checkAccessAllowed("setMinLiquidatableCollateral(uint256)");

        uint256 oldMinLiquidatableCollateral = minLiquidatableCollateral;
        minLiquidatableCollateral = newMinLiquidatableCollateral;
        emit NewMinLiquidatableCollateral(oldMinLiquidatableCollateral, newMinLiquidatableCollateral);
    }
```

As seen no sanity checks are implemented in protocol's execution of this function, meaning that `minLiquidatableCollateral` could be set to `0 or type(uint256).max`... **Impact**'s done a good job in explaining how this is faulty

### Recommendation

Though this seems like a little more complexity for protocol, sanity checks should be implemented for this function.

## [L-03] RewardsDistributor.sol: Inconsistency in sister function executions

### Vulnerability Details

If you check the code snippet from below you can see that both are sister functions, but in `_distributeBorrowerRewardToken()`, borrowerAmount in this case is been checked to ensure that its not 0 before this calculation goes ahead, where as in the `_distributeSupplierRewardToken()` function no `supplierTokens != 0` checks.

Take a look at the [\_distributeSupplierRewardToken() function](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L340-L367)

```

function \_distributeSupplierRewardToken((address vToken, address supplier) internal {
RewardToken storage supplyState = rewardTokenSupplyState[vToken];
uint256 supplyIndex = supplyState.index;
uint256 supplierIndex = rewardTokenSupplierIndex[vToken][supplier];

        // Update supplier's index to the current index since we are distributing accrued REWARD TOKEN
        rewardTokenSupplierIndex[vToken][supplier] = supplyIndex;

        if (supplierIndex == 0 && supplyIndex >= rewardTokenInitialIndex) {
            // Covers the case where users supplied tokens before the market's supply state index was set.
            // Rewards the user with REWARD TOKEN accrued from the start of when supplier rewards were first
            // set for the market.
            supplierIndex = rewardTokenInitialIndex;
        }

        // Calculate change in the cumulative sum of the REWARD TOKEN per vToken accrued
        Double memory deltaIndex = Double({ mantissa: sub_(supplyIndex, supplierIndex) });

        uint256 supplierTokens = VToken(vToken).balanceOf(supplier);

        // Calculate REWARD TOKEN accrued: vTokenAmount * accruedPerVToken
        //@audit no supplierTokens != 0 checks
        uint256 supplierDelta = mul_(supplierTokens, deltaIndex);

        uint256 supplierAccrued = add_(rewardTokenAccrued[supplier], supplierDelta);
        rewardTokenAccrued[supplier] = supplierAccrued;

        emit DistributedSupplierRewardToken(VToken(vToken), supplier, supplierDelta, supplierAccrued, supplyIndex);
    }

```

Now take a look at the [\_distributeBorrowerRewardToken() function](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L340-L367)

```

    function _distributeBorrowerRewardToken(
        address vToken,
        address borrower,
        Exp memory marketBorrowIndex
    ) internal {
        RewardToken storage borrowState = rewardTokenBorrowState[vToken];
        uint256 borrowIndex = borrowState.index;
        uint256 borrowerIndex = rewardTokenBorrowerIndex[vToken][borrower];

        // Update borrowers's index to the current index since we are distributing accrued REWARD TOKEN
        rewardTokenBorrowerIndex[vToken][borrower] = borrowIndex;

        if (borrowerIndex == 0 && borrowIndex >= rewardTokenInitialIndex) {
            // Covers the case where users borrowed tokens before the market's borrow state index was set.
            // Rewards the user with REWARD TOKEN accrued from the start of when borrower rewards were first
            // set for the market.
            borrowerIndex = rewardTokenInitialIndex;
        }

        // Calculate change in the cumulative sum of the REWARD TOKEN per borrowed unit accrued
        Double memory deltaIndex = Double({ mantissa: sub_(borrowIndex, borrowerIndex) });

        uint256 borrowerAmount = div_(VToken(vToken).borrowBalanceStored(borrower), marketBorrowIndex);

        // Calculate REWARD TOKEN accrued: vTokenAmount * accruedPerBorrowedUnit
        //@audit
        if (borrowerAmount != 0) {
            uint256 borrowerDelta = mul_(borrowerAmount, deltaIndex);

            uint256 borrowerAccrued = add_(rewardTokenAccrued[borrower], borrowerDelta);
            rewardTokenAccrued[borrower] = borrowerAccrued;

            emit DistributedBorrowerRewardToken(VToken(vToken), borrower, borrowerDelta, borrowerAccrued, borrowIndex);
        }
    }

```

### Recommendation

The same checks should be used here though i can't really think of an attack window i think execution should be constant for both sister functions

## [L-04] utilizationRate() could DOS in both WhitePaperInterestRateModel.sol or BaseJumpRateModelV2.sol

### Impact

DOS in `utilizationRate()` in an edge case (i.e when `(cash + borrows - reserves) == 0`)

#### Proof of Concept

[Take a look at utilizationRate()](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/WhitePaperInterestRateModel.sol#L86-L97)

```

    function utilizationRate(
        uint256 cash,
        uint256 borrows,
        uint256 reserves
    ) public pure returns (uint256) {
        // Utilization rate is 0 when there are no borrows
        if (borrows == 0) {
            return 0;
        }

        return (borrows * BASE) / (cash + borrows - reserves);
    }

```

If the denominator in any case equates 0, i.e `(cash + borrows - reserves) == 0` the execution reverts due to a division by zero.

### Recommendation

The calculation of the denominator in this case, `cash + borrows - reserves` should first be checked not to equate to zero before attempting the division.

The function could be changed to

```
    function utilizationRate(
        uint256 cash,
        uint256 borrows,
        uint256 reserves
    ) public pure returns (uint256) {
        // Utilization rate is 0 when there are no borrows
        if (borrows == 0) {
            return 0;
        }

    If ((cash + borrows - reserves) != 0){
        return (borrows * BASE) / (cash + borrows - reserves);
    }
return 0
    }
```

## [L-05] VToken.sol: Wrong steps in the execution of the `_redeemFresh()` functions

### Vulnerability Details

inconsistency in the execution code block of the \_redeemFresh() function

Take a look at the \_redeemFresh() function:

```

    function _redeemFresh(
        address redeemer,
        uint256 redeemTokensIn,
        uint256 redeemAmountIn
    ) internal {
        require(redeemTokensIn == 0 || redeemAmountIn == 0, "one of redeemTokensIn or redeemAmountIn must be zero");

        /* Verify market's block number equals current block number */
        if (accrualBlockNumber != _getBlockNumber()) {
            revert RedeemFreshnessCheck();
        }

        /* exchangeRate = invoke Exchange Rate Stored() */
        Exp memory exchangeRate = Exp({ mantissa: _exchangeRateStored() });

        uint256 redeemTokens;
        uint256 redeemAmount;
        /* If redeemTokensIn > 0: */
        if (redeemTokensIn > 0) {
            /*
             * We calculate the exchange rate and the amount of underlying to be redeemed:
             *  redeemTokens = redeemTokensIn
             *  redeemAmount = redeemTokensIn x exchangeRateCurrent
             */
            redeemTokens = redeemTokensIn;
            redeemAmount = mul_ScalarTruncate(exchangeRate, redeemTokensIn);
        } else {
            /*
             * We get the current exchange rate and calculate the amount to be redeemed:
             *  redeemTokens = redeemAmountIn / exchangeRate
             *  redeemAmount = redeemAmountIn
             */
            redeemTokens = div_(redeemAmountIn, exchangeRate);
            redeemAmount = redeemAmountIn;
        }

        // Require tokens is zero or amount is also zero
        if (redeemTokens == 0 && redeemAmount > 0) {
            revert("redeemTokens zero");
        }

        /* Fail if redeem not allowed */
        comptroller.preRedeemHook(address(this), redeemer, redeemTokens);

        /* Fail gracefully if protocol has insufficient cash */
        if (_getCashPrior() - totalReserves < redeemAmount) {
            revert RedeemTransferOutNotPossible();
        }
                /////////////////////////
        // EFFECTS & INTERACTIONS
        // (No safe failures beyond this point)

        /*
         * We write the previously calculated values into storage.
         *  Note: Avoid token reentrancy attacks by writing reduced supply before external transfer.
         */
        totalSupply = totalSupply - redeemTokens;
        uint256 balanceAfter = accountTokens[redeemer] - redeemTokens;
        accountTokens[redeemer] = balanceAfter;

        /*
         * We invoke _doTransferOut for the redeemer and the redeemAmount.
         *  On success, the vToken has redeemAmount less of cash.
         *  _doTransferOut reverts if anything goes wrong, since we can't be sure if side effects occurred.
         */
        _doTransferOut(redeemer, redeemAmount);

        /* We emit a Transfer event, and a Redeem event */
        emit Transfer(redeemer, address(this), redeemTokens);
        emit Redeem(redeemer, redeemAmount, redeemTokens, balanceAfter);
    }

```

So as not to make report unnecessarily bogus take a look at other sister functions, i.e: `\_borrowFresh(), \_repayBorrowFresh(), \_liquidateBorrowFresh(), \_seize(), \_transferTokens(), \_mintFresh() `. In each aforementioned function the first step in the execution of the functions is to query the comptroller to know if the action is allowed, i.e in a case of` _mintfresh()` a preMintHook is checked and would fail if mint is not allowed:

```

comptroller.preMintHook(address(this), minter, mintAmount);

```

And to each other case, their respective hooks are used, now the inconsistency comes in as only in the case of the `_redeemFresh()` is the preRedeemHook not checked as the first step and just doesn't make sense as there is no need to for all other executions if the redeem is not allowed in the comptroller.

Obviously this is submitted as a low as i couldn't find an attack window for this but this just hints a technical flaw.

### Recommendation

In short code should be consistent

## [L-06] VToken.sol: The error being `NO_ERROR ` in most cases is completely a downside of protocol as users can't know the reasons to why their transaction failed

### Vulnerability Details

Check out [VToken.sol](https://github.com/code-423n4/2023-05-venus/blob/main/contracts/VToken.sol)
In most cases the error is `NO_ERROR ` even in cases it shouldn't be, this means that users can't really know the reasons to why their tx failed, and is a very big flaw for off-chain services

### Recommendation

Though this is done for compatibility with Venus core tooling, this should still undergo a rethink

## [L-07] RiskFund.sol: Use of block.timestamp for `_swapAsset()` should be reconsidered

### Vulnerability Details

There are many instances of block.timestamp usage within the project, most especially in the implementation of the `IPancakeswapV2Router(pancakeSwapRouter).swapExactTokensForTokens() `in the `RiskFund._swapAsset()`function block.timestamp can be influenced by miners to a certain degree, so
developers should be aware that this may have some risk if miners collude
on time manipulation to influence the price oracles
Note that this is in relation to the swapForExactTokens function

[RiskFund.\_swapAsset():](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#L225-L274)

```

    function _swapAsset(
        VToken vToken,
        address comptroller,
        uint256 amountOutMin,
        address[] calldata path
    ) internal returns (uint256) {
        require(amountOutMin != 0, "RiskFund: amountOutMin must be greater than 0 to swap vToken");
        require(amountOutMin >= minAmountToConvert, "RiskFund: amountOutMin should be greater than minAmountToConvert");
        uint256 totalAmount;

        address underlyingAsset = VTokenInterface(address(vToken)).underlying();
        uint256 balanceOfUnderlyingAsset = poolsAssetsReserves[comptroller][underlyingAsset];

        ComptrollerViewInterface(comptroller).oracle().updatePrice(address(vToken));

        uint256 underlyingAssetPrice = ComptrollerViewInterface(comptroller).oracle().getUnderlyingPrice(
            address(vToken)
        );

        if (balanceOfUnderlyingAsset > 0) {
            Exp memory oraclePrice = Exp({ mantissa: underlyingAssetPrice });
            uint256 amountInUsd = mul_ScalarTruncate(oraclePrice, balanceOfUnderlyingAsset);

            if (amountInUsd >= minAmountToConvert) {
                assetsReserves[underlyingAsset] -= balanceOfUnderlyingAsset;
                poolsAssetsReserves[comptroller][underlyingAsset] -= balanceOfUnderlyingAsset;

                if (underlyingAsset != convertibleBaseAsset) {
                    require(path[0] == underlyingAsset, "RiskFund: swap path must start with the underlying asset");
                    require(
                        path[path.length - 1] == convertibleBaseAsset,
                        "RiskFund: finally path must be convertible base asset"
                    );
                    IERC20Upgradeable(underlyingAsset).safeApprove(pancakeSwapRouter, 0);
                    IERC20Upgradeable(underlyingAsset).safeApprove(pancakeSwapRouter, balanceOfUnderlyingAsset);
                    uint256[] memory amounts = IPancakeswapV2Router(pancakeSwapRouter).swapExactTokensForTokens(
                        balanceOfUnderlyingAsset,
                        amountOutMin,
                        path,
                        address(this),
                        //@audit isn't using block.timestamp as a window too short?
                        block.timestamp
                    );
                    totalAmount = amounts[path.length - 1];
                } else {
                    totalAmount = balanceOfUnderlyingAsset;
                }
            }
        }

        return totalAmount;

```

### Recommendation

One could advise that you use `block.number` instead of `block.timestamp` or now to reduce the risk of Maximal Extractable Value (MEV) attacks. Check if the timescale of the
project occurs across years, days and months rather than seconds. If possible, it is recommended to use Oracles.

## [L-08] Confusing names of scales in ExponentialNoError.sol

### Vulnerability Details

[ExponentialNoError.sol#L20-L23](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/ExponentialNoError.sol#L20-L23)

```

    uint256 internal constant expScale = 1e18;
    uint256 internal constant doubleScale = 1e36;
    uint256 internal constant halfExpScale = expScale / 2;
    uint256 internal constant mantissaOne = expScale;

```

One could argue that the calculation of the scales are somewhat not completely correct, cause as their name suggests

```expScale = 1e18;
doubleScale = 1e36:
halfExpScale = expScale / 2;
```

the third one from the above is not a half scale cause a half scale should be `1e9` but instead here it's just `5e17` which would cause complete lack of correct values where implemented.

Ps: I only submitted this as a low as i couldn't really think of any bug class that could make this exploitable

### Recommendation

Better names should be used or better still comments could be added for more clarification on this

## [L-09] Missing Zero address checks

### Vulnerability Details

Multiple instances within contracts in scope, in intializers, constructors, below is the initializer from the PoolRegistry.sol contract:

[Initialize()](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Pool/PoolRegistry.sol#L164-L180)

```

function initialize(
VTokenProxyFactory vTokenFactory*,
JumpRateModelFactory jumpRateFactory*,
WhitePaperInterestRateModelFactory whitePaperFactory*,
Shortfall shortfall*,
address payable protocolShareReserve*,
address accessControlManager*
) external initializer {
**Ownable2Step_init();
**AccessControlled*init_unchained(accessControlManager*);

        vTokenFactory = vTokenFactory_;
        jumpRateFactory = jumpRateFactory_;
        whitePaperFactory = whitePaperFactory_;
        _setShortfallContract(shortfall_);
        _setProtocolShareReserve(protocolShareReserve_);
    }

```

### Recommendation

Lack of zero address checks procedure for critical operations leaves them error-prone. Consider adding zero address checks on the critical functions.

## [L-10] Consider isContract checks in constructors

### Vulnerability Details

Most of the contracts within scope set addresses based on the value that's been provided to them in the constructor, where as most include checks that the address provided is not 0x0 it does not check that these addresses is a smart contract.
For example take a look at the [UpgradeableBeacon.sol, L6-9](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Proxy/UpgradeableBeacon.sol#L6-L9)

```

    constructor(address implementation_) UpgradeableBeacon(implementation_) {
        require(implementation_ != address(0), "Invalid implementation address");
    }

```

The only check added here is that `address != 0` but if this input gets mistakenly passed with the wrong address, an isContract() check would atleast stop the implementation from being set to the zero address.
Note that this is just one instance and multiple contracts within scope could emulate this as a another layer of protection against setting addresses to the wrong ones

### Recommendation

Consider adding `isContract()` checks

## [L-11] Renouncing ownership could break some functionalities

### Vulnerability Details

The Ownable2StepUpgradeable.sol is used and this inherits the Ownable contract which has the `renounceOwnership()` function.

Note that this function breaks the 2 step that's otherwise provided by the Ownable2StepUpgradeable.sol but rather just transfers the ownershp to the zero address in one step which would lead to a break of the whole protocol's logic, most importantly to note that if this gets called, protocol no longer has access to all `onlyOwner() `functions, which means that markets can't get paused incase of an emergency, a new operator can't be set incase access to previous ones gets missing all setters only the owners have acess to can no longer be set etc...

```
    function renounceOwnership() public virtual onlyOwner {
        _transferOwnership(address(0));
    }

```

https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/dd8ca8adc47624c5c5e2f4d412f5f421951dcc25/contracts/access/Ownable2StepUpgradeable.sol#L20

[Shortfall.sol](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/dd8ca8adc47624c5c5e2f4d412f5f421951dcc25/contracts/access/Ownable2StepUpgradeable.sol#L20)

```contract Shortfall is Ownable2StepUpgradeable, AccessControlledV8, ReentrancyGuardUpgradeable, IShortfall

```

[ProtocolShareReserve.sol](https://github.com/code-423n4/2023-05-venus/blob/9853f6f4fe906b635e214b22de9f627c6a17ba5b/contracts/RiskFund/ProtocolShareReserve.sol#L12)

```
contract ProtocolShareReserve is Ownable2StepUpgradeable, ExponentialNoError, ReserveHelpers, IProtocolShareReserve {

```

And multiple other instances within code.

### Recommendation

Reconsider if the renounceOwnership function is really needed and if not do remove it

## [NC-01] Comptroller.sol: redundant step in preMintHook()

### Vulnerability Details

Take a look at the [preMintHook() function](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L239-L278)

```
    /**
     * @notice Checks if the account should be allowed to mint tokens in the given market
     * @param vToken The market to verify the mint against
     * @param minter The account which would get the minted tokens
     * @param mintAmount The amount of underlying being supplied to the market in exchange for tokens
     * @custom:error ActionPaused error is thrown if supplying to this market is paused
     * @custom:error MarketNotListed error is thrown when the market is not listed
     * @custom:error SupplyCapExceeded error is thrown if the total supply exceeds the cap after minting
     * @custom:access Not restricted
     */
    function preMintHook(
        address vToken,
        address minter,
        uint256 mintAmount
    ) external override {
        _checkActionPauseState(vToken, Action.MINT);

        if (!markets[vToken].isListed) {
            revert MarketNotListed(address(vToken));
        }

        uint256 supplyCap = supplyCaps[vToken];
        // Skipping the cap check for uncapped coins to save some gas
        if (supplyCap != type(uint256).max) {
            uint256 vTokenSupply = VToken(vToken).totalSupply();
            Exp memory exchangeRate = Exp({ mantissa: VToken(vToken).exchangeRateStored() });
            uint256 nextTotalSupply = mul_ScalarTruncateAddUInt(exchangeRate, vTokenSupply, mintAmount);
            if (nextTotalSupply > supplyCap) {
                revert SupplyCapExceeded(vToken, supplyCap);
            }
        }

        // Keep the flywheel moving
        uint256 rewardDistributorsCount = rewardsDistributors.length;

        for (uint256 i; i < rewardDistributorsCount; ++i) {
            rewardsDistributors[i].updateRewardTokenSupplyIndex(vToken);
            rewardsDistributors[i].distributeSupplierRewardToken(vToken, minter);
        }
    }
```

As seen at L261, the execution skips the cap check for the uncapped coins to save some gas, I think for a case where the coins are already at their supply caps this execution should also be skipped

### Recommendation

The condition at L261-262 could be changed to:

```
 vTokenSupply =  VToken(vToken).totalSupply() )
if ((supplyCap != type(uint256).max) ||  vTokenSupply != supplyCap){
```

## [NC-02] Inconsistent storage gaps

### Vulnerability Details

When using the proxy pattern for upgrades, it is common practice to include storage gaps on parent contracts to reserve space for potential future variables. The size is typically chosen so that all contracts have the same number of variables (usually 50). However, the code base uses inconsistent sizes and does not always include a gap at all.
for example the `ComptrollerStorage.sol`has gaps set to 50, but the amount gap should be is `50 - "the amount of storage said contract has used"`, same thing happens in `vTokenInterfaces.sol`. I believe it should be 47 in `ReserveHelpers.sol`
[ComptrollerStorage.sol#L122](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/ComptrollerStorage.sol#L122)
[ReserveHelpers.sol#L26](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/ReserveHelpers.sol#L26)

### Recommendation

Consider using consistently sized storage gaps in all contracts that have not yet been deployed so as to facilitate safe upgrades.

## [NC-03] Some typos are in the docs and some were missed by bot

### Vulnerability Details

For docs

1. Here is the description to the "claim user function"

```

Allows users to claim their earnings from a specific contract. The user is required to enter the prediction **contract** id and an array of the claimable rounds. All of the rounds entered must be claimable, otherwise the transaction will revert.

```

2. Here under "How to interact with PRDT PRO smart contracts"

```

Step 4: Finish adding a custom ABI by **imputing**:

```

For code

1. [RiskFund.sol#L256](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/RiskFund/RiskFund.sol#L256)

2. [RewardsDistributor.sol#L411)](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Rewards/RewardsDistributor.sol#L411)

### Recommendation

### Recommendation

For Docs:

Change to

1.

```

Allows users to claim their earnings from a specific contract. The user is required to enter the prediction **contract** id and an array of the claimable rounds. All of the rounds entered must be claimable, otherwise the transaction will revert.

```

2.

```

Step 4: Finish adding a custom ABI by **inputing**:

```

For Code:

1. change error message from

```
"RiskFund: finally path must be convertible base asset"
```

to

```
"RiskFund: FINAL path must be convertible base asset"
```

2. change

```
     * @dev Note: If there is not enough REWARD TOKEN, we do not perform the transfer all.
```

to

```
     * @dev Note: If there is not enough REWARD TOKEN, we do not perform the transfer **AT**all.
```

## [NC-04] Some events are missing complete indexed fields

### Vulnerability Details

Multiple instances within code

### Recommendation

Index should be added to events when possible

## [NC-05] Comptroller.sol: suggestion to rename minLiquidatableCollateral to maxLiquidatableCollateral

### Vulnerability Details

Take a look at the [preMintHook() function](https://github.com/code-423n4/2023-05-venus/blob/8be784ed9752b80e6f1b8b781e2e6251748d0d7e/contracts/Comptroller.sol#L578-L626)

```
    function healAccount(address user) external {
        VToken[] memory userAssets = accountAssets[user];
        uint256 userAssetsCount = userAssets.length;

        address liquidator = msg.sender;
        // We need all user's markets to be fresh for the computations to be correct
        for (uint256 i; i < userAssetsCount; ++i) {
            userAssets[i].accrueInterest();
            oracle.updatePrice(address(userAssets[i]));
        }

        AccountLiquiditySnapshot memory snapshot = _getCurrentLiquiditySnapshot(user, _getLiquidationThreshold);

        if (snapshot.totalCollateral > minLiquidatableCollateral) {
            revert CollateralExceedsThreshold(minLiquidatableCollateral, snapshot.totalCollateral);
        }

        if (snapshot.shortfall == 0) {
            revert InsufficientShortfall();
        }

        // percentage = collateral / (borrows * liquidation incentive)
        Exp memory collateral = Exp({ mantissa: snapshot.totalCollateral });
        Exp memory scaledBorrows = mul_(
            Exp({ mantissa: snapshot.borrows }),
            Exp({ mantissa: liquidationIncentiveMantissa })
        );

        Exp memory percentage = div_(collateral, scaledBorrows);
        if (lessThanExp(Exp({ mantissa: mantissaOne }), percentage)) {
            revert CollateralExceedsThreshold(scaledBorrows.mantissa, collateral.mantissa);
        }

        for (uint256 i; i < userAssetsCount; ++i) {
            VToken market = userAssets[i];

            (uint256 tokens, uint256 borrowBalance, ) = _safeGetAccountSnapshot(market, user);
            uint256 repaymentAmount = mul_ScalarTruncate(percentage, borrowBalance);

            // Seize the entire collateral
            if (tokens != 0) {
                market.seize(liquidator, user, tokens);
            }
            // Repay a certain percentage of the borrow, forgive the rest
            if (borrowBalance != 0) {
                market.healBorrow(liquidator, user, repaymentAmount);
            }
        }
    }
```

### Recommendation

As seen from function block renaming minLiquidatableCollateral to maxLiquidatableCollateral would massively improve readabilty of code
