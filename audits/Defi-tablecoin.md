# Introduction

A time-boxed security review of the **Foundry Defi Stablecoin** project which was done by **Bauchibred** with a focus on the security aspects of the application's smart contracts implementation.

# Table of Contents

- [Disclaimer](#disclaimer)
- [Severity classification](#Severity-classification)
- [Scope](#scope)
- [About Bauchibred](#About-Bauchibred)
- [About Foundry Defi Stablecoin](#About-Foundry-Defi-Stablecoin)
- [Summary](#summary)
- [Findings](#findings)

# Disclaimer

A smart contract security review can never verify the complete absence of vulnerabilities. This is a time, resource and expertise bound effort where I try to find as many vulnerabilities as possible. I can not guarantee 100% security after the review or even if the review will find any problems with your smart contracts. Subsequent security reviews, bug bounty programs and on-chain monitoring are strongly recommended.

Additionally, this report was prepared for [this competition](https://www.codehawks.com/contests/cljx3b9390009liqwuedkn0m0). The competition spanned **12** days, from **_July 24th, 2023_** to **_August 5th, 2023_**, and was hosted on [CodeHawks](https://www.codehawks.com). Given the public nature of the contest, an appointed judge, aided by a panel of other judges and security researchers including myself, had the discretion to determine the final severity of each finding. Due to the inherently subjective nature of such judgments, the final severity assigned might differ from my own classifications. For clarity on the judge's final decisions regarding severity, please refer to [this link](https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/issues).

# Severity classification

| Severity                                                                                                                                                                                            | Description                                                                                                                                                                                                                                    |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ![](https://camo.githubusercontent.com/a0b140cbe7b198d62804d91996e3c09c6803cfbd484c61974af51892481f840e/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d437269746963616c2d643130623062) | A **directly** exploitable security vulnerability that leads to stolen/lost/locked/compromised assets or catastrophic denial of service.                                                                                                       |
| ![](https://camo.githubusercontent.com/77229c2f86cd6aaaaaeb3879eac44712b0e095147962e9058e455d0094f500d3/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d486967682d726564)               | A security vulnerability or bug that can affect the correct functioning of the system, lead to incorrect states or denial of service. It may not be directly exploitable or may require certain, external conditions in order to be exploited. |
| ![](https://camo.githubusercontent.com/d2cf6c2836b2143aeeb65c08b9c5aa1eb34a6fb8ad6fc55ba4345c467b64378a/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d656469756d2d6f72616e6765)     | Assets not at direct risk, but the function of the protocol or its availability could be impacted, or leak value with a hypothetical attack path with stated assumptions, but external requirements.                                           |
| ![](https://camo.githubusercontent.com/d42acfb8cb8228c156f34cb1fab83f431bf1fbebc102d922950f945b45e05587/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d696e6f722d79656c6c6f77)       | A violation of common best practices or incorrect usage of primitives, which may not currently have a major impact on security, but may do so in the future or introduce inefficiencies.                                                       |

# Scope

The review focused on the following commit [hash](https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/commit/c573394cf3f21f73ef974388138193609f432c7f) and was focused on the following scope:

- [src/DSCEngine.sol](https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/main/src/DSCEngine.sol)
- [src/DecentralizedStableCoin.sol](https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/main/src/DecentralizedStableCoin.sol)
- [src/libraries/OracleLib.sol](https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/main/src/libraries/OracleLib.sol)

~ 236 NSLOC in Y contracts

NSLOC stands for 'Normalized Source Code', which is one of the custom measurements security researchers use when evaluating the complexity of a codebase.

The steps below are taken to get the NSLOC count of a file

1.  For all functions, reduce any multiline function declarations to a single line.
2.  Remove all comments
3.  Remove all empty lines
4.  Count the remaining lines

# About Bauchibred

**Bauchibred** is an independent smart contract security researcher. Having found numerous security vulnerabilities in various protocols, he does his best to contribute to the blockchain ecosystem and its protocols by putting time and effort into security research & reviews. Check his previous work [here](https://github.com/bauchibred/audits) or reach out on Twitter [@bauchibred](https://twitter.com/bauchibred).

# About Foundry Defi Stablecoin

This project is meant to be a stablecoin where users can deposit WETH and WBTC in exchange for a token that will be pegged to the USD. The system is meant to be such that someone could fork this codebase, swap out WETH & WBTC for any basket of assets they like, and the code would work the same.

# Summary

This report contains **1 high severity** issue, **3 medium** severity issues and **4 minor** issues found by **Bauchibred** during the course of the time-boxed audit.

| #       | Title                                                                                                                            | Severity                                         | Status                                                  |
| ------- | -------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ | ------------------------------------------------------- |
| [H-01]  | DSCEngine.sol hardcodes `ADDITIONAL_FEED_PRECISION` which would lead to overvaluation of user's collateral value for some tokens | ![](https://img.shields.io/badge/-High-d10b0b)   | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-01]  | OracleLib does not implement a check for the sequencer uptime which could lead to unfair liquidations                            | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-02]  | Fee on transfer tokens will not behave as expected                                                                               | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [M-03]  | Hardcoding timeout to 3 hours doesn't protect DSC from using stale data                                                          | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-01]  | Pragma isn't specified correctly which can lead to nonfunction/damaged contract when deployed on Arbitrum                        | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [L-02]  | Incorrect collateral value calculation for users, if any of the collateral's chainlink feed aggregator hits minAnswer            | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [L-03]  | Price access could get DOS'd since queries to chainlink are not done in a try/catch incase of reverts                            | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [NC-01] | The `_amount <= 0` check in `DecentralizedStableCoin::burn()&mint` is a confusing implementation and should be rethought         | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Resolved-brightgreen) |

# Findings

## [H-01] DSCEngine.sol hardcodes `ADDITIONAL_FEED_PRECISION` which would lead to overvaluation of user's collateral value for some tokens

### Summary

`ADDITIONAL_FEED_PRECISION` is being hardcoded to `10` in DSCEngine.sol.

This is probably implemented since initially the primary tokens would be BTC, ETH, but from the public discord chat the contest's sponsor (_Patrick Collins in this case_) has stated the below as reply to a question on what type of tokens would be used:

> weth & wbtc are the intended tokens, but any token with a USD chainlink price feed can work

Further discussions in the public chat suggests that the stablecoin is now intended to work with any token that has a USD feed.
Now the `DSCEngine.sol` contract assumes that all chainlink USD feeds return price in 8 decimals which is why the `ADDITIONAL_FEED_PRECISION` is being hardcoded to `10 so as to normalise the price, but this is not always the case.

For example the AMPL/USD feed is denoted with 18 decimals.
As can be seen [here](https://data.chain.link/ethereum/mainnet/crypto-usd/ampl-usd)

Now do note that the `ADDITIONAL_FEED_PRECISION` is being used in 2 instances in to normalise price, i.e in the [getTokenAmountFromUsd()](https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/DSCEngine.sol#L340-L348) and [getUsdValue()](https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/DSCEngine.sol#L361-L367) functions, as can be seen below:

```solidity
    function getTokenAmountFromUsd(address token, uint256 usdAmountInWei) public view returns (uint256) {
        // price of ETH (token)
        // $/ETH ETH ??
        // $2000 / ETH. $1000 = 0.5 ETH
        AggregatorV3Interface priceFeed = AggregatorV3Interface(s_priceFeeds[token]);
        (, int256 price,,,) = priceFeed.staleCheckLatestRoundData();
        // ($10e18 * 1e18) / ($2000e8 * 1e10)
        return (usdAmountInWei * PRECISION) / (uint256(price) * ADDITIONAL_FEED_PRECISION);
    }
...
    function getUsdValue(address token, uint256 amount) public view returns (uint256) {
        AggregatorV3Interface priceFeed = AggregatorV3Interface(s_priceFeeds[token]);
        (, int256 price,,,) = priceFeed.staleCheckLatestRoundData();
        // 1 ETH = $1000
        // The returned value from CL will be 1000 * 1e8
        return ((uint256(price) * ADDITIONAL_FEED_PRECISION) * amount) / PRECISION;
    }
```

Would be important to note where as the `getTokenAmountFromUsd()` function is not heavily used within the engine, aside being called externally it's still being used in the `liquidate()` function, whereas the `getUsdValue()` is massively implemented, for example while checking the health factor of an account, or while gettiing it's information, and multiple other instances within [DSCEngine.sol](https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/DSCEngine.sol)

```solidity
    function getHealthFactor(address user) external view returns (uint256) {
        return _healthFactor(user);
    }


    /*
     * Returns how close to liquidation a user is
     * If a user goes below 1, then they can get liquidated
     */
    function _healthFactor(address user) private view returns (uint256) {
        (uint256 totalDscMinted, uint256 collateralValueInUsd) = _getAccountInformation(user);
        return _calculateHealthFactor(totalDscMinted, collateralValueInUsd);
    }


        function _getAccountInformation(address user)
        private
        view
        returns (uint256 totalDscMinted, uint256 collateralValueInUsd)
    {
        totalDscMinted = s_DSCMinted[user];
        collateralValueInUsd = getAccountCollateralValue(user);
    }
    function getAccountCollateralValue(address user) public view returns (uint256 totalCollateralValueInUsd) {
        // loop through each collateral token, get the amount they have deposited, and map it to
        // the price, to get the USD value
        for (uint256 i = 0; i < s_collateralTokens.length; i++) {
            address token = s_collateralTokens[i];
            uint256 amount = s_collateralDeposited[user][token];
            totalCollateralValueInUsd += getUsdValue(token, amount);
        }
        return totalCollateralValueInUsd;
    }

```

This means that all the calculation of user's current state or their collateral value would be massively mis-valued, since using this report's case study the AMPL/USD returned price would be unnecasirly normalized by 10 decimals, in short the collateral would be misvalued with 10 decimals.

### Impact

Massive misvaluation of collateral's dollar value, difference of a whooping 10 decimals.

### Recommend Mitigation

`ADDITIONAL_FEED_PRECISION` should not be hardcoded, since hardcoding most of the time leads to lack of flexibility of a protocol, in this instance too I'd suggest that the decimals of each feed should be queried and then `ADDITIONAL_FEED_PRECISION` be set to `18-decimals`

## [M-01] OracleLib does not implement a check for the sequencer uptime which could lead to unfair liquidations

### Vulnerability Details

Issue exist since DSC is not going to be deployed only on mainnet, and in the case where it gets deployed on L2s, say arbitrum, if the Arbitrum sequencer is down and then comes back up, all Chainlink price updates will become available on Arbitrum within a very short time.
This eventually leaves users no time to react to the price changes which can lead to unfair liquidations.

Chainlink explains their Sequencer Uptime Feeds [here](https://docs.chain.link/data-feeds/l2-sequencer-feeds).

Quoting from the documentation:

> To help your applications identify when the sequencer is unavailable, you can use a data feed that tracks the last known status of the sequencer at a given point in time. This helps you prevent mass liquidations by providing a grace period to allow customers to react to such an event.

In practise Users are still able in to avoid liquidations by interacting with the Arbitrum delayed inbox via L1, but this is out of reach for most users.

Now a call to the [getAccountCollateralValue()](https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/DSCEngine.sol#L350) function would be easily affected as it leads to a call to query the chainlink price via `getUsdValue()`, and note that this function is used while getting a user's account information, which would be queried while checking for a users health factor(_via [healthfactor()](https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/DSCEngine.sol#L310)_), all this essentially lead to the check for user's health factor in this [line](https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/DSCEngine.sol#L236) of the [liquidate()](https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/DSCEngine.sol#L229-L262) function being unfairly passed, since user didn't have ample time to react to the sequencer coming back up.

### Impact

Users can get unfairly liquidated because they cannot react to price movements when the sequencer is down and when the sequencer comes back up, all price updates will immediately become available.

### Recommend Mitigation

The Chainlink documentation contains an example for how to check the sequencer status, seen [here](https://docs.chain.link/data-feeds/l2-sequencer-feeds)

There should also be a grace period when the sequencer comes back up for users to act on their collateral (increase collateral to avoid liquidation).

### Additional Note

This issue is not rooted on the fact that wrong price could be used while the sequencer is down(_A pretty popular vulnerabily in the web3 security space of "L2 sequencer could be down"_), but rather the fact that when the sequencer comes back up all the old prices will be processed at once and users have no time to react to price changes, which can lead to unfair liquidations, which is why the suggestion of a grace period was made.
Additionally the [Aave V3 technical paper](https://github.com/aave/aave-v3-core/blob/master/techpaper/Aave_V3_Technical_Paper.pdf)(section 4.6), has a detailed explanation of the damages this issue could cause.
Quoting the third part of the referenced section:

> For the Aave Protocol, and other systems using oracle price-feeds, this means that the feeds are not updated while the sequencer is down (as they use transactions after all). Essentially having a case where the entire price-development that occurred throughout the downtime is applied when the sequencer comes up. This uncertainty, and possibility for ”slow flash crashes”, together with the fact that queuing L2 transactions directly at L1 is out of reach for most normal users lead Aave V3 to introduce a grace-period on liquidations in these exact cases. As long as the position is not heavily undercollateralized (0.95 < HF< 1), it will have grace period starting at the time the sequencer comes up until it can be liquidated. If the position goes below 0.95 it can be liquidated entirely as on L1. Note, that this grace period is only activated if the sequencer has been down. During the grace period users will also not be allowed to borrow.

## [M-02] Fee on transfer tokens will not behave as expected

### Vulnerability Details

Integrating fee-on-transfer tokens would completely break DSC's internal accounting

Key to note that the current implementation of the DSC protocol doesn't work with fee-on-transfer as collateral tokens.
When a fee is charged on a transfer of tokens in Solidity, it is important to check the balance of the sender's account before and after the transfer to ensure that the fee has been correctly deducted from the sender's balance.

Take a look at [DSCEngine.sol#L144-L161](https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/DSCEngine.sol#L144-L161)

```solidity
    /*
     * @notice follows CEI
     * @param tokenCollateralAddress The address of the token to deposit as collateral
     * @param amountCollateral The amount of collateral to deposit
     */
    function depositCollateral(address tokenCollateralAddress, uint256 amountCollateral)
        public
        moreThanZero(amountCollateral)
        isAllowedToken(tokenCollateralAddress)
        nonReentrant
    {
        s_collateralDeposited[msg.sender][tokenCollateralAddress] += amountCollateral;
        emit CollateralDeposited(msg.sender, tokenCollateralAddress, amountCollateral);
        bool success = IERC20(tokenCollateralAddress).transferFrom(msg.sender, address(this), amountCollateral);
        if (!success) {
            revert DSCEngine__TransferFailed();
        }
    }
```

As seen no check is performed to really check if the amount of tokens received are what was sent note that since this check is not performed, it can result in an accounting error where the depositor's collateral value is overstated, leading to potential issues with record keeping, reporting, and reconciliation.

For example the [getAccountCollateralValue()](https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/DSCEngine.sol#L350) function

```solidity
    function getAccountCollateralValue(address user) public view returns (uint256 totalCollateralValueInUsd) {
        // loop through each collateral token, get the amount they have deposited, and map it to
        // the price, to get the USD value
        for (uint256 i = 0; i < s_collateralTokens.length; i++) {
            address token = s_collateralTokens[i];
            uint256 amount = s_collateralDeposited[user][token];
            totalCollateralValueInUsd += getUsdValue(token, amount);
        }
        return totalCollateralValueInUsd;
    }
```

When this is called would iterate over all the tokens added in the user's collateral array, but user's collateral value would be easily overstated since the amount of registeered collateral is not really what's been registered in the contract

### Impact

Break of internal accounting, in the case of the `depositCollateral` function of `DSCEngine.sol`, the contract directly registers the _amountCollateral_ provided and not the one received.

### Recommendation

Check the balances before and after any transfer with fees to ensure accurate accounting.

## [M-03] Hardcoding timeout to 3 hours doesn't protect DSC from using stale data

### Vulnerability Details

`OracleLib.sol`implements a hardcoded _TIMEOUT_ value, which is used in the `staleCheckLatestRoundData()` function to know if the returned price is stale or not, however in some instances this could be side stepped, see _Vulnerability Details_

We can see verification of the above in this [line](https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/libraries/OracleLib.sol#L19) that _TIMEOUT_ has been hardcoded to three hours.
Additionally it's been used [here](https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/libraries/OracleLib.sol#L21-L33):

```solidity
    function staleCheckLatestRoundData(AggregatorV3Interface priceFeed)
        public
        view
        returns (uint80, int256, uint256, uint256, uint80)
    {
        (uint80 roundId, int256 answer, uint256 startedAt, uint256 updatedAt, uint80 answeredInRound) =
            priceFeed.latestRoundData();


        uint256 secondsSince = block.timestamp - updatedAt;
        //@audit
        if (secondsSince > TIMEOUT) revert OracleLib__StalePrice();


        return (roundId, answer, startedAt, updatedAt, answeredInRound);
    }
```

Now the issue is that 3 hours is actually too long for some feeds, take ETH/USD or BTC/USD as an example (_using this as an example since DSC is primarily supposed to be baceked by BTC/ETH_), here is the [feed](https://data.chain.link/ethereum/mainnet/crypto-usd/eth-usd) for ETH?USD as seen it has a deviation threshold of 0.5% and also a heartbeat of 1 hour, which means that in the case the price does not make a 0.5% difference in any direction it's supposed to get updated within 1 hour, now would be key to note that stale price checks are done in order to make sure that the right price is integrated, in a case where an update does not get relayed from chainlink within the past hour for any of these feeds the price is stale.
Which means that all instances of querying prices would be affected, use the [getUsdValue()](https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/DSCEngine.sol#L361) as an example

```soldity
    function getUsdValue(address token, uint256 amount) public view returns (uint256) {
        AggregatorV3Interface priceFeed = AggregatorV3Interface(s_priceFeeds[token]);
        (, int256 price,,,) = priceFeed.staleCheckLatestRoundData();
        // 1 ETH = $1000
        // The returned value from CL will be 1000 * 1e8
        return ((uint256(price) * ADDITIONAL_FEED_PRECISION) * amount) / PRECISION;
    }
```

Would be important to note that this function is also used while checking the health factor of an account, or while gettiing it's information, and multiple other instances within [DSCEngine.sol](https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/DSCEngine.sol):

```solidity
    function getHealthFactor(address user) external view returns (uint256) {
        return _healthFactor(user);
    }


    /*
     * Returns how close to liquidation a user is
     * If a user goes below 1, then they can get liquidated
     */
    function _healthFactor(address user) private view returns (uint256) {
        (uint256 totalDscMinted, uint256 collateralValueInUsd) = _getAccountInformation(user);
        return _calculateHealthFactor(totalDscMinted, collateralValueInUsd);
    }


        function _getAccountInformation(address user)
        private
        view
        returns (uint256 totalDscMinted, uint256 collateralValueInUsd)
    {
        totalDscMinted = s_DSCMinted[user];
        collateralValueInUsd = getAccountCollateralValue(user);
    }
    function getAccountCollateralValue(address user) public view returns (uint256 totalCollateralValueInUsd) {
        // loop through each collateral token, get the amount they have deposited, and map it to
        // the price, to get the USD value
        for (uint256 i = 0; i < s_collateralTokens.length; i++) {
            address token = s_collateralTokens[i];
            uint256 amount = s_collateralDeposited[user][token];
            totalCollateralValueInUsd += getUsdValue(token, amount);
        }
        return totalCollateralValueInUsd;
    }

```

This easily means that in the case where a stale price is integrated, the calculation of user's current state in the internal `_healthFactor()` could be wrongly returned as above 1 and they can't get liquidated where as they are below the healthy level, leading to DSC allowing users hold overcollaterized tokens, alternativelythe current state could be returned as below 1 and users get unfairly liquidatred whereas their account is above the healthy level.

### Impact

DSC would work with outdated price data for ETH/USD and BTC/USD.

### Recommend Mitigation

Set a different _TIMEOUT_ value for different feeds, it should be shorter than 3 hours for the aforementioned feeds, i.e BTC, ETH.
Addittionally since it's being relayed in the public discord chat that any erc20 token that has a USD chainlink feed can be used for protocol then respective checks should be made for each and the timeout value should be set based on their heartbeats.

## [L-01] Pragma isn't specified correctly which can lead to nonfunction/damaged contract when deployed on Arbitrum

### Vulnerability Details

It's been relayed in the public discord that DSC would be deployed in EVM compatible chains, but arbitrum is not compatible with Solidity version 0.8.20, which has not been accounted for.

Floating pragma is used, allowing the contracts to be compiled with any 0.8.x compiler higher than the specified version. The problem with this is that Arbitrum is [NOT compatible](https://developer.arbitrum.io/solidity-support) with 0.8.20 and newer due to the introduction of a new opcode `PUSH0`. Contracts compiled with those versions will result in a nonfunctional or potentially damaged version that won't behave as expected. The default behavior of compiler would be to use the newest version which would mean by default it will be compiled with the 0.8.20 version which will produce broken code.

### Impact

Damaged or nonfunctional contracts when deployed on Arbitrum.

### Recommendation

Constrain pragma could be something as follows:

    pragma solidity >=0.8.18 <=0.8.19

## [L-02] Incorrect collateral value calculation for users, if any of the collateral's chainlink feed aggregator hits minAnswer

### Vulnerability Detail

Chainlink aggregators have a built in circuit breaker if the price of an asset goes outside of a predetermined price band. The result is that if an asset experiences a huge drop in value (i.e. LUNA crash) the price of the oracle will continue to return the minPrice instead of the actual price of the asset. This would allow user to continue integrating with protocol with wrong price.
NB: This is exactly what happened to Venus on BSC when LUNA imploded.

Take a look at [DSCEngine.sol#L350-L359](https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/DSCEngine.sol#L350-L359)

```solidity
    function getAccountCollateralValue(address user) public view returns (uint256 totalCollateralValueInUsd) {
        // loop through each collateral token, get the amount they have deposited, and map it to
        // the price, to get the USD value
        for (uint256 i = 0; i < s_collateralTokens.length; i++) {
            address token = s_collateralTokens[i];
            uint256 amount = s_collateralDeposited[user][token];
            totalCollateralValueInUsd += getUsdValue(token, amount);
        }
        return totalCollateralValueInUsd;
    }
```

This function is used to iterate through all the collateral tokens a user has and provide the summation of all their values in usd, now the function calls the [getUsdValue()](https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/DSCEngine.sol#L361-L367) function:

```solidity
    function getUsdValue(address token, uint256 amount) public view returns (uint256) {
        AggregatorV3Interface priceFeed = AggregatorV3Interface(s_priceFeeds[token]);
        (, int256 price,,,) = priceFeed.staleCheckLatestRoundData();
        // 1 ETH = $1000
        // The returned value from CL will be 1000 * 1e8
        return ((uint256(price) * ADDITIONAL_FEED_PRECISION) * amount) / PRECISION;
    }
```

This function just queries the price from chainlink and retrieves the latest round data from the aggregator, but there is no check for the `minAnswer` circuit breaker. This circuit breaker is a built-in mechanism in Chainlink aggregators that prevents the price from going outside a predefined range.

Example:

Consider fetching the price of any erc20 token in USD, say ABC.
Let's assume that ABC has a `minAnswer` value set at $0.1. Now, imagine a situation where the actual price of ABC plummets to $0.01 due to market conditions.

Despite the significant drop in value, the aggregator continues to report the price of ABC as $0.1. Consequently, the DSCEngine.sol contract would be misled into thinking ABC is collateral value would be somewhat inflated, in the case where user only has the ABC token as a collateral then the DSCEngine.sol contract would assume thier collateral value to be times 10 the value it really is

### Impact

In short wrong collateral value assumption for users

### Recommendation

Implement a check for the `minAnswer` circuit breaker. If the returned price falls below the `minAnswer` threshold, DSC should handle it appropriately.

## [L-03] Price access could get DOS'd since queries to chainlink are not done in a try/catch incase of reverts

### Vulnerability Details

Chainlink is heavily relied upon within DSC, infact it's the only oracle body being used, which exarcebates this issue.
Now it'd be key note that asides this [blog from openzeppelin](https://blog.openzeppelin.com/secure-smart-contract-guidelines-the-dangers-of-price-oracles/) mentioning that it is possible that Chainlink’s "multisigs immediately block access to price feeds at will". Oracles can also be taken down for maintenance/safety reasons, which is why it's a pretty popular practise to wrap chainlink queries in a try/catch, and if the call fails for whatever reason the fallback mechanism is there to sort things out and prevent a denial of service from occurring when trying to access the price feed.

Additionally note that the query to chainlink reverting means that all instance of getting price is inacessible and all actions attached to them, for example the call to get the health factor would no longer be accessible.
See the [getHealthFactor()](https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/DSCEngine.sol#L393-L395) function of
[DSCEngine.sol](https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/DSCEngine.sol):

```solidity
    function getHealthFactor(address user) external view returns (uint256) {
        return _healthFactor(user);
    }


    /*
     * Returns how close to liquidation a user is
     * If a user goes below 1, then they can get liquidated
     */
    function _healthFactor(address user) private view returns (uint256) {
        (uint256 totalDscMinted, uint256 collateralValueInUsd) = _getAccountInformation(user);
        return _calculateHealthFactor(totalDscMinted, collateralValueInUsd);
    }


        function _getAccountInformation(address user)
        private
        view
        returns (uint256 totalDscMinted, uint256 collateralValueInUsd)
    {
        totalDscMinted = s_DSCMinted[user];
        collateralValueInUsd = getAccountCollateralValue(user);
    }
    function getAccountCollateralValue(address user) public view returns (uint256 totalCollateralValueInUsd) {
        // loop through each collateral token, get the amount they have deposited, and map it to
        // the price, to get the USD value
        for (uint256 i = 0; i < s_collateralTokens.length; i++) {
            address token = s_collateralTokens[i];
            uint256 amount = s_collateralDeposited[user][token];
            totalCollateralValueInUsd += getUsdValue(token, amount);
        }
        return totalCollateralValueInUsd;
    }

```

This would be iacessible since the getUsdValue() function would revert the call.

### Recommend Mitigation

Use a try/catch block around the `latestRoundData()` calls. If these calls revert, the catch block should handle the failure accordingly. This can include a fallback mechanism, an alternative oracle call, or a contingency procedure to pause operations or any reasonable mechanism.

## [NC-01] The `_amount <= 0` check in `DecentralizedStableCoin::burn()&mint` is a confusing implementation and should be rethought

### Vulnerability Details

See these:

- https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/DecentralizedStableCoin.sol#L62
- https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/DecentralizedStableCoin.sol#L49

This check if not passed reverts with an error message `DecentralizedStableCoin__MustBeMoreThanZero()`, however the provided calue is a `uint `which hints that it could never be negative in the first place and the check should instead be if `_amount =  0`

### Impact

Incorrect context

### Recommend Mitigation

Fix this, if our thought process align.

### Additional Note

This [line](https://github.com/Cyfrin/2023-07-foundry-defi-stablecoin/blob/d1c5501aa79320ca0aeaa73f47f0dbc88c7b77e2/src/DSCEngine.sol#L210) has a subtle typo that leads to confusion while reading the natspec commenst, "you" should instead be "your". _submitting as an attached finding as it's to minuscle to create as a stand alone finding._
