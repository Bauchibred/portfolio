# Introduction

A time-boxed security review of the **Index** protocol which was done by **Bauchibred** with a focus on the security aspects of the application's smart contracts implementation.

# Table of Contents

- [Disclaimer](#disclaimer)
- [Severity classification](#Severity-classification)
- [Scope](#scope)
- [About Bauchibred](#About-Bauchibred)
- [About Index](#About-Index)
- [Summary](#summary)
- [Findings](#findings)

# Disclaimer

A smart contract security review can never verify the complete absence of vulnerabilities. This is a time, resource and expertise bound effort where I try to find as many vulnerabilities as possible. I can not guarantee 100% security after the review or even if the review will find any problems with your smart contracts. Subsequent security reviews, bug bounty programs and on-chain monitoring are strongly recommended.

Additionally, this report was prepared for [this competition](https://audits.sherlock.xyz/contests/81). which took place over a span of **26** days from **_19th May, 2023_** to **_14th June, 2023_** on [Sherlock](https://www.sherlock.xyz/). Given the public nature of the contest, an appointed judge, aided by a panel of other judges and security researchers including myself, had the discretion to determine the final severity of each finding. Due to the inherently subjective nature of such judgments, the final severity assigned might differ from my own classifications. For clarity on the judge's final decisions regarding severity, please refer to [this link](https://github.com/sherlock-audit/2023-05-Index-judging/issues).

# Severity classification

| Severity                                                                                                                                                                                            | Description                                                                                                                                                                                                                                    |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ![](https://camo.githubusercontent.com/a0b140cbe7b198d62804d91996e3c09c6803cfbd484c61974af51892481f840e/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d437269746963616c2d643130623062) | A **directly** exploitable security vulnerability that leads to stolen/lost/locked/compromised assets or catastrophic denial of service.                                                                                                       |
| ![](https://camo.githubusercontent.com/77229c2f86cd6aaaaaeb3879eac44712b0e095147962e9058e455d0094f500d3/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d486967682d726564)               | A security vulnerability or bug that can affect the correct functioning of the system, lead to incorrect states or denial of service. It may not be directly exploitable or may require certain, external conditions in order to be exploited. |
| ![](https://camo.githubusercontent.com/d2cf6c2836b2143aeeb65c08b9c5aa1eb34a6fb8ad6fc55ba4345c467b64378a/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d656469756d2d6f72616e6765)     | Assets not at direct risk, but the function of the protocol or its availability could be impacted, or leak value with a hypothetical attack path with stated assumptions, but external requirements.                                           |
| ![](https://camo.githubusercontent.com/d42acfb8cb8228c156f34cb1fab83f431bf1fbebc102d922950f945b45e05587/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d696e6f722d79656c6c6f77)       | A violation of common best practices or incorrect usage of primitives, which may not currently have a major impact on security, but may do so in the future or introduce inefficiencies.                                                       |

# Scope

This review was focused on the following scope:

[index-coop-smart-contracts @ 317dfb677e9738fc990cf69d198358065e8cb595](https://github.com/IndexCoop/index-coop-smart-contracts/tree/317dfb677e9738fc990cf69d198358065e8cb595)

- [index-coop-smart-contracts/contracts/adapters/AaveV3LeverageStrategyExtension.sol](index-coop-smart-contracts/contracts/adapters/AaveV3LeverageStrategyExtension.sol)
- [index-coop-smart-contracts/contracts/manager/BaseManagerV2.sol](index-coop-smart-contracts/contracts/manager/BaseManagerV2.sol)
- [index-coop-smart-contracts/contracts/adapters/AaveLeverageStrategyExtension.sol](index-coop-smart-contracts/contracts/adapters/AaveLeverageStrategyExtension.sol)
- [index-coop-smart-contracts/contracts/lib/BaseExtension.sol](index-coop-smart-contracts/contracts/lib/BaseExtension.sol)
- [index-coop-smart-contracts/contracts/lib/StringArrayUtils.sol](index-coop-smart-contracts/contracts/lib/StringArrayUtils.sol)

[index-protocol @ 86be7ee76d9a7e4f7e93acfc533216ebef791c89](https://github.com/IndexCoop/index-protocol/tree/86be7ee76d9a7e4f7e93acfc533216ebef791c89)

- [index-protocol/contracts/protocol/Controller.sol](index-protocol/contracts/protocol/Controller.sol)
- [index-protocol/contracts/protocol/IntegrationRegistry.sol](index-protocol/contracts/protocol/IntegrationRegistry.sol)
- [index-protocol/contracts/protocol/SetToken.sol](index-protocol/contracts/protocol/SetToken.sol)
- [index-protocol/contracts/protocol/SetTokenCreator.sol](index-protocol/contracts/protocol/SetTokenCreator.sol)
- [index-protocol/contracts/protocol/integration/lib/AaveV3.sol](index-protocol/contracts/protocol/integration/lib/AaveV3.sol)
- [index-protocol/contracts/protocol/modules/v1/AaveV3LeverageModule.sol](index-protocol/contracts/protocol/modules/v1/AaveV3LeverageModule.sol)
- [index-protocol/contracts/protocol/modules/v1/AirdropModule.sol](index-protocol/contracts/protocol/modules/v1/AirdropModule.sol)
- [index-protocol/contracts/protocol/modules/v1/AmmModule.sol](index-protocol/contracts/protocol/modules/v1/AmmModule.sol)
- [index-protocol/contracts/protocol/modules/v1/ClaimModule.sol](index-protocol/contracts/protocol/modules/v1/ClaimModule.sol)
- [index-protocol/contracts/protocol/modules/v1/DebtIssuanceModuleV2.sol](index-protocol/contracts/protocol/modules/v1/DebtIssuanceModuleV2.sol)
- [index-protocol/contracts/protocol/modules/v1/TradeModule.sol](index-protocol/contracts/protocol/modules/v1/TradeModule.sol)
- [index-protocol/contracts/protocol/modules/v1/WrapModuleV2.sol](index-protocol/contracts/protocol/modules/v1/WrapModuleV2.sol)
- [index-protocol/contracts/protocol/modules/v1/StreamingFeeModule.sol](index-protocol/contracts/protocol/modules/v1/StreamingFeeModule.sol)
- [index-protocol/contracts/protocol/lib/ModuleBase.sol](index-protocol/contracts/protocol/lib/ModuleBase.sol)
- [index-protocol/contracts/protocol/lib/Invoke.sol](index-protocol/contracts/protocol/lib/Invoke.sol)
- [index-protocol/contracts/protocol/lib/IssuanceValidationUtils.sol](index-protocol/contracts/protocol/lib/IssuanceValidationUtils.sol)
- [index-protocol/contracts/protocol/lib/Position.sol](index-protocol/contracts/protocol/lib/Position.sol)
- [index-protocol/contracts/lib/AddressArrayUtils.sol](index-protocol/contracts/lib/AddressArrayUtils.sol)
- [index-protocol/contracts/lib/PreciseUnitMath.sol](index-protocol/contracts/lib/PreciseUnitMath.sol)
- [index-protocol/contracts/protocol/modules/v1/DebtIssuanceModule.sol](index-protocol/contracts/protocol/modules/v1/DebtIssuanceModule.sol)
- [index-protocol/contracts/protocol/lib/ResourceIdentifier.sol](index-protocol/contracts/protocol/lib/ResourceIdentifier.sol)
- [index-protocol/contracts/lib/ExplicitERC20.sol](index-protocol/contracts/lib/ExplicitERC20.sol)

~ 4225 NSLOC in Y contracts

NSLOC stands for 'Normalized Source Code', which is one of the custom measurements security researchers use when evaluating the complexity of a codebase.

The steps below are taken to get the NSLOC count of a file

1.  For all functions, reduce any multiline function declarations to a single line.
2.  Remove all comments
3.  Remove all empty lines
4.  Count the remaining lines

# About **Bauchibred**

**Bauchibred** is an independent smart contract security researcher. Having found numerous security vulnerabilities in various protocols, he does his best to contribute to the blockchain ecosystem and its protocols by putting time and effort into security research & reviews. Check his previous work [here](https://github.com/bauchibred/audits) or reach out on Twitter [@bauchibred](https://twitter.com/bauchibred).

# About Index

**Index Coop** builds decentralized structured products that make crypto simple, accessible, and secure and is planned to be deployed to most EVM compatible chains including: Ethereum mainnet, Polygon, Optimism, Avalanche, and Arbitrum.

# Summary

This report contains **4 medium** severity issues and **3 minor** issues found by **Bauchibred** during the course of the time-boxed audit.

| #      | Title                                                                                                                                                                 | Severity                                         | Status                                                  |
| ------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ | ------------------------------------------------------- |
| [M-01] | Protocol doesn't completely protect itself from `LTV = 0` tokens                                                                                                      | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [M-02] | Invoke Library: Lack of Return Value Checks in ERC20 operations                                                                                                       | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-03] | AaveLeverageStrategyExtension: Usage of Deprecated Chainlink API is a Risk                                                                                            | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-04] | DebtIssuanceModuleV2.sol: Current Implementenation of `_resolveEquityPositions()` and `_resolveDebtPositions()` causes Potential issues when Dealing with Some Tokens | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-01] | BaseExtension.sol: `onlyEOA` modifier that ensures call is from EOA might not hold completely true in the future                                                      | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [L-02] | AaveV3LeverageStrategyExtension is at Risk of DOS if Chainlink Access is Blocked                                                                                      | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [L-03] | AaveLeverageStrategyExtension: Unnecessary Precision loss while delevering                                                                                            | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |

# Findings

## [M-01] Protocol doesn't completely protect itself from `LTV = 0` tokens

### Summary

The AaveLeverageStrategyExtension does not completely protect against tokens with a Loan-to-Value (LTV) of 0. Tokens with an LTV of 0 in Aave V3 pose significant risks, as they cannot be used as collateral to borrow upon a breaking withdraw. Moreover, LTVs of assets could be set to 0, even though they currently aren't, it could create substantial problems with potential disruption of multiple functionalities. This bug could cause a Denial-of-Service (DoS) situation in some cases, and has a potential to impact the borrowing logic in the protocol, leading to an unintentionally large perceived borrowing limit.

### Vulnerability Detail

When an AToken has LTV = 0, Aave restricts the usage of certain operations. Specifically, if a user owns at least one AToken as collateral with an LTV = 0, certain operations could revert:

1. **Withdraw**: If the asset being withdrawn is collateral and the user is borrowing something, the operation will revert if the withdrawn collateral is an AToken with LTV > 0.
2. **Transfer**: If the asset being transferred is an AToken with LTV > 0 and the sender is using the asset as collateral and is borrowing something, the operation will revert.
3. **Set the reserve of an AToken as non-collateral**: If the AToken being set as non-collateral is an AToken with LTV > 0, the operation will revert.

Take a look at [AaveLeverageStrategyExtension.sol#L1050-L1119](https://github.com/sherlock-audit/2023-05-Index/blob/3190057afd3085143a31746d65045a0d1bacc78c/index-coop-smart-contracts/contracts/adapters/AaveLeverageStrategyExtension.sol#L1050-L1119)

```solidity
    /**
     * Calculate total notional rebalance quantity and chunked rebalance quantity in collateral units.
     *
     * return uint256          Chunked rebalance notional in collateral units
     * return uint256          Total rebalance notional in collateral units
     */
    function _calculateChunkRebalanceNotional(
        LeverageInfo memory _leverageInfo,
        uint256 _newLeverageRatio,
        bool _isLever
    )
        internal
        view
        returns (uint256, uint256)
    {
        // Calculate absolute value of difference between new and current leverage ratio
        uint256 leverageRatioDifference = _isLever ? _newLeverageRatio.sub(_leverageInfo.currentLeverageRatio) : _leverageInfo.currentLeverageRatio.sub(_newLeverageRatio);

        uint256 totalRebalanceNotional = leverageRatioDifference.preciseDiv(_leverageInfo.currentLeverageRatio).preciseMul(_leverageInfo.action.collateralBalance);

        uint256 maxBorrow = _calculateMaxBorrowCollateral(_leverageInfo.action, _isLever);

        uint256 chunkRebalanceNotional = Math.min(Math.min(maxBorrow, totalRebalanceNotional), _leverageInfo.twapMaxTradeSize);

        return (chunkRebalanceNotional, totalRebalanceNotional);
    }

    /**
     * Calculate the max borrow / repay amount allowed in base units for lever / delever. This is due to overcollateralization requirements on
     * assets deposited in lending protocols for borrowing.
     *
     * For lever, max borrow is calculated as:
     * (Net borrow limit in USD - existing borrow value in USD) / collateral asset price adjusted for decimals
     *
     * For delever, max repay is calculated as:
     * Collateral balance in base units * (net borrow limit in USD - existing borrow value in USD) / net borrow limit in USD
     *
     * Net borrow limit for levering is calculated as:
     * The collateral value in USD * Aave collateral factor * (1 - unutilized leverage %)
     *
     * Net repay limit for delevering is calculated as:
     * The collateral value in USD * Aave liquiditon threshold * (1 - unutilized leverage %)
     *
     * return uint256          Max borrow notional denominated in collateral asset
     */
    function _calculateMaxBorrowCollateral(ActionInfo memory _actionInfo, bool _isLever) internal view returns(uint256) {

        // Retrieve collateral factor and liquidation threshold for the collateral asset in precise units (1e16 = 1%)
        ( , uint256 maxLtvRaw, uint256 liquidationThresholdRaw, , , , , , ,) = strategy.aaveProtocolDataProvider.getReserveConfigurationData(address(strategy.collateralAsset));

        // Normalize LTV and liquidation threshold to precise units. LTV is measured in 4 decimals in Aave which is why we must multiply by 1e14
        // for example ETH has an LTV value of 8000 which represents 80%
        if (_isLever) {
            uint256 netBorrowLimit = _actionInfo.collateralValue
                .preciseMul(maxLtvRaw.mul(10 ** 14))
                .preciseMul(PreciseUnitMath.preciseUnit().sub(execution.unutilizedLeveragePercentage));

            return netBorrowLimit
                .sub(_actionInfo.borrowValue)
                .preciseDiv(_actionInfo.collateralPrice);
        } else {
            uint256 netRepayLimit = _actionInfo.collateralValue
                .preciseMul(liquidationThresholdRaw.mul(10 ** 14))
                .preciseMul(PreciseUnitMath.preciseUnit().sub(execution.unutilizedLeveragePercentage));

            return _actionInfo.collateralBalance
                .preciseMul(netRepayLimit.sub(_actionInfo.borrowValue))
                .preciseDiv(netRepayLimit);
        }
    }
```

Apart from the aforementioned issue with `LTV = 0` tokens, there's another issue with the `_calculateMaxBorrowCollateral()` function. When LTV = 0, `maxLtvRaw` also equals 0, leading to a `netBorrowLimit` of 0. When the borrowing value is subtracted from this, it results in an underflow, causing the borrowing limit to appear incredibly large. This essentially breaks the borrowing logic of the protocol.

### Impact

This bug could potentially disrupt the entire borrowing logic within the protocol by inflating the perceived borrowing limit. This could lead to users borrowing an unlimited amount of assets due to the underflow error. In extreme cases, this could lead to a potential loss of user funds or even a complete protocol shutdown, thus impacting user trust and the overall functionality of the protocol.

### Recommendation

The protocol should consider implementing additional protections against tokens with an LTV of 0.

## [M-02] Invoke Library: Lack of Return Value Checks in ERC20 operations

### Summary

The `Invoke` library in the project lacks return value checks in ERC20 transfers/approvals. This could lead to silent failures and incorrect internal accounting.

### Vulnerability Detail

The vulnerability arises from the following code snippets in the [Invoke.sol](https://github.com/sherlock-audit/2023-05-Index/blob/3190057afd3085143a31746d65045a0d1bacc78c/index-protocol/contracts/protocol/lib/Invoke.sol#L1-L136) library:

```solidity
function invokeApprove(ISetToken _setToken, address _token, address _spender, uint256 _quantity)
    internal
{
    // @audit the return value of the execution is not checked

    bytes memory callData = abi.encodeWithSignature("approve(address,uint256)", _spender, _quantity);
    _setToken.invoke(_token, 0, callData);
}

function invokeTransfer(ISetToken _setToken, address _token, address _to, uint256 _quantity)
    internal
{
    // @audit the return value of the execution is not checked

    if (_quantity > 0) {
        bytes memory callData = abi.encodeWithSignature("transfer(address,uint256)", _to, _quantity);
        _setToken.invoke(_token, 0, callData);
    }
}

function strictInvokeTransfer(ISetToken _setToken, address _token, address _to, uint256 _quantity)
    internal
{
    // @audit same thing
    if (_quantity > 0) {
        uint256 existingBalance = IERC20(_token).balanceOf(address(_setToken));
        Invoke.invokeTransfer(_setToken, _token, _to, _quantity);
        uint256 newBalance = IERC20(_token).balanceOf(address(_setToken));
        require(newBalance == existingBalance.sub(_quantity), "Invalid post transfer balance");
    }
}
```

The code snippets show that the return values of ERC20 operations are not checked. This can result in silent failures and incorrect internal accounting.

### Impact

The lack of return value checks in ERC20 operations can lead to silent failures. For example if a transfer fails, it will not be detected, resulting in incorrect internal accounting.

### Recommendation

To address this vulnerability, it is recommended to implement return value checks for all ERC20 transfers/approvals in the `Invoke` library. By checking the return values, the library can detect transfer failures and handle them appropriately, ensuring accurate internal accounting.

One recommended approach is to use the SafeERC20 library provided by OpenZeppelin. The library includes safe versions of ERC20 transfer functions that handle return value checks and revert the transaction if the transfer fails. Integrating the SafeERC20 library will enhance the robustness of the token transfer operations in the `Invoke` library.

## [M-03] AaveLeverageStrategyExtension: Usage of Deprecated Chainlink API is a Risk

### Summary

The AaveLeverageStrategyExtension contract uses Chainlink's deprecated `latestAnswer()` API in its `_createActionInfo()` function. This function is extensively used within the contract, including in the `engage()`, `getCurrentLeverageRatio()`, `getChunkRebalanceNotional()`, and internal `_getAndValidateLeveragedInfo()` functions. The continued use of this deprecated API could potentially lead to stale data and pose a significant risk to the overall functioning of the AaveLeverageStrategyExtension contract.

### Vulnerability Detail

The `_createActionInfo()` function retrieves asset prices using the deprecated `latestAnswer()` method from Chainlink oracles. As indicated by Chainlink, this method is deprecated and may stop working if Chainlink discontinues support. Furthermore, `latestAnswer()` does not have the capability to validate the freshness of the data it retrieves.

This means the function could potentially return stale or outdated price data, which could lead to incorrect calculations of collateral and borrow values in the AaveLeverageStrategyExtension contract. Since the `_createActionInfo()` function is extensively used within the contract, this vulnerability could impact a broad range of operations.

Take a look at [AaveLeverageStrategyExtension.sol#L884-L907](https://github.com/sherlock-audit/2023-05-Index/blob/3190057afd3085143a31746d65045a0d1bacc78c/index-coop-smart-contracts/contracts/adapters/AaveLeverageStrategyExtension.sol#L884-L907)

```solidity
    /**
     * Create the action info struct to be used in internal functions
     *
     * return ActionInfo                Struct containing data used by internal lever and delever functions
     */
    function _createActionInfo() internal view returns(ActionInfo memory) {
        ActionInfo memory rebalanceInfo;

        // Calculate prices from chainlink. Chainlink returns prices with 8 decimal places, but we need 36 - underlyingDecimals decimal places.
        // This is so that when the underlying amount is multiplied by the received price, the collateral valuation is normalized to 36 decimals.
        // To perform this adjustment, we multiply by 10^(36 - 8 - underlyingDecimals)
        int256 rawCollateralPrice = strategy.collateralPriceOracle.latestAnswer();
        rebalanceInfo.collateralPrice = rawCollateralPrice.toUint256().mul(10 ** strategy.collateralDecimalAdjustment);
        int256 rawBorrowPrice = strategy.borrowPriceOracle.latestAnswer();
        rebalanceInfo.borrowPrice = rawBorrowPrice.toUint256().mul(10 ** strategy.borrowDecimalAdjustment);

        rebalanceInfo.collateralBalance = strategy.targetCollateralAToken.balanceOf(address(strategy.setToken));
        rebalanceInfo.borrowBalance = strategy.targetBorrowDebtToken.balanceOf(address(strategy.setToken));
        rebalanceInfo.collateralValue = rebalanceInfo.collateralPrice.preciseMul(rebalanceInfo.collateralBalance);
        rebalanceInfo.borrowValue = rebalanceInfo.borrowPrice.preciseMul(rebalanceInfo.borrowBalance);
        rebalanceInfo.setTotalSupply = strategy.setToken.totalSupply();

        return rebalanceInfo;
    }

```

At L895 and 897 we cam see the double use of the deprecated `latestAnswer()` method.

### Impact

Continued use of deprecated APIs poses a risk of inoperability if the API is discontinued or its functionality is altered. In this case, the failure of `latestAnswer()` would affect the `_createActionInfo()` function, which is instrumental in several core functionalities of the AaveLeverageStrategyExtension contract.

Moreover, the lack of data freshness validation with `latestAnswer()` can lead to the retrieval of stale or outdated prices. As a result, inaccurate calculations of collateral and borrow values could occur, potentially leading to significant financial and operational issues for the contract.

### Recommendation

We recommend switching from the deprecated `latestAnswer()` to the `latestRoundData()` method provided by the Chainlink V3 API. The `latestRoundData()` method provides fresh data and allows for extra validations, enhancing data reliability and accuracy.

## [M-04] DebtIssuanceModuleV2.sol: Current Implementenation of `_resolveEquityPositions()` and `_resolveDebtPositions()` causes Potential issues when Dealing with Some Tokens

### Summary

Two functions, `_resolveEquityPositions()` and `_resolveDebtPositions()` in the updated [DebtIssuanceModuleV2.sol](https://github.com/sherlock-audit/2023-05-Index/blob/3190057afd3085143a31746d65045a0d1bacc78c/index-protocol/contracts/protocol/modules/v1/DebtIssuanceModule.sol#L4), usage of `invokeTransfer()` instead of the `strictInvokeTransfer()` method to transfer tokens is implemented. With this update unlike the old version of [DebtIssuanceModule.sol](https://github.com/sherlock-audit/2023-05-Index/blob/3190057afd3085143a31746d65045a0d1bacc78c/index-protocol/contracts/protocol/modules/v1/DebtIssuanceModule.sol#L137) where `strictInvokeTransfer()` was being used, now there is no protection against fee on transfer tokens and if these tokens are to be allowed inprotocol they could easily break the accounting logic.

### Vulnerability Detail

In both functions, i.e
[Take a look at DebtIssuanceModuleV2.sol#L251-L333](https://github.com/sherlock-audit/2023-05-Index/blob/3190057afd3085143a31746d65045a0d1bacc78c/index-protocol/contracts/protocol/modules/v1/DebtIssuanceModuleV2.sol#L251-L333)

```solidity
    function _resolveEquityPositions(
        ISetToken _setToken,
        uint256 _quantity,
        address _to,
        bool _isIssue,
        address[] memory _components,
        uint256[] memory _componentEquityQuantities,
        uint256 _initialSetSupply,
        uint256 _finalSetSupply
    )
        internal
    {
//ommited for brevity
                } else {
                    _executeExternalPositionHooks(_setToken, _quantity, IERC20(component), false, true);

                    // Call Invoke#invokeTransfer instead of Invoke#strictInvokeTransfer
                    _setToken.invokeTransfer(component, _to, componentQuantity);

//ommited for brevity
                }
            }
        }
    }



    function _resolveDebtPositions(
        ISetToken _setToken,
        uint256 _quantity,
        bool _isIssue,
        address[] memory _components,
        uint256[] memory _componentDebtQuantities,
        uint256 _initialSetSupply,
        uint256 _finalSetSupply
    )
        internal
    {
//ommited for brevity

                    // Call Invoke#invokeTransfer instead of Invoke#strictInvokeTransfer
                    _setToken.invokeTransfer(component, msg.sender, componentQuantity);

                    IssuanceValidationUtils.validateCollateralizationPostTransferOut(_setToken, component, _finalSetSupply);
//ommited for brevity
                }
            }
        }
    }
```

As seen both functions now use Invoke#invokeTransfer instead of Invoke#strictInvokeTransfer
The method `strictInvokeTransfer()` has a mechanism that ensures the final balance of the token is equivalent to the initial balance minus the quantity transferred. This would help in this case as it ensures that if the token being transferred has a transfer fee or is a deflationary token, the transfer wouldn't go through and the balance check would fail.

Now take a look at [Invoke.sol#L66-L112](https://github.com/sherlock-audit/2023-05-Index/blob/3190057afd3085143a31746d65045a0d1bacc78c/index-protocol/contracts/protocol/lib/Invoke.sol#L58-L112)

```solidity
    /**
     * Instructs the SetToken to transfer the ERC20 token to a recipient.
     *
     * @param _setToken        SetToken instance to invoke
     * @param _token           ERC20 token to transfer
     * @param _to              The recipient account
     * @param _quantity        The quantity to transfer
     */
    function invokeTransfer(
        ISetToken _setToken,
        address _token,
        address _to,
        uint256 _quantity
    )
        internal
    {
        if (_quantity > 0) {
            bytes memory callData = abi.encodeWithSignature("transfer(address,uint256)", _to, _quantity);
            _setToken.invoke(_token, 0, callData);
        }
    }

    /**
     * Instructs the SetToken to transfer the ERC20 token to a recipient.
     * The new SetToken balance must equal the existing balance less the quantity transferred
     *
     * @param _setToken        SetToken instance to invoke
     * @param _token           ERC20 token to transfer
     * @param _to              The recipient account
     * @param _quantity        The quantity to transfer
     */
    function strictInvokeTransfer(
        ISetToken _setToken,
        address _token,
        address _to,
        uint256 _quantity
    )
        internal
    {
        if (_quantity > 0) {
            // Retrieve current balance of token for the SetToken
            uint256 existingBalance = IERC20(_token).balanceOf(address(_setToken));

            Invoke.invokeTransfer(_setToken, _token, _to, _quantity);

            // Get new balance of transferred token for SetToken
            uint256 newBalance = IERC20(_token).balanceOf(address(_setToken));

            // Verify only the transfer quantity is subtracted
            require(
                newBalance == existingBalance.sub(_quantity),
                "Invalid post transfer balance"
            );
        }
    }

```

However, it is noted that there are no FEE-ON-TRANSFER tokens interacting with the smart contracts currently. But if there are plans to introduce such tokens in the future, the current implementation of `strictInvokeTransfer()` would cause issues.
The above paragraph is coined from one of the Q/A by sponsors which validates issule
[Discussion](https://github.com/sherlock-audit/2023-05-Index/tree/3190057afd3085143a31746d65045a0d1bacc78c#q-are-there-any-fee-on-transfer-tokens-interacting-with-the-smart-contracts)

```diff
Q: Are there any FEE-ON-TRANSFER tokens interacting with the smart contracts?
A: Not at the moment, but it would be good to know if they can interact with our smart contracts
```

### Impact

Inability to integrate Fee-on-transfer tokens in the future

### Recommendation

If there are plans of integrating fee-on transfer tokens in the future then integrate correct acccounting mesure to know the exact amount of tokens thst are being accounted for

## [L-01] BaseExtension.sol: `onlyEOA` modifier that ensures call is from EOA might not hold completely true in the future

### Summary

The `BaseExtension` contract implements the `onlyEOA` modifier to ensure that the caller is an externally owned account (EOA) and not a smart contract. However, it is important to note that the introduction of [EIP 3074](https://eips.ethereum.org/EIPS/eip-3074) would affect the behavior of this check since it initroduces two new EVM opcodes. While the `onlyEOA` modifier would still hold true for traditional EOAs, users who have signed AUTHCALLs or other EIP 3074 instructions might not be able to execute transactions in the contract since the `require(msg.sender == tx.origin)` condition would prevent them from interacting with the protocol.

NB: The core extension, [AaveLeverageStrategyExtension.sol](https://github.com/sherlock-audit/2023-05-Index/blob/3190057afd3085143a31746d65045a0d1bacc78c/index-coop-smart-contracts/contracts/adapters/AaveLeverageStrategyExtension.sol) contract heavliy uses the `onlyEOA` modifier which it inherits from [BaseExtension.sol]()

### Vulnerability Detail

The `BaseExtension` contract includes the `onlyEOA` modifier to ensure that the caller is an EOA:

[onlyEOA()](https://github.com/sherlock-audit/2023-05-Index/blob/3190057afd3085143a31746d65045a0d1bacc78c/index-coop-smart-contracts/contracts/lib/BaseExtension.sol#L56-L62C1)

```solidity
modifier onlyEOA() {
    require(msg.sender == tx.origin, "Caller must be EOA Address");
    _;
}
```

However, with the introduction of EIP 3074, which allows smart contracts to act on behalf of EOAs using the `AUTH` and `AUTHCALL` instructions, users who have signed such instructions might encounter difficulties executing transactions in the contract. This is because the `require(msg.sender == tx.origin)` condition in the `onlyEOA` modifier would prevent these users from interacting with the protocol.

### Impact

If EIP 3074 is implemented and users have signed AUTHCALLs or other similar instructions, they might not be able to execute transactions in the `BaseExtension` contract and all contracts that inherit from it due to the `onlyEOA` modifier's verification check. This could restrict their access to the protocol's functionalities and potentially result in user experience issues.

Below referenced are the affected code snippets:

[onlyEOA()](https://github.com/sherlock-audit/2023-05-Index/blob/3190057afd3085143a31746d65045a0d1bacc78c/index-coop-smart-contracts/contracts/lib/BaseExtension.sol#L56-L62C1)

[AaveLeverageStrategyExtension.sol](https://github.com/sherlock-audit/2023-05-Index/blob/3190057afd3085143a31746d65045a0d1bacc78c/index-coop-smart-contracts/contracts/adapters/AaveLeverageStrategyExtension.sol)

### Recommendation

To address the potential impact of EIP 3074 and ensure a more inclusive user experience, it is recommended to reconsider the usage of the `onlyEOA` modifier in the `BaseExtension` contract. Instead of relying solely on the `require(msg.sender == tx.origin)` condition, alternative approaches can be explored.

One option is to update the modifier to allow users who have signed AUTHCALLs or other EIP 3074 instructions to interact with the protocol. This can be achieved by removing the `require(msg.sender == tx.origin)` condition and implementing additional checks to validate the caller's authorization, such as maintaining a whitelist of authorized contracts or implementing a role-based access control mechanism.

Alternatively, if it is desired to strictly restrict interactions to traditional EOAs, the modifier can be kept as is. However, it is important to communicate this limitation to users and provide clear instructions on how to interact with the protocol if they have signed AUTHCALLs or similar instructions.

## [L-02] AaveV3LeverageStrategyExtension is at Risk of DOS if Chainlink Access is Blocked

### Summary

The AaveV3LeverageStrategyExtension's functionality could become severely limited if access to the Chainlink oracle data feed is blocked. A denial of service (DOS) could occur, impacting the creation of the ActionInfo struct utilized in internal functions.

NB: This function is extensively used within the AaveLeverageStrategyExtension contract, in functions like `engage()`, `getCurrentLeverageRatio()`, `getChunkRebalanceNotional()`, and internal `_getAndValidateLeveragedInfo()` functions.

### Vulnerability Detail

The AaveV3LeverageStrategyExtension uses the Chainlink oracle data feed to get prices in the `_createActionInfo` function. This function is used to create an ActionInfo struct that stores vital data used by internal lever/delever and other functions.

The vulnerability lies in these lines:

```solidity
int256 rawCollateralPrice = strategy.collateralPriceOracle.latestAnswer();
rebalanceInfo.collateralPrice = rawCollateralPrice.toUint256().mul(10 ** strategy.collateralDecimalAdjustment);

int256 rawBorrowPrice = strategy.borrowPriceOracle.latestAnswer();
rebalanceInfo.borrowPrice = rawBorrowPrice.toUint256().mul(10 ** strategy.borrowDecimalAdjustment);
```

As https://blog.openzeppelin.com/secure-smart-contract-guidelines-the-dangers-of-price-oracles/ mentions, it is possible that Chainlink’s “multisigs can immediately block access to price feeds at will”. When this occurs, the execution of either `strategy.collateralPriceOracle.latestAnswer()` or `strategy.borrowPriceOracle.latestAnswer()` will revert.

As a result, the creation of the ActionInfo struct fails, leading to a cascade of issues in other functions that depend on the ActionInfo struct, and ultimately, causing a DOS.

#### Proof of Concept

Consider a scenario:

1. Chainlink oracle data feed is used for getting prices.
2. Any of the function that calls `_createActionInfo` is executed to get an ActionInfo struct.
3. Suddenly, Chainlink's multisigs block access to price feeds. Executing `strategy.collateralPriceOracle.latestAnswer()` or `strategy.borrowPriceOracle.latestAnswer()` now reverts.
4. The creation of the ActionInfo struct fails, which eventually causes the parent function to fail.
5. If other functions that depend on `_createActionInfo` are called, they will also fail, leading to a potential DOS.

### Impact

Should Chainlink's multisigs block access to price feeds, the AaveV3LeverageStrategyExtension's functionality becomes very limited due to the failed creation of the ActionInfo struct. This effectively results in a denial of service (DOS) attack.

### Recommendation

Refactor `_createActionInfo` to include a try-catch block for the price feed calls. Consider providing a fallback logic when the access to the Chainlink oracle data feed is denied.

Here is a base to build a refactored version on using try-catch:

```solidity
try {
    int256 rawCollateralPrice = strategy.collateralPriceOracle.latestAnswer();
    rebalanceInfo.collateralPrice = rawCollateralPrice.toUint256().mul(10 ** strategy.collateralDecimalAdjustment);
} catch Error(string memory) {
    // Implement fallback logic here
}

try {
    int256 rawBorrowPrice = strategy.borrowPriceOracle.latestAnswer();
    rebalanceInfo.borrowPrice = rawBorrowPrice.toUint256().mul(10 ** strategy.borrowDecimalAdjustment);
} catch Error(string memory) {
    // Implement fallback logic here
}
```

The fallback logic could be obtaining the token's price from another reliable oracle or any other predetermined strategy that ensures continuity in the absence of Chainlink price feeds.

## [L-03] AaveLeverageStrategyExtension: Unnecessary Precision loss while delevering

### Summary

The `_calculateMinRepayUnits()` function, used in the AaveLeverageStrategyExtension contract's `_delever()` function, leads to unnecessary precision loss due to the order of operations. Performing division amidst a sequence of multiplications could cause precision loss due to possible rounding errors.

### Vulnerability Detail

During the execution of the `_delever()` function, an operation in `_calculateMinRepayUnits()` multiplies `_collateralRebalanceUnits` with `_actionInfo.collateralPrice`, divides the result by `_actionInfo.borrowPrice`, and multiplies again by the difference between `PreciseUnitMath.preciseUnit()` and `_slippageTolerance`. This sequence causes an unnecessary precision loss since a division operation is performed before completing all multiplication operations.

Take a look at both functions below to have a visual representation of the vulnerability

[`_delever()`](https://github.com/sherlock-audit/2023-05-Index/blob/3190057afd3085143a31746d65045a0d1bacc78c/index-coop-smart-contracts/contracts/adapters/AaveLeverageStrategyExtension.sol#L772-L797)

```

    /**
     * Calculate delever units Invoke delever on AaveLeverageModule.
     */
    function _delever(
        LeverageInfo memory _leverageInfo,
        uint256 _chunkRebalanceNotional
    )
        internal
    {
        uint256 collateralRebalanceUnits = _chunkRebalanceNotional.preciseDiv(_leverageInfo.action.setTotalSupply);

        uint256 minRepayUnits = _calculateMinRepayUnits(collateralRebalanceUnits, _leverageInfo.slippageTolerance, _leverageInfo.action);

        bytes memory deleverCallData = abi.encodeWithSignature(
            "delever(address,address,address,uint256,uint256,string,bytes)",
            address(strategy.setToken),
            strategy.collateralAsset,
            strategy.borrowAsset,
            collateralRebalanceUnits,
            minRepayUnits,
            _leverageInfo.exchangeName,
            exchangeSettings[_leverageInfo.exchangeName].deleverExchangeData
        );

        invokeManager(address(strategy.leverageModule), deleverCallData);
    }

```

[`_calculateMinRepayUnits()`](https://github.com/sherlock-audit/2023-05-Index/blob/3190057afd3085143a31746d65045a0d1bacc78c/index-coop-smart-contracts/contracts/adapters/AaveLeverageStrategyExtension.sol#L1141-L1152)

```solidity
    /**
     * Derive the min repay units from collateral units for delever. Units are calculated as target collateral rebalance units multiplied by slippage tolerance
     * and pair price (collateral oracle price / borrow oracle price). Output is measured in borrow unit decimals.
     *
     * return uint256           Min position units to repay in borrow asset
     */
    function _calculateMinRepayUnits(uint256 _collateralRebalanceUnits, uint256 _slippageTolerance, ActionInfo memory _actionInfo) internal pure returns (uint256) {

    //@audit the division should be done as the last execution
        return _collateralRebalanceUnits
            .preciseMul(_actionInfo.collateralPrice)
            .preciseDiv(_actionInfo.borrowPrice)
            .preciseMul(PreciseUnitMath.preciseUnit().sub(_slippageTolerance));
    }
```

### Impact

This unnecessary precision loss might cause inaccuracies in the calculation of `minRepayUnits`, which could potentially affect the deleveraging process and the overall strategy operation.

### Recommendation

To address this issue, consider modifying the `_calculateMinRepayUnits()` function to perform all multiplication operations before division, like this:

```solidity
    return _collateralRebalanceUnits
        .preciseMul(_actionInfo.collateralPrice)
        .preciseMul(PreciseUnitMath.preciseUnit().sub(_slippageTolerance))
        .preciseDiv(_actionInfo.borrowPrice);
```

This adjustment minimizes precision loss by reducing the impact of rounding errors caused by division operations.
