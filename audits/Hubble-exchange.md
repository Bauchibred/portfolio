# Introduction

A time-boxed security review of the **Hubble Exchange** protocol which was done by **Bauchibred** with a focus on the security aspects of the application's smart contracts implementation.

# Table of Contents

- [Disclaimer](#disclaimer)
- [Severity classification](#Severity-classification)
- [Scope](#scope)
- [About Bauchibred](#About-Bauchibred)
- [About Hubble Exchange](#About-Hubble-Exchange)
- [Summary](#summary)
- [Findings](#findings)

# Disclaimer

A smart contract security review can never verify the complete absence of vulnerabilities. This is a time, resource and expertise bound effort where I try to find as many vulnerabilities as possible. I can not guarantee 100% security after the review or even if the review will find any problems with your smart contracts. Subsequent security reviews, bug bounty programs and on-chain monitoring are strongly recommended.

Additionally, this report was prepared for [this competition](https://audits.sherlock.xyz/contests/72). The competition spanned **12** days, from **_June 21, 2023_** to **_July 3, 2023_**, and was hosted on [Sherlock](https://www.sherlock.xyz/). Given the public nature of the contest, an appointed judge, aided by a panel of other judges and security researchers including myself, had the discretion to determine the final severity of each finding. Due to the inherently subjective nature of such judgments, the final severity assigned might differ from my own classifications. For clarity on the judge's final decisions regarding severity, please refer to [this link](https://github.com/sherlock-audit/2023-04-hubble-exchange-judging/issues).

# Severity classification

| Severity                                                                                                                                                                                            | Description                                                                                                                                                                                                                                    |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ![](https://camo.githubusercontent.com/a0b140cbe7b198d62804d91996e3c09c6803cfbd484c61974af51892481f840e/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d437269746963616c2d643130623062) | A **directly** exploitable security vulnerability that leads to stolen/lost/locked/compromised assets or catastrophic denial of service.                                                                                                       |
| ![](https://camo.githubusercontent.com/77229c2f86cd6aaaaaeb3879eac44712b0e095147962e9058e455d0094f500d3/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d486967682d726564)               | A security vulnerability or bug that can affect the correct functioning of the system, lead to incorrect states or denial of service. It may not be directly exploitable or may require certain, external conditions in order to be exploited. |
| ![](https://camo.githubusercontent.com/d2cf6c2836b2143aeeb65c08b9c5aa1eb34a6fb8ad6fc55ba4345c467b64378a/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d656469756d2d6f72616e6765)     | Assets not at direct risk, but the function of the protocol or its availability could be impacted, or leak value with a hypothetical attack path with stated assumptions, but external requirements.                                           |
| ![](https://camo.githubusercontent.com/d42acfb8cb8228c156f34cb1fab83f431bf1fbebc102d922950f945b45e05587/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f2d4d696e6f722d79656c6c6f77)       | A violation of common best practices or incorrect usage of primitives, which may not currently have a major impact on security, but may do so in the future or introduce inefficiencies.                                                       |

# Scope

The review focused on the commit [hash](https://github.com/hubble-exchange/hubble-protocol/tree/d89714101dd3494b132a3e3f9fed9aca4e19aef6) and primarily covered the following contracts:

- [hubble-protocol/contracts/AMM.sol](hubble-protocol/contracts/AMM.sol)
- [hubble-protocol/contracts/ClearingHouse.sol](hubble-protocol/contracts/ClearingHouse.sol)
- [hubble-protocol/contracts/GenesisTUP.sol](hubble-protocol/contracts/GenesisTUP.sol)
- [hubble-protocol/contracts/HGT.sol](hubble-protocol/contracts/HGT.sol)
- [hubble-protocol/contracts/HGTCore.sol](hubble-protocol/contracts/HGTCore.sol)
- [hubble-protocol/contracts/HGTRemote.sol](hubble-protocol/contracts/HGTRemote.sol)
- [hubble-protocol/contracts/InsuranceFund.sol](hubble-protocol/contracts/InsuranceFund.sol)
- [hubble-protocol/contracts/Interfaces.sol](hubble-protocol/contracts/Interfaces.sol)
- [hubble-protocol/contracts/MarginAccount.sol](hubble-protocol/contracts/MarginAccount.sol)
- [hubble-protocol/contracts/MarginAccountHelper.sol](hubble-protocol/contracts/MarginAccountHelper.sol)
- [hubble-protocol/contracts/MinimalForwarder.sol](hubble-protocol/contracts/MinimalForwarder.sol)
- [hubble-protocol/contracts/Oracle.sol](hubble-protocol/contracts/Oracle.sol)
- [hubble-protocol/contracts/VUSD.sol](hubble-protocol/contracts/VUSD.sol)
- [hubble-protocol/contracts/orderbooks/OrderBook.sol](hubble-protocol/contracts/orderbooks/OrderBook.sol)

~ 2000 NSLOC

NSLOC stands for 'Normalized Source Code', which is one of the custom measurements security researchers use when evaluating the complexity of a codebase.

The steps below are taken to get the NSLOC count of a file

1.  For all functions, reduce any multiline function declarations to a single line.
2.  Remove all comments
3.  Remove all empty lines
4.  Count the remaining lines

# About Bauchibred

**Bauchibred** is an independent smart contract security researcher. Having found numerous security vulnerabilities in various protocols, he does his best to contribute to the blockchain ecosystem and its protocols by putting time and effort into security research & reviews. Check his previous work [here](https://github.com/bauchibred/audits) or reach out on Twitter [@bauchibred](https://twitter.com/bauchibred).

# About Hubble Exchange

**Hubble Exchange** is a trustless and composable trading system for perpetuals. Engineered at the intersection of derivatives and application-specific VMs. Hubblenet is a perp-specific chain; customized to embed a fully Decentralized Limit Order Book and its Matching Engine.

Additionally, Hubble Exchange is the first Multi-collateral / Cross-Margin Perpetual Futures protocol on Avalanche.

- Website: https://hubble.exchange/

- Twitter: https://twitter.com/HubbleExchange

# Summary

This report contains **5 medium** severity issues and **2 minor** issues found by **Bauchibred** during the course of the time-boxed audit.

| #      | Title                                                                                                                    | Severity                                         | Status                                                  |
| ------ | ------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------ | ------------------------------------------------------- |
| [M-01] | Wrong assumption has been made that stablecoins never depeg                                                              | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-02] | No `minAnswer/maxAnswer` Circuit Breaker Checks while Querying Prices in Oracle.sol                                      | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Resolved-brightgreen) |
| [M-03] | An outrageously different price could be used in the case where all of the last hour data is negative in `getRoundData() | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/-Disputed-grey)        |
| [M-04] | Pricing on liquidations can be bogus                                                                                     | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/Disputed-grey)         |
| [M-05] | Share minting logic can be broken by an early depositor                                                                  | ![](https://img.shields.io/badge/-Medium-orange) | ![](https://img.shields.io/badge/Acknowledged-black)    |
| [L-01] | Liquidations will be frozen if a token's oracle goes down or chainlink reverts call for any reason                       | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |
| [L-02] | Potential DOS to MarginAccount::isLiquidatable() or any other function that calls this                                   | ![](https://img.shields.io/badge/-Minor-yellow)  | ![](https://img.shields.io/badge/-Acknowledged-black)   |

# Findings

## [M-01] Wrong assumption has been made that stablecoins never depeg

### Vulnerability Details

After extensive discussion with the sponsors, it's been realised that Hubble values stablecoins at $1 forever and assumes it (potentially _they_ in the case where more stablecoins are integrated in the future) never depegs, Do note that having a constant value for any stablecoin easily allows malicious usage of the protocol once a particular stable depegs, leaving the protocol to hold potential worthless tokens, in the event that the stablecoin depegs and never gets back to it's peg, even if it gets back to peg, while it was not pegged `1:1` malicious actors can easily game Hubble and make ill-gotten gains.

Take a look at [Oracle.sol#L16-L32](https://github.com/sherlock-audit/2023-04-hubble-exchange/blob/1f9a5ed0ca8f6004bbb7b099ecbb8ae796557849/hubble-protocol/contracts/Oracle.sol#L16-L33)

```solidity
    mapping(address => int256) public stablePrice;
    //...ommitted for brevity
            if (stablePrice[underlying] != 0) {
            return stablePrice[underlying];
        }
```

Note that the stablecoin most likely relied on within Hubble is USDC, being it's it's gas token, USDC has had multiple occurences of depegs, in one instance which happened around 4 motnhs ago (as at the time of writing this report) it depegged to a whooping 87 cents for a dollar, source [here](https://decrypt.co/123211/usdc-stablecoin-depegs-90-cents-circle-exposure-silicon-valley-bank) which means that if someone provided USDC to mint VUSD, they make an outrageous profit ~15%, i.e provide $100k worth of tokens to mint a$114,943 worth of VUSD, note that this USDC depeg, caused a storm in the market and even triggered the depegging of other s notable dollar-pegged stablecoins: DAI, USDD, and USDP, though not being a common occurence one inaccurate encounter of a serious depeg could severely deter protocol.

### Impact

In the case where the market price of a stablecoin drops the protocol still assumes it to be pegged which incentivises people to arbitraging and other malicious actions.

### Recommendation

Consistently fetch the real price feed of all assets, USDC/USD can be found [here](https://data.chain.link/ethereum/mainnet/stablecoins/usdc-usd)

## [M-02] No `minAnswer/maxAnswer` Circuit Breaker Checks while Querying Prices in Oracle.sol

### Vulnerability Details

The Oracle.sol contract, while currently applying a safety check (this can be side stepped, check my other submission ) to ensure returned prices are greater than zero, which is commendable, as it effectively mitigates the risk of using negative prices, there should be an implementation to ensure the returned prices are not at the extreme boundaries (`minAnswer` and `maxAnswer`).
Without such a mechanism, the contract could operate based on incorrect prices, which could lead to an over- or under-representation of the asset's value, potentially causing significant harm to the protocol.

Chainlink aggregators have a built in circuit breaker if the price of an asset goes outside of a predetermined price band. The result is that if an asset experiences a huge drop in value (i.e. LUNA crash) the price of the oracle will continue to return the minPrice instead of the actual price of the asset. This would allow user to continue borrowing with the asset but at the wrong price. This is exactly what happened to [Venus on BSC when LUNA imploded](https://rekt.news/venus-blizz-rekt/).
In its current form, the `getUnderlyingPrice()` function within the Oracle.sol contract retrieves the latest round data from Chainlink, if the asset's market price plummets below `minAnswer` or skyrockets above `maxAnswer`, the returned price will still be `minAnswer` or `maxAnswer`, respectively, rather than the actual market price. This could potentially lead to an exploitation scenario where the protocol interacts with the asset using incorrect price information.

Take a look at [Oracle.sol#L106-L123](https://github.com/sherlock-audit/2023-04-hubble-exchange/blob/1f9a5ed0ca8f6004bbb7b099ecbb8ae796557849/hubble-protocol/contracts/Oracle.sol#L106-L123):

```solidity

    function getLatestRoundData(AggregatorV3Interface _aggregator)
        internal
        view
        returns (
            uint80,
            uint256 finalPrice,
            uint256
        )
    {
        (uint80 round, int256 latestPrice, , uint256 latestTimestamp, ) = _aggregator.latestRoundData();
        finalPrice = uint256(latestPrice);
        if (latestPrice <= 0) {
            requireEnoughHistory(round);
            (round, finalPrice, latestTimestamp) = getRoundData(_aggregator, round - 1);
        }
        return (round, finalPrice, latestTimestamp);
    }
```

#### Illustration:

- Present price of TokenA is $10
- TokenA has a minimum price set at $1 on chainlink
- The actual price of TokenA dips to $0.10
- The aggregator continues to report $1 as the price.

Consequently, users can interact with protocol using TokenA as though it were still valued at $1, which is a tenfold overestimate of its real market value.

### Impact

The potential for misuse arises when the actual price of an asset drastically changes but the oracle continues to operate using the `minAnswer` or `maxAnswer` as the asset's price. In the case of it going under the `minAnswer` malicious actors obviously have the upperhand and could give their potential _going to zero_ worth tokens to protocol

### Recommendation

Since there is going to be a whitelist of tokens to be added, the minPrice/maxPrice could be checked and a revert could be made when this is returned by chainlink or a fallback oracle that does not have circuit breakers could be implemented in that case

## [M-03] An outrageously different price could be used in the case where all of the last hour data is negative in `getRoundData()`

### Vulnerability Detail

This check is included in the [getLatestRoundData()](https://github.com/sherlock-audit/2023-04-hubble-exchange/blob/1f9a5ed0ca8f6004bbb7b099ecbb8ae796557849/hubble-protocol/contracts/Oracle.sol#L106-L141) function:

```solidity
  function getLatestRoundData(AggregatorV3Interface _aggregator)
...
        if (latestPrice <= 0) {
            requireEnoughHistory(round);
            (round, finalPrice, latestTimestamp) = getRoundData(_aggregator, round - 1);
        }
...
}
```

Key to note that this function is used for the twap calculation of last hour, problem is that if all of last hour data is negative then the `last positive price` is used, this is obviously an issue as if all instance for the twap price of the last hour is `-ve,` a revert should instead be the best case, than potentially valuing the token at a very outrageous price, since it got worthless already

Take a look at the full [getLatestRoundData()](https://github.com/sherlock-audit/2023-04-hubble-exchange/blob/1f9a5ed0ca8f6004bbb7b099ecbb8ae796557849/hubble-protocol/contracts/Oracle.sol#L106-L141) function

```


    function getLatestRoundData(AggregatorV3Interface _aggregator)
        internal
        view
        returns (
            uint80,
            uint256 finalPrice,
            uint256
        )
    {
        (uint80 round, int256 latestPrice, , uint256 latestTimestamp, ) = _aggregator.latestRoundData();
        finalPrice = uint256(latestPrice);
        if (latestPrice <= 0) {
            requireEnoughHistory(round);
            (round, finalPrice, latestTimestamp) = getRoundData(_aggregator, round - 1);
        }
        return (round, finalPrice, latestTimestamp);
    }


    function getRoundData(AggregatorV3Interface _aggregator, uint80 _round)
        internal
        view
        returns (
            uint80,
            uint256,
            uint256
        )
    {
        (uint80 round, int256 latestPrice, , uint256 latestTimestamp, ) = _aggregator.getRoundData(_round);
        while (latestPrice <= 0) {
            requireEnoughHistory(round);
            round = round - 1;
            (, latestPrice, , latestTimestamp, ) = _aggregator.getRoundData(round);
        }
        return (round, uint256(latestPrice), latestTimestamp);
    }
```

### Impact

Potentially valuing already worthless tokens

### Recommendation

Revert instead in the case where all of last hour data is negative

## [M-04] Pricing on liquidations can be bogus

### Vulnerability Details

This was originally submitted by [hyh](https://code4rena.com/@hyh) in the V1 contest, found [here](https://github.com/code-423n4/2022-02-hubble-findings/issues/46), issue was submitted, the mitigation applied doesn't still does not solve this as the check was applied to the price immediately returned from `latestRoundData` but Hubble deals in 6 decimals and if `answer < 100`
the division at [L35](https://github.com/sherlock-audit/2023-04-hubble-exchange/blob/1f9a5ed0ca8f6004bbb7b099ecbb8ae796557849/hubble-protocol/contracts/Oracle.sol#L35C1-L35C23)
`    answer /= 100;` sidesteps the non-positive check at [L34](https://github.com/sherlock-audit/2023-04-hubble-exchange/blob/1f9a5ed0ca8f6004bbb7b099ecbb8ae796557849/hubble-protocol/contracts/Oracle.sol#L34), i.e in this scenario the submitted issue is not really mitigated against

NB: A similar case can also be made for `formatPrice()`, and the getUnderLyingPrice function is extensively used in: `AMM.sol, orderbook.sol, IF.sol...` all within Hubble.

Take a look at [Oracle.sol#L24-L36](https://github.com/sherlock-audit/2023-04-hubble-exchange/blob/1f9a5ed0ca8f6004bbb7b099ecbb8ae796557849/hubble-protocol/contracts/Oracle.sol#L24-L36)

```solidity
    function getUnderlyingPrice(address underlying)
        virtual
        external
        view
        returns(int256 answer)
    {
        if (stablePrice[underlying] != 0) {
            return stablePrice[underlying];
        }
        (,answer,,,) = AggregatorV3Interface(chainLinkAggregatorMap[underlying]).latestRoundData();
        require(answer > 0, "Oracle.getUnderlyingPrice.non_positive");
                //@audit anything below 100 would result in the value returning 0 as the price, so the protection against a zero price does not really protect
        answer /= 100;
    }
```

Check the `@audit` tag, whereas one could argue that for this to happen price has already gone very low and in the case of the index/mark ideology it doesn't count since index price < 100 (in 8 decimals) will _probably mean_ 0 in 6 decimals. All other instances of implementing `getUnderlyingPrice()` would be affected, be it the liquidations or implementations in `InsuranceFund.sol` or `clearingHouse.sol`...

Note that this can easily allow a liquidation at a non-market price that happen to not even be printed in the Oracle feed, for more insights, since other erc20 are going to be added to, A flash crash could happen to any of the future erc20 to be integrated, it easily means that users could easily be liquidated at a value, where they shouldn't be liquidatable.

#### Hypothetical POC

- User provides a good amount of collateral
- Borrows a very little amount to ensure that he _almost_ never gets liquidated
- A flash price crash happens
- User could easily get liquidated when his collateral is at an edge case `2-99x` more than the needed healthy level
- To even worsen the issue, If the prices go back up, user has already gotten liquidated and can't get his funds back.
- Key to note that it's not uncommon that during a flash crash, the prices can outrageously drop, go back up to be stable for a while
- A user who is waiting for this scenario since he believes he has 90x the health level would unfairly get liquidated. muser might want to take back his funds at this time but he already got liquidated.

### Impact

Same as previous report, but even worse since in this case attacker doesn't even need to wait for a price outbreak but rather when it's just less than 100

- The originally submitted issue by [hyh](https://code4rena.com/@hyh) is the 46th issue from the Code4rena hubble [findings](https://github.com/code-423n4/2022-02-hubble-findings/issues/) of February 2022.

### Recommendation

This all stems from incorrectly checking for non-positive prices, so this should be fixed, i.e amount should be checked to be above it's denominator

## [M-05] Share minting logic can be broken by an early depositor

### Vulnerability Details

Take a look at `deposit()` and `depositFor()`:

```solidity

    function deposit(uint amount) external {
        depositFor(_msgSender(), amount);
    }

    function depositFor(address to, uint amount) override public {
//... code blocks commented out for brevity
        if (_pool == 0) {
            shares = amount;
        } else {
            shares = amount * _totalSupply / _pool;//@audit
        }
        _mint(to, shares);
        emit FundsAdded(to, amount, _blockTimestamp());
    }

```

grep the "@audit" tag This is a pretty common vulnerability on how the first minter could break the way shares are calculated

#### Hypothetical POC

- Alice deposits `1` unit of `Usdc` . She receives `1` share.
- Alice transfers `1e6 - 1` `USDC` to IF.sol using the `ERC20.transfer()` method.
- Bob deposits `(1.999999e6)`.
- Because of Alice's transfer, Bob's deposit rounds to `1`: Bob receives `1` share: the same amount as Alice.
- Alice can then redeem her shares, because she owns half of the shares, she will receive ~ `1.5 * 1e6` tokens, effectively stealing approximately `0.5 * 1e6` from Bob.

### Impact

Early minters can break the shares logic

### Recommendation

Consider using a similar mitigation as the one used in [Uniswap V2](https://github.com/Uniswap/v2-core/blob/master/contracts/UniswapV2Pair.sol#L119-L124), where they permanently lock the first minimum liquidity tokens

> NB: This was not submitted during the contest as is seen from [here](https://github.com/sherlock-audit/2023-04-hubble-exchange-judging/issues?q=bauchibred), this is beacuse this is an _update_ contest for Hubble and after writing the report it had to be withdrawn as a similar finding exists in their previous security review.

## [L-01] Liquidations will be frozen if a token's oracle goes down or chainlink reverts call for any reason

### Vulnerability Detail

Hubble only implements chainlink oracles, if any of these oracles goes down or any of the call reverts all liquidations or anything attached to querying chainlink would not possible, potentially DOSing all instances of querying that particular oracle in scope. isn't it reasonable to implement a query to chainlink in a try/catch incase this call is reverted and implement a fallback oracle?

Chainlink has taken oracles offline in extreme cases, which one could argue is to ensure that it wasn't providing inaccurate data to protocols as at the time they decide to take it down.

In such a situation everything pertaining a call to chainlink would be inaccessible, i.e liquidations and every other logic where getting the price of a token is needed.

[Oracle.sol#L106-L123](https://github.com/sherlock-audit/2023-04-hubble-exchange/blob/1f9a5ed0ca8f6004bbb7b099ecbb8ae796557849/hubble-protocol/contracts/Oracle.sol#L106-L123):

```solidity

    function getLatestRoundData(AggregatorV3Interface _aggregator)
        internal
        view
        returns (
            uint80,
            uint256 finalPrice,
            uint256
        )
    {
        (uint80 round, int256 latestPrice, , uint256 latestTimestamp, ) = _aggregator.latestRoundData();
        finalPrice = uint256(latestPrice);
        if (latestPrice <= 0) {
            requireEnoughHistory(round);
            (round, finalPrice, latestTimestamp) = getRoundData(_aggregator, round - 1);
        }
        return (round, finalPrice, latestTimestamp);
    }
```

### Impact

Normal operation of protocol can not be guaranteed, multiple DOS could be encountered, in the case of liquidations it could even be worse as these may not be possible at a time when the protocol needs them most. Resulting in the value of user's asset falling way below their debts, i.e in the case where oracle is down while the price of the asset is falling, this easily pushes the protocol into insolvency.

### Recommendation

Provide a safeguard, after discussions with the sponsors the idea of a try/catch was accepted, and then an implementation of a fallback oracle in the case where chainlink is inacessible

## [L-02] Potential DOS to MarginAccount::isLiquidatable() or any other function that calls this

### Summary

The `isLiquidatable()` function contains multiple calls to other contracts, i.e the`ClearingHouse.sol` and `AMM.sol` note that asides the gas hefty execution of the `isLiquidatable()`function itself there exist an unrestricted iteration over the AMMs with the help of `getTotalNotionalPositionAndUnrealizedPnl()` , which could potentially lead to gas exhaustion and render the system not being able to know if the user is liquidatable or not.
Do note that protocol seems to know about the issue of how gas hefty the calls could be, meaning that the `getNotionalPositionAndMargin()` should instead be used in `isLiquidatable()` since it uses _precompile (bibliophile)_

### Vulnerability Detail

The `isLiquidatable` function in `MarginAccount.sol` calls the `getTotalNotionalPositionAndUnrealizedPnl()` function of `ClearingHouse.sol`, which, in turn, iterates over all the AMM contracts stored in the `amms` array. For each AMM contract, it calls the `getOptimalPnl()` function. However, there are no limits imposed on the number of AMMs a trader can have positions in, making the iteration process computationally expensive and prone to gas exhaustion, note that this could even be worse, if say this loop occured after a call to the `liquidateExactRepay()` function, i.e a gas has been spent on updating positions already and then a call to `isLiquidate()` is made

### Impact

An attacker could exploit this vulnerability by creating a large number of positions across multiple AMMs. As a result, the iteration process within the `isLiquidatable` function could consume excessive gas, leading to an "out of gas" error, essentially meaning that the user can side step being liquidatable

Affected code snippets:

[MarginAccount.sol#L251-L311](https://github.com/sherlock-audit/2023-04-hubble-exchange/blob/1f9a5ed0ca8f6004bbb7b099ecbb8ae796557849/hubble-protocol/contracts/MarginAccount.sol#L251-L311)
[ClearingHouse.sol#L374-L388](https://github.com/sherlock-audit/2023-04-hubble-exchange/blob/1f9a5ed0ca8f6004bbb7b099ecbb8ae796557849/hubble-protocol/contracts/ClearingHouse.sol#L374-L388)

```solidity
// Relevant portion of the `isLiquidatable` function in `MarginAccount.sol`
function isLiquidatable(address trader, bool includeFunding)
    override
    public
    view
    returns(IMarginAccount.LiquidationStatus _isLiquidatable, uint repayAmount, uint incentivePerDollar)
{
    // ... (omitted code for brevity) ...

    (uint256 notionalPosition,) = clearingHouse.getTotalNotionalPositionAndUnrealizedPnl(trader, 0, IClearingHouse.Mode.Min_Allowable_Margin);

    // ... (omitted code for brevity) ...
}


function getTotalNotionalPositionAndUnrealizedPnl(address trader, int256 margin, Mode mode)
    override
    public
    view
    returns (uint256 notionalPosition, int256 unrealizedPnl)
{
    uint256 _notionalPosition;
    int256 _unrealizedPnl;
    uint numAmms = amms.length;
    for (uint i; i < numAmms; ++i) {
        (_notionalPosition, _unrealizedPnl) = amms[i].getOptimalPnl(trader, margin, mode);
        notionalPosition += _notionalPosition;
        unrealizedPnl += _unrealizedPnl;
    }
}

function getOptimalPnl(address trader, int256 margin, IClearingHouse.Mode mode)
    override
    external
    view
    returns (uint notionalPosition, int256 unrealizedPnl)
{
    Position memory position = positions[trader];
    if (position.size == 0) {
        return (0, 0);
    }

    // based on last price
    int256 lastPriceBasedMF;
    (notionalPosition, unrealizedPnl, lastPriceBasedMF) = getPositionMetadata(
        lastPrice(),
        position.openNotional,
        position.size,
        margin
    );

    // based on oracle price
    (uint oracleBasedNotional, int256 oracleBasedUnrealizedPnl, int256 oracleBasedMF) = getPositionMetadata(
        oracle.getUnderlyingPrice(underlyingAsset).toUint256(),
        position.openNotional,
        position.size,
        margin
    );

    // while evaluating margin for liquidation, we give the best deal to the user
    if (
        (mode == IClearingHouse.Mode.Maintenance_Margin && oracleBasedMF > lastPriceBasedMF) ||
        (mode == IClearingHouse.Mode.Min_Allowable_Margin && oracleBasedMF < lastPriceBasedMF)
    ) {
        return (oracleBasedNotional, oracleBasedUnrealizedPnl);
    }
}



```

### Tool used

Manual Audit

### Recommendation

Simple fix, since there will not be an out of gas issue if the precompile is used, use `getNotionalPositionAndMargin()` in `isLiquidatable()` instead of `getTotalNotionalPositionAndUnrealizedPnl()`
