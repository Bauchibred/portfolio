# Introduction

A time-boxed security review of the **USSD** protocol which was done by **Bauchibred** with a focus on the security aspects of the application's smart contracts implementation.

# Table of Contents

- [Disclaimer](#disclaimer)
- [Severity classification](#Severity-classification)
- [Scope](#scope)
- [About Bauchibred](#About-Bauchibred)
- [About ProtocolName](#About-USSD)
- [Summary](#summary)
- [Findings](#findings)

# Disclaimer

A smart contract security review can never verify the complete absence of vulnerabilities. This is a time, resource and expertise bound effort where I try to find as many vulnerabilities as possible. I can not guarantee 100% security after the review or even if the review will find any problems with your smart contracts. Subsequent security reviews, bug bounty programs and on-chain monitoring are strongly recommended.

Additionally, this report was prepared for [this competition](https://audits.sherlock.xyz/contests/82). The competition spanned **4** days, from **_May 19th, 2023_** to **_May 23rd, 2023_**, and was hosted on [Sherlock](https://www.sherlock.xyz/). Given the public nature of the contest, an appointed judge, aided by a panel of other judges and security researchers including myself, had the discretion to determine the final severity of each finding. Due to the inherently subjective nature of such judgments, the final severity assigned might differ from my own classifications. For clarity on the judge's final decisions regarding severity, please refer to [this link](https://github.com/sherlock-audit/2023-05-USSD-judging/issues).

# Severity classification

| Severity                                                                                                                                                                                            | Description                                                                                                                                                                                                                                    |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ![](https://camo.githubusercontent.com/a0b140cbe7b198d62804d91996e3c09c6803cfbd484c61974af51892481f840e/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d437269746963616c2d643130623062) | A **directly** exploitable security vulnerability that leads to stolen/lost/locked/compromised assets or catastrophic denial of service.                                                                                                       |
| ![](https://camo.githubusercontent.com/77229c2f86cd6aaaaaeb3879eac44712b0e095147962e9058e455d0094f500d3/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d486967682d726564)               | A security vulnerability or bug that can affect the correct functioning of the system, lead to incorrect states or denial of service. It may not be directly exploitable or may require certain, external conditions in order to be exploited. |
| ![](https://camo.githubusercontent.com/d2cf6c2836b2143aeeb65c08b9c5aa1eb34a6fb8ad6fc55ba4345c467b64378a/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d656469756d2d6f72616e6765)     | Assets not at direct risk, but the function of the protocol or its availability could be impacted, or leak value with a hypothetical attack path with stated assumptions, but external requirements.                                           |
| ![](https://camo.githubusercontent.com/d42acfb8cb8228c156f34cb1fab83f431bf1fbebc102d922950f945b45e05587/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d696e6f722d79656c6c6f77)       | A violation of common best practices or incorrect usage of primitives, which may not currently have a major impact on security, but may do so in the future or introduce inefficiencies.                                                       |

# Scope

The review focused on the commit [hash](https://github.com/USSDofficial/ussd-contracts/tree/f44c726371f3152634bcf0a3e630802e39dec49c) and primarily covered the following contracts:

- [ussd-contracts/contracts/USSD.sol](ussd-contracts/contracts/USSD.sol)
- [ussd-contracts/contracts/USSDRebalancer.sol](ussd-contracts/contracts/USSDRebalancer.sol)
- [ussd-contracts/contracts/interfaces/IStableOracle.sol](ussd-contracts/contracts/interfaces/IStableOracle.sol)
- [ussd-contracts/contracts/interfaces/IStaticOracle.sol](ussd-contracts/contracts/interfaces/IStaticOracle.sol)
- [ussd-contracts/contracts/interfaces/IUSSDRebalancer.sol](ussd-contracts/contracts/interfaces/IUSSDRebalancer.sol)
- [ussd-contracts/contracts/oracles/StableOracleDAI.sol](ussd-contracts/contracts/oracles/StableOracleDAI.sol)
- [ussd-contracts/contracts/oracles/StableOracleWBGL.sol](ussd-contracts/contracts/oracles/StableOracleWBGL.sol)
- [ussd-contracts/contracts/oracles/StableOracleWBTC.sol](ussd-contracts/contracts/oracles/StableOracleWBTC.sol)
- [ussd-contracts/contracts/oracles/StableOracleWETH.sol](ussd-contracts/contracts/oracles/StableOracleWETH.sol)

~ 662 NSLOC in 9 contracts

NSLOC stands for 'Normalized Source Code', which is one of the custom measurements security researchers use when evaluating the complexity of a codebase.

The steps below are taken to get the NSLOC count of a file

1.  For all functions, reduce any multiline function declarations to a single line.
2.  Remove all comments
3.  Remove all empty lines
4.  Count the remaining lines

# About Bauchibred

**Bauchibred** is an independent smart contract security researcher. Having found numerous security vulnerabilities in various protocols, he does his best to contribute to the blockchain ecosystem and its protocols by putting time and effort into security research & reviews. Check his previous work [here](https://github.com/bauchibred/audits) or reach out on Twitter [@bauchibred](https://twitter.com/bauchibred).

# About USSD

**The Autonomous Secure Dollar (USSD)** is an open-source protocol designed for creating an autonomous ERC20 stablecoin. It is pegged to the US dollar and has a multiple times collateralization ratio, ensuring long-term stability. The USSD protocol only consists of crypto assets and has no ties to any physical-world financial instruments such as banks or treasuries.
It has been designed to be censorship-resistant, ensuring that it can remain operational without interference.

- Website: http://ussd.ai/
- Twitter: https://twitter.com/USSD_official
- WhitePaper: https://github.com/USSDofficial/ussd-whitepaper/blob/main/whitepaper.pdf

# Summary

This report contains **1 critical** issue, **5 high severity** issues, **4 medium** severity issues and **6 minor** issues found by **Bauchibred** during the course of the time-boxed audit.

| #       | Title                                                                                                | Severity                                           | Status                                                  |
| ------- | ---------------------------------------------------------------------------------------------------- | -------------------------------------------------- | ------------------------------------------------------- |
| [C-01]  | USSD.sol: Missing Access Control in `mintRebalancer` and `burnRebalancer` Functions                  | ![](https://img.shields.io/badge/-Critical-d10b0b) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [H-01]  | Lack of Slippage Protection in `UniV3SwapInput()`                                                    | ![](https://img.shields.io/badge/-Critical-d10b0b) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [H-02]  | Missing Deadline Checks in USSD Contract                                                             | ![](https://img.shields.io/badge/-High-red)        | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [H-03]  | Inactive ethOracle in StableOracleDAI Contract                                                       | ![](https://img.shields.io/badge/-High-red)        | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [H-04]  | USSDRebalancer.sol: Price Manipulation Vulnerability                                                 | ![](https://img.shields.io/badge/-High-red)        | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [H-05]  | Incorrect Aggregator Set for WBTC Price in StableOracleWBTC.sol                                      | ![](https://img.shields.io/badge/-High-red)        | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-01]  | StableOracleWBTC use BTC/USD chainlink oracle to price WBTC which is problematic if WBTC depegs      | ![](https://img.shields.io/badge/-Medium-orange)   | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-02]  | Missing Check for Stale Data in StableOracle Contracts                                               | ![](https://img.shields.io/badge/-Medium-orange)   | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-03]  | Potential DOS / lack of acccess to oracle price due to unhandled chainlink revert in                 | ![](https://img.shields.io/badge/-Medium-orange)   | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-04]  | Incorrect Price Calculation in Stable Oracles (WETH, WBTC, DAI) when Aggregator Hits minAnswer       | ![](https://img.shields.io/badge/-Minor-yellow)    | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-01]  | Multiple Instances of Precision Loss in the USSDRebalancer Contract                                  | ![](https://img.shields.io/badge/-Minor-yellow)    | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [L-02]  | `abi.encodePacked` Allows Hash Collision                                                             | ![](https://img.shields.io/badge/-Minor-yellow)    | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [L-03]  | USSD.sol: `addCollateral() `lacks a duplicate collateral check                                       | ![](https://img.shields.io/badge/-Minor-yellow)    | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [L-04]  | USSD.sol: Incorrect Collateral Index in USSD Contract                                                | ![](https://img.shields.io/badge/-Minor-yellow)    | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [L-05]  | USSD.sol: Division before multiplication incurs unnecessary precision loss in collateral calculation | ![](https://img.shields.io/badge/-Minor-yellow)    | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [NC-01] | Limited Routing Options in Protocol is a Downside                                                    | ![](https://img.shields.io/badge/-Minor-yellow)    | ![](https://img.shields.io/badge/-Acknowledged-black)   |

# Findings

## [C-01] USSD.sol: Missing Access Control in `mintRebalancer` and `burnRebalancer` Functions

### Summary

The contract `USSD` is missing access control in the `mintRebalancer` and `burnRebalancer` functions, allowing any address to call and execute these functions. This poses a critical security vulnerability as it enables potential manipulation of the rebalancing process and system exploitation.

### Vulnerability Detail

The `mintRebalancer` and `burnRebalancer` functions in the `USSD` contract lack access control, which means that there are no restrictions on who can call and execute these functions. As a result, any address, including malicious actors, can exploit this vulnerability by invoking these functions at advantageous moments, potentially manipulating the rebalancing process to their advantage.

```solidity

function mintRebalancer(uint256 amount) public override {
    _mint(address(this), amount);
}

function burnRebalancer(uint256 amount) public override {
    _burn(address(this), amount);
}
```

As seen there are no access controls on both mintRebalancer() & burnRebalancer(), any one can call and can try to tweak this in their favor before for example before executing the collateralFactor() function since the `totalSupply()` has a direct impact on the return,one can call either of mintRebalancer() orburnRebalancer() to game the system

### Impact

Manipulation of the rebalancing process: Unauthorized addresses can call these functions, potentially manipulating the rebalancing mechanism to their advantage.

### Recommendation

To address this issue, it is recommended to implement access control in the `mintRebalancer` and `burnRebalancer` functions to restrict their execution to authorized addresses only. You can utilize the OpenZeppelin `AccessControlUpgradeable` contract, which is already imported and inherited by the `USSD` contract.

1. Define a new role constant for the rebalancer, e.g., `REBALANCER_ROLE`, in the `USSD` contract:

```solidity
bytes32 public constant REBALANCER_ROLE = keccak256("REBALANCER");
```

2. Grant the `REBALANCER_ROLE` to the rebalancer contract during initialization or separately:

```solidity
function initialize(string memory name, string memory symbol) public initializer {
    // existing code

    _setupRole(REBALANCER_ROLE, address(rebalancer));
}
```

3. Update the `mintRebalancer` and `burnRebalancer` functions to include the `onlyRole` modifier:

```solidity
function mintRebalancer(uint256 amount) public override onlyRole(REBALANCER_ROLE) {
    _mint(address(this), amount);
}

function burnRebalancer(uint256 amount) public override onlyRole(REBALANCER_ROLE) {
    _burn(address(this), amount);
}
```

By implementing access control, only addresses with the `REBALANCER_ROLE` will be able to call these functions, ensuring the stability and integrity of the rebalancing process.

Note: It is essential to properly configure and manage role assignments to trusted addresses within the system to maintain security and prevent unauthorized access.

## [H-01] Lack of Slippage Protection in `UniV3SwapInput()`

### Summary

The `UniV3SwapInput` function implements no slippage protection. This means that swaps executed through this function can be subject to significant slippage, resulting in potential financial losses.

### Vulnerability Detail

The vulnerability is present in the following code snippet:

```solidity
function UniV3SwapInput(
    bytes memory _path,
    uint256 _sellAmount
) public override onlyBalancer {
    IV3SwapRouter.ExactInputParams memory params = IV3SwapRouter.ExactInputParams({
        path: _path,
        recipient: address(this),
        amountIn: _sellAmount,
        amountOutMinimum: 0
    });
    uniRouter.exactInput(params);
}
```

In this code, the `UniV3SwapInput` function performs a swap using the `exactInput` function of the `uniRouter` instance. However, no minimum output amount (slippage tolerance) is specified (`amountOutMinimum` is set to `0`). This means that the swap can be executed even if the output amount is significantly lower than expected, resulting in potential financial losses for the contract and its users.

### Impact

The lack of slippage protection in the `UniV3SwapInput` function can lead to significant financial losses. Swaps executed through this function are vulnerable to high slippage, meaning that the actual output amount may differ greatly from the expected amount. This can result in the loss of funds or an unfavorable exchange rate for the contract and its users.

### Recommendation

Modify the `UniV3SwapInput` function as follows:

1. Specify Minimum Output Amount: Add a parameter to the function that allows the caller to specify a minimum output amount (slippage tolerance) for the swap. This ensures that the swap is executed only if the actual output amount meets the desired threshold, providing protection against high slippage.

2. Validate Minimum Output Amount: Implement checks to ensure that the specified minimum output amount is reasonable and within an acceptable range. This prevents users from setting extremely low values that may hinder the functionality of the contract or expose it to unnecessary risks.

## [H-02] Missing Deadline Checks in USSD Contract

### Summary

The USSD contract lacks proper deadline checks in the UniV3SwapInput function, which poses a potential security vulnerability. Without a deadline parameter, balancer can unknowingly perform unfavorable or malicious trades due to pending transactions being executed at a later point.

### Vulnerability Detail

The UniV3SwapInput function in the USSD contract uses the UniSwap V3 router to perform a swap operation.

```solidity
function UniV3SwapInput(
    bytes memory _path,
    uint256 _sellAmount
) public override onlyBalancer {
    IV3SwapRouter.ExactInputParams memory params = IV3SwapRouter.ExactInputParams({
        path: _path,
        recipient: address(this),
        //deadline: block.timestamp,
        amountIn: _sellAmount,
        amountOutMinimum: 0
    });
    uniRouter.exactInput(params);
}
```

However, it does not include a deadline parameter, which allows pending transactions to be executed even after an extended period of time. This omission can lead to two potential issues:

1. Unfavorable Trades:
   If a user submits a transaction with a low transaction fee that remains pending in the mempool for an extended period, the price of tokens involved in the swap can change significantly. When the transaction eventually becomes interesting for miners to include, the swap will be executed based on outdated token prices. As a result, the user unknowingly performs a trade that results in unfavorable token conversion rates.

2. Malicious Execution through MEV:
   In a scenario where a transaction remains pending in the mempool and the price of the token being swapped has increased significantly, a malicious actor, such as a Miner Extractable Value (MEV) bot, can exploit the situation. The outdated maximum slippage value allows the MEV bot to sandwich the user, resulting in substantial profits for the bot and significant losses for the user.

### Impact

The missing deadline checks in the USSD contract's UniV3SwapInput function can lead to potential financial losses for users. Users might unknowingly execute trades with unfavorable token conversion rates or fall victim to malicious MEV attacks.

Below is the affected code snippet:

[UniV3SwapInput()](https://github.com/sherlock-audit/2023-05-USSD/blob/6d7a9fdfb1f1ed838632c25b6e1b01748d0bafda/ussd-contracts/contracts/USSD.sol#L227-L240)

```solidity
function UniV3SwapInput(
    bytes memory _path,
    uint256 _sellAmount
) public override onlyBalancer {
    IV3SwapRouter.ExactInputParams memory params = IV3SwapRouter.ExactInputParams({
        path: _path,
        recipient: address(this),
        //deadline: block.timestamp,
        amountIn: _sellAmount,
        amountOutMinimum: 0
    });
    uniRouter.exactInput(params);
}
```

### Recommendation

Introduce a deadline parameter in the UniV3SwapInput function. This parameter should specify a timestamp by which the transaction must be included in a block; otherwise, it should expire. Including a deadline enables protection from unfavorable trades and reduces the risk of MEV attacks.

Note: The exact implementation of the deadline parameter will depend on the requirements and design of the USSD contract and should consider factors such as transaction confirmation times and acceptable trade execution windows.

## [H-03] Inactive ethOracle in StableOracleDAI Contract

### Summary

The contract `StableOracleDAI` contains a bug where the `ethOracle` address is set to the zero address (`0x0000000000000000000000000000000000000000`). This results in the inability to query the `ethOracle` for the WETH oracle price. As a consequence, the contract fails to retrieve accurate USD prices, leading to incorrect calculations.

### Vulnerability Detail

The vulnerable code snippet is located in the `StableOracleDAI` contract, specifically in the constructor where the `ethOracle` address is assigned:

```solidity
constructor() {
    // ...
    ethOracle = IStableOracle(0x0000000000000000000000000000000000000000); // TODO: WETH oracle price
}
```

As seen in the `StableOracleDAI` contract, the `ethOracle` address is assigned the value of the zero address in the constructor:

```solidity
ethOracle = IStableOracle(0x0000000000000000000000000000000000000000); // TODO: WETH oracle price
```

Due to this assignment, the `getPriceUSD()` function, which relies on the `ethOracle` for obtaining the WETH oracle price, always fails. The contract assumes that the `ethOracle` address will provide the necessary price data, but since it is set to the zero address, querying it results in an error and incorrect price calculations.
Take a look at [getPriceUSD():](https://github.com/sherlock-audit/2023-05-USSD/blob/6d7a9fdfb1f1ed838632c25b6e1b01748d0bafda/ussd-contracts/contracts/oracles/StableOracleDAI.sol#LL33C1-L53C6)

```solidity
   function getPriceUSD() external view override returns (uint256) {
        address[] memory pools = new address[](1);
        pools[0] = 0x60594a405d53811d3BC4766596EFD80fd545A270;
        uint256 DAIWethPrice = DAIEthOracle.quoteSpecificPoolsWithTimePeriod(
            1000000000000000000, // 1 Eth
            0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2, // WETH (base token)
            0x6B175474E89094C44Da98b954EedeAC495271d0F, // DAI (quote token)
            pools, // DAI/WETH pool uni v3
            600 // period
        );

    // this wouldn't work since ethOracle = 0x0, no?
        uint256 wethPriceUSD = ethOracle.getPriceUSD();

        // chainlink price data is 8 decimals for WETH/USD, so multiply by 10 decimals to get 18 decimal fractional
        //(uint80 roundID, int256 price, uint256 startedAt, uint256 timeStamp, uint80 answeredInRound) = priceFeedDAIETH.latestRoundData();
        (, int256 price, , , ) = priceFeedDAIETH.latestRoundData();

        return
            (wethPriceUSD * 1e18) /
            ((DAIWethPrice + uint256(price) * 1e10) / 2);
    }
```

### Impact

The bug affects the accuracy of price calculations in the `StableOracleDAI` contract. As the `getPriceUSD()` function attempts to query the inactive `ethOracle` for the WETH oracle price, it always fails, leading to incorrect USD price calculations for the contract. This issue undermines the reliability and functionality of the contract, potentially impacting any operations or applications that rely on accurate price data.

### Recommendation

To resolve this issue, it is recommended to update the `ethOracle` address with the correct address of an active oracle that provides the WETH oracle price. The correct address should be obtained from a reliable and trusted source, ensuring that it can be queried successfully to retrieve accurate price data. By updating the `ethOracle` address, the `getPriceUSD()` function will be able to retrieve the WETH oracle price correctly, resulting in accurate USD price calculations for the contract.

## [H-04] USSDRebalancer.sol: Price Manipulation Vulnerability

### Summary

The USSDRebalancer contract is vulnerable to price manipulation due to its reliance on the `slot0` function of the UniswapV3Pool contract for price estimation. This vulnerability can be exploited by an attacker to manipulate the price of the USSD token and perform malicious actions.

### Vulnerability Detail

The `getOwnValuation` function in the USSDRebalancer contract retrieves the current price of the USSD token by calling the `slot0` function of the `uniPool` Uniswap V3 pool contract. However, using `slot0` to determine the token price makes the contract susceptible to price manipulation attacks.

```solidity
    /// @dev get price estimation to DAI using pool address and uniswap price
function getOwnValuation() public view returns (uint256 price) {
  (uint160 sqrtPriceX96,,,,,,) = uniPool.slot0();
  if (uniPool.token0() == USSD) {
    price = uint(sqrtPriceX96)*(uint(sqrtPriceX96))/(1e6) >> (96 * 2);
  } else {
    price = uint(sqrtPriceX96)*(uint(sqrtPriceX96))*(1e18 /* 1e12 + 1e6 decimal representation */) >> (96 * 2);
    // flip the fraction
    price = (1e24 / price) / 1e12;
  }
}
```

### Impact

The vulnerability allows an attacker to manipulate the price of the USSD token, potentially leading to unauthorized trades, incorrect valuation calculations, and financial losses for users of the contract.

### Recommendation

To address the vulnerability and mitigate price manipulation risks, it is recommended to implement a time-weighted average price (TWAP) oracle instead of relying on the `slot0` function. The TWAP oracle provides a more robust and tamper-resistant price feed by averaging the token prices over a specific time period. By using a TWAP oracle, the USSDRebalancer contract can obtain a more accurate and resistant price estimation, reducing the risk of price manipulation.

Additionally, it is crucial to conduct a comprehensive security audit of the entire contract codebase to identify and address any other potential vulnerabilities or issues that may exist.

Note: The above recommendations provide general guidance for addressing the specific vulnerability mentioned in the bug report. It is essential to consult with security experts and conduct a thorough security assessment to ensure the overall security and reliability of the contract.

## [H-05] Incorrect Aggregator Set for WBTC Price in StableOracleWBTC.sol

### Summary

The contract `StableOracleWBTC` contains a bug where the aggregator interface is set incorrectly for retrieving the WBTC/USD price. The wrong aggregator address is used, resulting in incorrect prices being returned by the contract.

### Vulnerability Detail

In the `StableOracleWBTC` contract, the `priceFeed` aggregator interface is initialized with an incorrect address in the constructor:
[StableOracleWBTC.sol#contructor](https://github.com/sherlock-audit/2023-05-USSD/blob/6d7a9fdfb1f1ed838632c25b6e1b01748d0bafda/ussd-contracts/contracts/oracles/StableOracleWBTC.sol#L15C1-L19)

```solidity
    constructor() {
//@audit the below is the pricefeed for the ETH/USD and BTC/USD
        priceFeed = AggregatorV3Interface(
            0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419
        );
    }
```

The address `0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419` corresponds to the Chainlink ETH/USD price feed, not the BTC/USD price feed. As a result, when the `getPriceUSD()` function retrieves the latest price using `priceFeed.latestRoundData()`, it incorrectly retrieves the ETH/USD price instead of the expected WBTC/USD price.

NB: This seems to be probably a copy paste error as just a few lines above the contract you can see that the correct pricefeed address for BTC/USD is commented out
[At L8-11](https://github.com/sherlock-audit/2023-05-USSD/blob/6d7a9fdfb1f1ed838632c25b6e1b01748d0bafda/ussd-contracts/contracts/oracles/StableOracleWBTC.sol#L8-L11)

```solidity
/*
    wbtc 0x2260FAC5E5542a773Aa44fBCfeDf7C193bc2C599;
    chainlink btc/usd priceFeed 0xf4030086522a5beea4988f8ca5b36dbc97bee88c;
*/
```

### Impact

The bug impacts the accuracy of price calculations in the `StableOracleWBTC` contract. By setting the wrong aggregator address, the contract fetches incorrect prices, which can lead to erroneous calculations and inaccurate WBTC/USD conversion rates. This issue undermines the reliability and functionality of the contract, potentially affecting any operations or applications relying on accurate WBTC price data.

### Recommendation

To address this issue, it is recommended to update the `priceFeed` aggregator address with the correct address of the BTC/USD price feed. The accurate address for the BTC/USD price feed should be `0xf4030086522a5beea4988f8ca5b36dbc97bee88c`. By setting the correct aggregator address, the `getPriceUSD()` function will retrieve the WBTC/USD price accurately, ensuring correct price calculations in the contract.

## [M-01] StableOracleWBTC use BTC/USD chainlink oracle to price WBTC which is problematic if WBTC depegs

### Summary

The StableOracleWBTC contract utilizes a BTC/USD Chainlink oracle to determine the price of WBTC. However, this approach can lead to potential issues if WBTC were to depeg from BTC. In such a scenario, WBTC would no longer maintain an equivalent value to BTC. This can result in significant problems, including borrowing against a devalued asset and the accumulation of bad debt. Given that the protocol continues to value WBTC based on BTC/USD, the issuance of bad loans would persist, exacerbating the overall level of bad debt.

Important to note that this is like a 2 in 1 report as the same idea could work on the StableOracleWBGL contract too.

### Vulnerability Detail

The vulnerability lies in the reliance on a single BTC/USD Chainlink oracle to obtain the price of WBTC. If the bridge connecting WBTC to BTC becomes compromised and WBTC depegs, WBTC may depeg from BTC. Consequently, WBTC's value would no longer be equivalent to BTC, potentially rendering it worthless (hopefully this never happens). The use of the BTC/USD oracle to price WBTC poses risks to the protocol and its users.

The following code snippet represents the relevant section of the StableOracleWBTC contract responsible for retrieving the price of WBTC using the BTC/USD Chainlink oracle:

```solidity
contract StableOracleWBTC is IStableOracle {
    AggregatorV3Interface priceFeed;

    constructor() {
        priceFeed = AggregatorV3Interface(
            0x5f4eC3Df9cbd43714FE2740f5E3616155c5b8419

        );
    }

    function getPriceUSD() external view override returns (uint256) {
        (, int256 price, , , ) = priceFeed.latestRoundData();
        // chainlink price data is 8 decimals for WBTC/USD
        return uint256(price) * 1e10;
    }
}
```

NB: key to note that the above pricefeed is set to the wrong aggregator, the correct one is this: `0x2260FAC5E5542a773Aa44fBCfeDf7C193bc2C599`

### Impact

Should the WBTC bridge become compromised or WBTC depeg from BTC, the protocol would face severe consequences. The protocol would be burdened with a substantial amount of bad debt stemming from outstanding loans secured by WBTC. Additionally, due to the protocol's reliance on the BTC/USD oracle, the issuance of loans against WBTC would persist even if its value has significantly deteriorated. This would lead to an escalation in bad debt, negatively impacting the protocol's financial stability and overall performance.

### Recommendation

To mitigate the vulnerability mentioned above, it is strongly recommended to implement a double oracle setup for WBTC pricing. This setup would involve integrating both the BTC/USD Chainlink oracle and an additional on-chain liquidity-based oracle, such as UniV3 TWAP.

The double oracle setup serves two primary purposes. Firstly, it reduces the risk of price manipulation by relying on the Chainlink oracle, which ensures accurate pricing for WBTC. Secondly, incorporating an on-chain liquidity-based oracle acts as a safeguard against WBTC depegging. By monitoring the price derived from the liquidity-based oracle and comparing it to the Chainlink oracle's price, borrowing activities can be halted if the threshold deviation (e.g., 2% lower) is breached.

Adopting a double oracle setup enhances the protocol's stability and minimizes the risks associated with WBTC depegging. It ensures accurate valuation, reduces the accumulation of bad debt, and safeguards the protocol and its users

## [M-02] Missing Check for Stale Data in StableOracle Contracts

### Summary

The contracts `StableOracleDAI`, `StableOracleWETH` and `StableOracleWBTC` have a vulnerability where they lack a check for stale data when retrieving the latest round data from the Chainlink price feed. This can result in the usage of outdated or incorrect price information, potentially leading to inaccurate calculations or decisions based on the price data.

### Vulnerability Detail

The missing check for stale data can be found in the following lines of code:

Contracts that can be affected by this:

- `StableOracleDAI`
- `StableOracleDAI`
- `StableOracleWBTC`

Example code snippet from `StableOracleDAI`:

```solidity
(uint80 roundID, int256 price, uint256 startedAt, uint256 timeStamp, uint80 answeredInRound) = priceFeedDAIETH.latestRoundData();
```

### Impact

The impact of the vulnerability is as follows:

- Incorrect or outdated price data can be used, leading to inaccurate calculations or decisions.
- Financial losses or incorrect outcomes may occur due to relying on stale or incorrect price information.

### Recommendation

To mitigate this vulnerability, the following steps are recommended:

1. Retrieve the complete round data from the Chainlink price feed.
2. Add checks for stale data, comparing the `answeredInRound` value with the `roundID`.
3. Verify that the `startedAt` and `timeStamp` values are not zero, indicating a complete round.
4. Adjust the calculations to account for valid and non-stale data.
5. Implement these mitigation steps in the affected contracts (`StableOracleDAI` and `StableOracleWBTC`).

By implementing the recommended steps, the contracts can validate the freshness and reliability of the price data retrieved from the Chainlink price feed, reducing the risk of using stale or incorrect prices in calculations and operations.

## [M-03] Potential DOS / lack of acccess to oracle price due to unhandled chainlink revert in

### Summary

The contract `StableOracleDAI` and her sister contracts (`StableOracleWBTC` and `StableOracleWETH`) all contain a vulnerability in the `getPriceUSD()` function, where it fails to handle potential reverts when accessing the Chainlink oracle. This can lead to a denial of service if the Chainlink oracle denies access, resulting in apermanent denial of service of querying prices.

NB: This report would only focus on the `StableOracleDAI` contract, but this issue affects all three of them.

### Vulnerability Detail

The `getPriceUSD()` function utilizes Chainlink's `latestRoundData()` to fetch the latest price. However, it lacks proper error handling or fallback logic in case the Chainlink oracle denies access. As a result, if access to the Chainlink oracle is blocked, the function will revert, leading to a denial of service scenario. Currently, there is no mechanism in place to regulate contract access to the Chainlink oracle, leaving it vulnerable to denial of service attacks.

The vulnerable code snippet can be found in the `StableOracleDAI` contract, specifically in the `getPriceUSD()` function. Here is the relevant code snippet:

```solidity
function getPriceUSD() external view override returns (uint256) {
    // ...
    (, int256 price, , , ) = priceFeedDAIETH.latestRoundData();
    // ...
}
```

### Impact

The vulnerability in the `getPriceUSD()` function can result in a denial of service for the contract. If the Chainlink oracle denies access to the data feed, the function will revert, making it impossible to retrieve prices. This denial of service can have severe consequences, as it hampers the functionality and usability of the contract.

### Recommendation

To mitigate this vulnerability, it is recommended to implement proper error handling and fallback logic within the `getPriceUSD()` function. This can be achieved by using a try/catch block to handle potential reverts when accessing the Chainlink oracle. In the catch block, suitable fallback logic should be implemented to handle scenarios where access to the Chainlink oracle is denied.
NB: Implementing proper error handling and fallback mechanisms will enhance the robustness of the contract and prevent a denial of service in case of restricted access to the Chainlink oracle.

## [M-04] Incorrect Price Calculation in Stable Oracles (WETH, WBTC, DAI) when Aggregator Hits minAnswer

### Summary

The Stable Oracles (StableOracleDAI, StableOracleWBGL, StableOracleWETH) all have a bug that leads to incorrect price calculations.
This is due to the fact that Chainlink aggregators have a built in circuit breaker if the price of an asset goes outside of a predetermined price band. The result is that if an asset experiences a huge drop in value (i.e. LUNA crash) the price of the oracle will continue to return the minPrice instead of the actual price of the asset. This would allow user to continue borrowing with the asset but at the wrong price. This is exactly what happened to Venus on BSC when LUNA imploded.
Take the StableOracleDAI as an example, when the price aggregator used, `priceFeedDAIETH`, hits the `minAnswer` circuit breaker and goes lower, chainlink instead returns `minAnswer` instead.
This issue can result in inaccurate USD price values for DAI, potentially causing financial discrepancies and vulnerabilities within the system.

### Vulnerability Detail

Going to use the StableOracleDAI as main point of focus so as not to make it too bogus, but good to note that this also affects StableOracleWETH and StableOracleWBTC too
The StableOracleDAI contract uses the `priceFeedDAIETH` aggregator from the Chainlink library to obtain the price of DAI in USD. The `getPriceUSD()` function retrieves the latest round data from the aggregator, but there is no check for the `minAnswer` circuit breaker. This circuit breaker is a built-in mechanism in Chainlink aggregators that prevents the price from going outside a predefined range.

The vulnerable code snippet is as follows:

```solidity
    constructor() {
        priceFeedDAIETH = AggregatorV3Interface(
            0x773616E4d11A78F511299002da57A0a94577F1f4
        );
        DAIEthOracle = IStaticOracle(
            0x982152A6C7f732Ec7C9EA998dDD9Ebde00Dfa16e
        );
        ethOracle = IStableOracle(0x0000000000000000000000000000000000000000); // TODO: WETH oracle price
    }

    function getPriceUSD() external view override returns (uint256) {
        address[] memory pools = new address[](1);
        pools[0] = 0x60594a405d53811d3BC4766596EFD80fd545A270;
        uint256 DAIWethPrice = DAIEthOracle.quoteSpecificPoolsWithTimePeriod(
            1000000000000000000, // 1 Eth
            0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2, // WETH (base token)
            0x6B175474E89094C44Da98b954EedeAC495271d0F, // DAI (quote token)
            pools, // DAI/WETH pool uni v3
            600 // period
        );

        uint256 wethPriceUSD = ethOracle.getPriceUSD();

        // chainlink price data is 8 decimals for WETH/USD, so multiply by 10 decimals to get 18 decimal fractional
        //(uint80 roundID, int256 price, uint256 startedAt, uint256 timeStamp, uint80 answeredInRound) = priceFeedDAIETH.latestRoundData();
        (, int256 price, , , ) = priceFeedDAIETH.latestRoundData();

        return
            (wethPriceUSD * 1e18) /
            ((DAIWethPrice + uint256(price) * 1e10) / 2);
    }
```

Example:

Consider the case of StableOracleDAI contract, which utilizes the Chainlink aggregator `priceFeedDAIETH` to fetch the price of DAI in USD. Let's assume that DAI has a `minAnswer` value set at $0.1. Now, imagine a situation where the actual price of DAI plummets to $0.01 due to market conditions.

Despite the significant drop in value, the `priceFeedDAIETH` aggregator continues to report the price of DAI as $0.1. Consequently, users interacting with the StableOracleDAI contract may be misled into borrowing against DAI as if it were valued at $0.1 per token. This means that users can borrow a larger amount of DAI than they should be entitled to, as the reported price is artificially inflated by a factor of 10.

#### Key point on how even using an aggregator with multiple oracle sources does not stop this attack

It is important to note that Chainlink oracles are typically used as part of an OracleAggregator system, with the assumption that combining multiple oracles can help mitigate such scenarios. However, in this case, relying on additional oracles may not provide a safeguard. For instance, if the Chainlink oracle is used in conjunction with a UniswapV3Oracle that relies on a long TWAP (Time-Weighted Average Price), the vulnerability can still be exploited. When the TWAP approaches the `minAnswer` threshold during a downward price movement, both oracles may align their reported prices, rendering the third oracle irrelevant. Moreover, even if secondary oracles like Band are utilized, a malicious user could launch a distributed denial-of-service (DDoS) attack against the relayers to prevent price updates. Once the price data becomes stale, the Chainlink oracle would be the only remaining oracle, and its potentially incorrect price would be utilized, exacerbating the issue.

This example highlights the potential impact of the bug in the StableOracleDAI contract, as it allows users to borrow DAI at an inflated price, potentially leading to financial losses and exploitation.

### Impact

The incorrect price calculation caused by the bug can have several implications:
In the case of StableOracleDAI

- The reported USD price of DAI will be inaccurate if the aggregator hits the `minAnswer` circuit breaker.
- Users relying on the StableOracleDAI contract may make financial decisions based on incorrect price information, leading to potential losses or exploitation.
- The stability of the system may be compromised if other components rely on the accurate USD price of DAI.

And similar impacts on other Stable oracles, i.e WETH and WBTC

### Recommendation

To address the bug and ensure accurate price calculations, the StableOracleDAI contract should implement a check for the `minAnswer` circuit breaker. If the returned price falls below the `minAnswer` threshold, the contract should handle it appropriately. Here is a recommended code addition:

```solidity
(, int256 price, , , ) = priceFeedDAIETH.latestRoundData();
require(price >= minAnswer, "Price below minimum threshold");
```

By adding this check, the StableOracleDAI contract will verify that the price is within the expected range and handle it accordingly, preventing incorrect price calculations.

## [L-01] Multiple Instances of Precision Loss in the USSDRebalancer Contract

### Summary

The USSDRebalancer contract is designed to perform swaps in order to maintain a 1-to-1 balance between USSD and DAI tokens in the main USSD/DAI Uniswap V3 pool. However, there are instances in the contract where precision loss occurs due to division operations being performed before multiplication operations. This can lead to inaccuracies in the calculation of token amounts and may result in unintended behavior.

### Vulnerability Detail

The USSDRebalancer contract contains instances where precision loss occurs during calculations. These precision losses may impact the accuracy of rebalancing operations and could potentially result in incorrect token minting, swapping, or burning.

The precision loss occurs in the following code instances:

1. **rebalance()** function:

   - The division operation

```solidity
((DAIamount / 1e12 - USSDamount)/2) * 99 / 100
```

results in precision loss due to division before multiplication.

2. **BuyUSSDSellCollateral()** function:

The calculation

```solidity
IERC20Upgradeable(collateral[i].token).balanceOf(USSD) * 1e18 / (10**IERC20MetadataUpgradeable(collateral[i].token).decimals()) * collateral[i].oracle.getPriceUSD() / 1e18
```

introduces precision loss due to division before multiplication.

3. **SellUSSDBuyCollateral()** function:
   similar calculation to that of the `BuyUSSDSellCollateral()` function

### Impact

The precision loss may lead to inaccurate calculations, potentially causing the following issues:

- Insufficient token minting or burning, resulting in an imbalance between USSD and collateral tokens.
- Incorrect token swapping, leading to undesirable trade executions.
- Incorrect estimation of token prices, affecting rebalancing decisions.

Below are the relevant code snippets where precision loss occurs:

[BuyUSSDSellCollateral()](https://github.com/sherlock-audit/2023-05-USSD/blob/6d7a9fdfb1f1ed838632c25b6e1b01748d0bafda/ussd-contracts/contracts/USSDRebalancer.sol#L109-L140)
[SellUSSDBuyCollateral()](https://github.com/sherlock-audit/2023-05-USSD/blob/6d7a9fdfb1f1ed838632c25b6e1b01748d0bafda/ussd-contracts/contracts/USSDRebalancer.sol#L163-L205)
[rebalance()](https://github.com/sherlock-audit/2023-05-USSD/blob/6d7a9fdfb1f1ed838632c25b6e1b01748d0bafda/ussd-contracts/contracts/USSDRebalancer.sol#L92-L107)

```solidity

// rebalance function
function rebalance() override public {
  uint256 ownval = getOwnValuation();
  (uint256 USSDamount, uint256 DAIamount) = getSupplyProportion();
  if (ownval < 1e6 - threshold) {
    // peg-down recovery
    BuyUSSDSellCollateral((USSDamount - DAIamount / 1e12)/2);
  } else if (ownval > 1e6 + threshold) {
    // mint and buy collateral
    // ...
    IUSSD(USSD).mintRebalancer(((DAIamount / 1e12 - USSDamount)/2) * 99 / 100);
    // ...
  }
}

// BuyUSSDSellCollateral function
function BuyUSSDSellCollateral(uint256 amountToBuy) internal {
  // ...
  uint256 collateralval = IERC20Upgradeable(collateral[i].token).balanceOf(USSD) * 1e
```

### Recommendation

To address the precision loss issues, it is recommended to rearrange the order of operations in the affected code sections. All division operations should be performed at the end of the calculations, after all multiplication operations have been completed. This will help maintain the desired precision and accuracy in the token amount calculations.

## [L-02] `abi.encodePacked` Allows Hash Collision

### Summary

The contract utilizes the `abi.encodePacked` function in several places, and from the solidity documentation:
https://docs.soliditylang.org/en/v0.8.17/abi-spec.html?highlight=collisions#non-standard-packed-mode

> If you use `keccak256(abi.encodePacked(a, b))` and both a and b are dynamic types, it is easy to craft collisions in the hash value by moving parts of a into b and vice-versa. More specifically,` abi.encodePacked("a", "bc") == abi.encodePacked("ab", "c")`.

### Vulnerability Detail

The issue lies in the usage of the `abi.encodePacked` function to concatenate and hash multiple values together. This approach can lead to hash collisions, where different inputs produce the same hash output. In Solidity, hash collisions can have unintended consequences, such as unexpected behavior or security vulnerabilities.

The vulnerable code snippets where `abi.encodePacked` is used are as follows:

1. In the `BuyUSSDSellCollateral` function:

```solidity
if (uniPool.token0() == USSD) {
    IUSSD(USSD).UniV3SwapInput(bytes.concat(abi.encodePacked(uniPool.token1(), hex"0001f4", uniPool.token0())), DAItosell);
} else {
    IUSSD(USSD).UniV3SwapInput(bytes.concat(abi.encodePacked(uniPool.token0(), hex"0001f4", uniPool.token1())), DAItosell);
}
```

2. In the `SellUSSDBuyCollateral` function:

```solidity
if (uniPool.token0() == USSD) {
    IUSSD(USSD).UniV3SwapInput(bytes.concat(abi.encodePacked(uniPool.token0(), hex"0001f4", uniPool.token1())), amount);
    daibought = IERC20Upgradeable(baseAsset).balanceOf(USSD) - daibought; // would revert if not bought
} else {
    IUSSD(USSD).UniV3SwapInput(bytes.concat(abi.encodePacked(uniPool.token1(), hex"0001f4", uniPool.token0())), amount);
    daibought = IERC20Upgradeable(baseAsset).balanceOf(USSD) - daibought; // would revert if not bought
}
```

As seen in both snippets, `abi.encodePacked` is used to concatenate the values `uniPool.token1()`, `hex"0001f4"`, and `uniPool.token0()`. The resulting byte array is then passed as input to the `UniV3SwapInput`

### Impact

The vulnerability can have various impacts depending on the specific usage and context within the contract. However, potential consequences may include:

1. Unpredictable behavior since hash collisions can lead to unexpected outcomes and behavior, making it difficult to reason about the contract's execution flow.

2. Inaccurate execution since in this case `UniV3SwapInput` could be passed the wrong data if the hash collides

The affected code snippet is shown below:

- [SellUSSDBuyCollateral()](https://github.com/sherlock-audit/2023-05-USSD/blob/6d7a9fdfb1f1ed838632c25b6e1b01748d0bafda/ussd-contracts/contracts/USSDRebalancer.sol#L163-L175)

```solidity
    function SellUSSDBuyCollateral() internal {
      uint256 amount = IUSSD(USSD).balanceOf(USSD);
      // sell for DAI then swap by DAI routes
      uint256 daibought = 0;
      if (uniPool.token0() == USSD) {
        daibought = IERC20Upgradeable(baseAsset).balanceOf(USSD);
        IUSSD(USSD).UniV3SwapInput(bytes.concat(abi.encodePacked(uniPool.token0(), hex"0001f4", uniPool.token1())), amount);
        daibought = IERC20Upgradeable(baseAsset).balanceOf(USSD) - daibought; // would revert if not bought
      } else {
        daibought = IERC20Upgradeable(baseAsset).balanceOf(USSD);
        IUSSD(USSD).UniV3SwapInput(bytes.concat(abi.encodePacked(uniPool.token1(), hex"0001f4", uniPool.token0())), amount);
        daibought = IERC20Upgradeable(baseAsset).balanceOf(USSD) - daibought; // would revert if not bought
      }
      ***//omitted for brevity
      }
```

- [BuyUSSDSellCollateral()](https://github.com/sherlock-audit/2023-05-USSD/blob/6d7a9fdfb1f1ed838632c25b6e1b01748d0bafda/ussd-contracts/contracts/USSDRebalancer.sol#L109-L158)

```
    function BuyUSSDSellCollateral(uint256 amountToBuy) internal {
 ***//omitted for brevity

      if (DAItosell > 0) {
        if (uniPool.token0() == USSD) {
            IUSSD(USSD).UniV3SwapInput(bytes.concat(abi.encodePacked(uniPool.token1(), hex"0001f4", uniPool.token0())), DAItosell);
        } else {
            IUSSD(USSD).UniV3SwapInput(bytes.concat(abi.encodePacked(uniPool.token0(), hex"0001f4", uniPool.token1())), DAItosell);
        }
      }

```

### Recommendation

Replace the usage of abi.encodePacked with a safer encoding method to prevent hash collisions.

## [L-03] USSD.sol: `addCollateral() `lacks a duplicate collateral check

### Summary

The `addCollateral()` lacks a check to ensure that the addition of duplicate collaterals does not go through when `index > = collateral.length` This issue can lead to undesired behavior and manipulation of the collateral system.

### Vulnerability Detail

The `addCollateral` function allows the addition of new collaterals to the `collateral` array when `index > = collateral.length` without checking if a collateral with the same address /infoalready exists. This could result in duplicate collateral entries, potentially leading to incorrect behavior or manipulation of the collateral system.

Take a look at [addCollateral():](https://github.com/sherlock-audit/2023-05-USSD/blob/6d7a9fdfb1f1ed838632c25b6e1b01748d0bafda/ussd-contracts/contracts/USSD.sol#LL84C1-L108C6)

```solidity
    function addCollateral(
        address _address,
        address _oracle,
        bool _mint,
        bool _redeem,
        uint256[] calldata _ratios,
        bytes memory _pathbuy,
        bytes memory _pathsell,
        uint256 index
    ) public onlyControl {
        CollateralInfo memory newCollateral = CollateralInfo({
            token: _address,
            mint: _mint,
            redeem: _redeem,
            oracle: IStableOracle(_oracle),
            pathbuy: _pathbuy,
            pathsell: _pathsell,
            ratios: _ratios
        });
        if (index < collateral.length) {
            collateral[index] = newCollateral; // for editing
        } else {
            collateral.push(newCollateral); // for adding new collateral
        }
    }
```

As seen if the index provided is more than collateral.length the said collateral is pushed into the array without checking if it already exists

### Impact

Duplicated collaterals would lead to inconsistent behavior within the collateral management system, affecting the stability and reliability of the contract.

### Recommendation

Implement duplicate collateral checks: Before adding a new collateral in the `addCollateral` function, iterate over the `collateral` array to verify if a collateral with the same address already exists. If a duplicate collateral is found, appropriate action should be taken, such as updating the existing collateral instead of adding a new one.

## [L-04] USSD.sol: Incorrect Collateral Index in USSD Contract

### Summary

The `getCollateralIndex` function in the USSD contract returns an incorrect index when a collateral token is not found in the `collateral` array. The function always returns 0 as the index, leading to the misconception that the collateral is indexed at 0. This can potentially mislead users and the team, causing confusion and misinterpretation of the collateral management system.

### Vulnerability Detail

The `getCollateralIndex` function is responsible for finding the index of a given collateral token within the `collateral` array. However, if the collateral is not found, the function mistakenly returns 0 as the index. This behavior can have implications in other functionalities that rely on this index. For example, the `calculateMint` function uses this index to calculate the amount of stablecoin a user receives for a given amount of asset. If the index is incorrectly assumed to be 0, the calculation will be based on the wrong collateral, potentially resulting in incorrect amounts and loss of trust in the protocol.

Take a look at [getCollateralIndex():](https://github.com/sherlock-audit/2023-05-USSD/blob/6d7a9fdfb1f1ed838632c25b6e1b01748d0bafda/ussd-contracts/contracts/USSD.sol#L125-L133)

```solidity
function getCollateralIndex(address _token) public view returns (uint256 index) {
    for (index = 0; index < collateral.length; index++) {
        if (collateral[index].token == _token) {
            return index;
        }
    }
}
```

Below is [calculateMint()](https://github.com/sherlock-audit/2023-05-USSD/blob/6d7a9fdfb1f1ed838632c25b6e1b01748d0bafda/ussd-contracts/contracts/USSD.sol#L169-L173)

```solidity
    /// @dev Return how much STABLECOIN does user receive for AMOUNT of asset
    function calculateMint(address _token, uint256 _amount) public view returns (uint256 stableCoinAmount) {
        uint256 assetPrice = collateral[getCollateralIndex(_token)].oracle.getPriceUSD();
        return (((assetPrice * _amount) / 1e18) * (10 ** decimals())) / (10 ** IERC20MetadataUpgradeable(_token).decimals());
    }
```

### Impact

The incorrect collateral index can lead to misleading information and incorrect calculations. Users relying on the `calculateMint` function may receive inaccurate amounts of stablecoin for their assets, resulting in what users might think are financial losses. Moreover, the incorrect calculations can undermine trust in the protocol and its collateral management system.

### Recommendation

Return a custom number when the collateral is not found, i.e modify the `getCollateralIndex` function to return a unique number, such as `9999`, when the collateral is not found in the `collateral` array. This will indicate that the collateral is not present and the `calculateMint` function should check for this custom number and revert with a meaningful error message.

```solidity
function getCollateralIndex(address _token) public view returns (int256 index) {
    for (uint256 i = 0; i < collateral.length; i++) {
        if (collateral[i].token == _token) {
            return int256(i);
        }
    }
    return 9999; // Custom number indicating collateral not found
}
```

## [L-05] USSD.sol: Division before multiplication incurs unnecessary precision loss in collateral calculation

### Summary

The USSD contract could possibly result in a precision loss during the calculation of collateral tokens. This precision loss can lead to users receiving a lesser amount of tokens than they deserve when minting. The bug is present in the `calculateMint()` function of the contract.

### Vulnerability Detail

The `calculateMint()` function is responsible for determining the amount of stablecoin a user receives for a given amount of collateral token. However, the current implementation suffers from a precision loss issue. In Solidity, performing division before multiplication can lead to loss of precision due to integer division truncation. This means that the fractional part of the result is truncated, leading to inaccurate calculations.

Take a look at the [calculateMint()](https://github.com/sherlock-audit/2023-05-USSD/blob/6d7a9fdfb1f1ed838632c25b6e1b01748d0bafda/ussd-contracts/contracts/USSD.sol#LL169C1-L173C6) function

```solidity
function calculateMint(address _token, uint256 _amount) public view returns (uint256 stableCoinAmount) {
    uint256 assetPrice = collateral[getCollateralIndex(_token)].oracle.getPriceUSD();
    return (((assetPrice * _amount) / 1e18) * (10 ** decimals())) / (10 ** IERC20MetadataUpgradeable(_token).decimals());
}
```

The incorrect sequence of operations `(assetPrice * _amount) / 1e18)` leads to the precision loss issue. It is crucial to perform multiplication before division to maintain precision in Solidity.

### Impact

The precision loss bug in the collateral calculation can result in users receiving a lesser amount of tokens than they deserve when minting. This issue can lead to financial losses for users and a lack of trust in the collateral management system of the protocol.

### Recommendation

Modify the `calculateMint()` function to perform multiplication before division. This can be achieved by rearranging the calculation as follows:

```solidity
function calculateMint(address _token, uint256 _amount) public view returns (uint256 stableCoinAmount) {
    uint256 assetPrice = collateral[getCollateralIndex(_token)].oracle.getPriceUSD();
    stableCoinAmount = (assetPrice * _amount * (10 ** decimals())) / (1e18 * (10 ** IERC20MetadataUpgradeable(_token).decimals()));
    return stableCoinAmount;
}
```

By performing multiplication before division, the precision loss issue can be avoided, ensuring accurate token minting.

## [NC-01] Limited Routing Options in Protocol is a Downside

### Summary

Though this is a problem of the whole protocol, let's just use the USSD contract as an explanation case, the USSD contract is designed to be an autonomous on-chain stablecoin, but it suffers from a limitation in its routing options. The contract restricts all swap operations to a single Uniswap router, which hinders the decentralization efforts of the protocol. Additionally, this limitation poses a risk as the selected Uniswap router may not provide optimal liquidity for a specific token at a specific time compared to another router. Utilizing a different aggregator like Paraswap or allowing multiple routers could potentially offer better liquidity and improve the routing efficiency.

### Vulnerability Detail

The USSD contract enforces the use of a single Uniswap router for all swap operations:

Take the below just for example

```solidity
function UniV3SwapInput(
    bytes memory _path,
    uint256 _sellAmount
) public override onlyBalancer {
    IV3SwapRouter.ExactInputParams memory params = IV3SwapRouter
        .ExactInputParams({
            path: _path,
            recipient: address(this),
            //deadline: block.timestamp,
            amountIn: _sellAmount,
            amountOutMinimum: 0
        });
    uniRouter.exactInput(params);
}
```

As a result, all swaps are performed exclusively through the specified Uniswap router.

### Impact

The limited routing options in the USSD contract can have the following impact:

1. Centralization Risk: By enforcing the use of a single Uniswap router, the contract deviates from the protocol's objective of decentralization. Restricting swap operations to a single router contradicts the principles of decentralized finance (DeFi).

2. Limited Liquidity Access: The selected Uniswap router may not have optimal liquidity for all tokens. This limitation poses a risk to users as they may encounter challenges in finding sufficient liquidity sources for certain tokens, potentially resulting in suboptimal swap rates and increased slippage.

### Recommendation

To address the limited routing options and promote decentralization, it is recommended to enhance the USSD contract's functionality by considering the following recommendations:

1. Multiple Routers: Allow users to select from multiple routers for swap operations. By incorporating support for different routers, such as Uniswap V2, Sushiswap, and other decentralized exchanges, users gain access to a wider range of liquidity sources, improving efficiency and reducing slippage.

2. Aggregator Integration: Consider integrating aggregator protocols like Paraswap to enhance the routing capabilities. Aggregators can leverage multiple liquidity sources and routing strategies to provide users with improved swap rates and reduced slippage.

3. Dynamic Routing Logic: Implement dynamic routing logic that intelligently selects the most optimal router or aggregator based on factors such as liquidity availability, token pair popularity, and historical trade data. This ensures efficient and cost-effective token swaps for users.
